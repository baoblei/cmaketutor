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

##库
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

  







偏偏 这地球这么挤这么小这么瘦 太阳刻意晒的那么凶