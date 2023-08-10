## 基础
`CmakeLists.txt`
#### 三个必要的命令

- cmake_minimun_required()
- project()
- add_executable()
#### 添加C++11
```cmake
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```
(在add_executable()上方添加)

#### 预定义的信息添加到代码中
configure_file(TutorialConfig.h.in TutorialConfig.h)
```cpp
(.h.in)
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```
```cmake
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )

```
#### 正常流程
-   mkdir build
-   cd build
-   cmake ../Step1    ("构建文件在Step1文件夹下")
-   cmake --build .

## 库
#### 创建库
add_library():
add_subdirectory(): 处理另一个目录下的CMakeLists.txt 文件
target_link_libraries():
target_include_directories():  add the binary tree to the search path for include files

-   添加可选条件
    -  option()
    -  target_compile_definetions(target INTERFACE|PUBLIC|..  [items]):  set the compile definition "USE_MYMATH"
    -  add_library()
    -  target_link_libraried()


## 添加库的使用要求
- target_include_directories(name INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}):
    通过将源代码目录添加为接口包含路径，你可以确保任何依赖于 "MathFunctions" 目标的其他目标都能够访问 "MathFunctions" 子目录中的头文件。这对于实现模块化、解耦和复用代码非常有用。
- 添加接口库设定编译要求
  - add_library(tutorial_compiler_flags INTERFACE)：接口库通常用于定义接口和属性，而不包含实际的源代码。
  - target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)：将 C++11 标准作为接口的编译要求。这表示任何使用这个接口库的目标都需要支持 C++11 标准。

##  添加生成表达式
-   `set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")`：
    这部分定义了一个变量 gcc_like_cxx，它使用条件表达式 $<COMPILE_LANG_AND_ID:...> 来判断当前编译环境是否为像 GCC 或 Clang 这样的 C++ 编译器。如果当前编译器符合条件，那么这个变量将被设置为 1，否则为 0。
-   `set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")`
    这部分定义了一个变量 msvc_cxx，它使用条件表达式 $<COMPILE_LANG_AND_ID:...> 来判断当前编译环境是否为 Microsoft Visual C++ 编译器（MSVC）。如果当前编译器符合条件，那么这个变量将被设置为 1，否则为 0。

使用上述两个变量的例子：
-   Add warning flag compile options to the interface library tutorial_compiler_flags.
     * For gcc_like_cxx, add flags -Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused
     * For msvc_cxx, add flags -W3
     Hint: Use target_compile_options()
```cmake
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>"
  "$<${msvc_cxx}:-W3>"
)
```
-   With nested generator expressions, only use the flags for thebuild-tree
 Hint: Use BUILD_INTERFACE
 ```cmake
 target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
 ```

## 安装
-  将所有库（.a/.so）安装在lib文件夹中
-  将头文件（.h）安装在include文件夹中
-  将可执行文件安装在bin文件夹中
-  命令：sudo cmake --install . --prefix "/Users/baobinglei/cmaketutor/Step5_build/release" 
```cmake
set(installable_libs MathFunctions tutorial_compiler_flags)
install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```
## 测试
-   启用测试
    -   enable_testing()

-   添加测试
    -  add_test(NAME xxx COMMAND xxx):不测试结果，只测试应用程序是否运行、没有段错误或以其他方式崩溃，并且返回值为零
    -  add_test(NAME Usage COMMAND Tutorial)
       set_tests_properties(Usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")
        测试结果里是否包含某些字符串
    -   add_test(NAME StandardUse COMMAND Tutorial 4)
        set_tests_properties(StandardUse PROPERTIES PASS_REGULAR_EXPRESSION "4 is 2")
        测试结果是否正确
    -   ```cmake
        function(do_test target arg result)
            add_test(NAME Comp${arg} COMMAND ${target} ${arg})
            set_tests_properties(Comp${arg} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
        endfunction()

        # do a bunch of result based tests
        do_test(Tutorial 4 "4 is 2")
        do_test(Tutorial 9 "9 is 3")
        do_test(Tutorial 5 "5 is 2.236")
        do_test(Tutorial 7 "7 is 2.645")
        do_test(Tutorial 25 "25 is 5")
        do_test(Tutorial -25 "-25 is (-nan|nan|0)")
        do_test(Tutorial 0.0001 "0.0001 is 0.01")
        ```
        利用函数来测试

### ctest -VV -D Experimental
ctest 可执行文件将构建项目（不用在本地构建）、运行任何测试并将结果提交到 Kitware 的公共仪表板：https://my.cdash.org/index.php?project=CMakeTutorial。
源代码目录下包含CTestConfig.cmake 文件：
```cmake
set(CTEST_PROJECT_NAME "CMakeTutorial")
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")

set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```
dashboard：
![](https://cdn.jsdelivr.net/gh/baoblei/imgs_md/20230810231743.png)

## 添加系统自省
- 首先添加CheckCXXSourceCmompiles 模块
    `include(CheckCXXSourceCompiles)`
-   编写测试是否可用的函数并返回 变量
```cmake
 check_cxx_source_compiles("
    #include <cmath>
    int main() {
      std::log(1.0);
      return 0;
    }
  " HAVE_LOG)
  check_cxx_source_compiles("
    #include <cmath>
    int main() {
      std::exp(1.0);
      return 0;
    }
  " HAVE_EXP)
```
-   再把返回的变量传递给源代码
```cmake
  if(HAVE_LOG AND HAVE_EXP)
    target_compile_definitions(SqrtLibrary
                               PRIVATE "HAVE_LOG" "HAVE_EXP"
                               )
  endif()
```
-   源代码中测试
  ```cmake
  #if defined(HAVE_LOG) && defined(HAVE_EXP)
  double result = std::exp(std::log(x) * 0.5);
  std::cout << "Computing sqrt of " << x << " to be " << result
            << " using log and exp" << std::endl;
#else
  original code
#endif
  ```

## 添加自定义命令和生成文件
- 编写一个cmake脚本：
  ```cmake
    add_executable(MakeTable MakeTable.cxx)

    target_link_libraries(MakeTable PRIVATE tutorial_compiler_flags)

    # 自定义命令
    add_custom_command(
            OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
            COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
            DEPENDS MakeTable
        )
  ```   
-  include(MakeTable.cmake)
-  将自定义生成的头文件链接到目标中，并且在include路径中包含
  ```cmake
    add_library(SqrtLibrary STATIC
              mysqrt.cxx
              ${CMAKE_CURRENT_BINARY_DIR}/Table.h
              )
    target_include_directories(SqrtLibrary PRIVATE
                             ${CMAKE_CURRENT_BINARY_DIR}
                             )

  ``` 
-   这样在源码中包含生成的头文件并调用其中的函数（变量）

偏偏 这地球这么挤这么小这么瘦 太阳刻意晒的那么凶