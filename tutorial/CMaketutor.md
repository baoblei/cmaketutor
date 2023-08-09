#### file
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