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
    -  target_compile_definetions(name info USE_MYMATH):  set the compile definition "USE_MYMATH"
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






偏偏 这地球这么挤这么小这么瘦 太阳刻意晒的那么凶