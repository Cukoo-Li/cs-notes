# CMake入门

## 语法特性介绍

基本语法格式：指令(参数1 参数2 ...)

- 参数使用括号括起
- 参数之间使用空格或分号隔开
- 指令是不区分大小写的，参数和变量是区分大小写的
- 变量使用${}方式取值，但是在if控制语句中可以直接使用变量名

```cmake
set(HELLO hello.cpp)
add_executable(hello main.cpp hello.cpp)
ADD_EXECUTABLE(hello main.cpp ${HELLO})
```

## 重要指令和常用变量

### 重要指令

- cmake_minimum_required - 指定CMake的最小版本要求

  语法：camke_minimum_required(VERSION version_number [FATAL_ERROR])

  ```cmake
  # CMake最小版本要求为2.8.3
  cmake_minimum_required(VERSION 2.8.3)
  ```

- project - 定义项目名称，并可指定项目支持的语言，默认是支持所有语言

  语法：project(project_name \[CXX] [C] [JAVA])

  ```cmake
  # 指定项目名为HELLOWORLD
  project(HELLOWORLD)
  ```

- set - 显式地定义变量

  语法：set(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])

  ```cmake
  # 定义一个变量，名为SRC，值为sayhello.cpp hello.cpp
  set(SRC sayhello.cpp hello.cpp)
  ```

- include_directories - 向项目添加头文件搜索路径

  语法：include_directories([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)

  ```cmake
  # 将/usr/include/myincludefolder和./include添加到头文件搜索路径
  include_directories(/usr/include/myincludefolder ./include)
  ```

- link_directories - 向项目添加库文件搜索路径

  语法：link_directories(dir1 dir2 ...)

  ```cmake
  # 将/usr/lib/mylibfolder和./lib添加到库文件搜索路径
  link_directories(/usr/lib/mylibfolder ./lib)
  ```

- add_library - 生成库文件

  语法：add_library(lib_name [SHARED|STATIC|MODULE] [EXCLUDE_FROM_ALL]  source1 source2 ... sourceN)

  ```cmake
  # 通过变量SRC生成libhello.so共享库
  add_library(hello SHARED ${SRC})
  ```

- add_compile_options - 添加编译参数

  语法：add_compile_options(...)

  ```cmake
  # 添加编译参数 -Wall -std=c++11 -O2
  add_compile_options(-Wall -std=c++11 -O2)
  ```

- add_executable - 生成可执行文件

  语法：add_executable(exe_name source1 source2 ...)

  ```cmake
  # 编译main.cpp生成可执行文件main
  add_executable(main main.cpp)
  ```

- target_link_libraries - 为target添加需要链接的共享库

  语法：target_link_libraries(target_name library1[debug|optimized] library2 ...)

  ```cmake
  # 将动态库文件hello链接到可执行文件main
  target_link_libraries(main hello)
  ```

- add_subdirectory - 添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置

  语法：add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])

  ```cmake
  # 添加src子目录，src中需有一个CMakeLists.txt
  add_subdirectory(src)
  ```

- aux_source_directory - 发现一个目录下所有的源代码文件并将列表存储在一个变量中，这个指令被用来自动构建源文件列表

  语法：aux_source_directory(dir VAR)

  ```cmake
  # 定义SRC变量，其值为当前目录下所有的源代码文件
  aux_source_directory(. SRC)
  # 编译SRC变量所代表的源代码文件，生成main可执行文件
  add_executable(main ${SRC})
  ```

### 常用变量

- CMAKE_C_FLAGS - gcc编译参数

- CMAKE_CXX_FLAGS - g++编译参数

  ```cmake
  # 在CMAKE_CXX_FLAGS编译选项后追加-std=c++11
  set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  ```

- CMAKE_BUILD_TYPE - 编译类型(Debug|Release)

  ```cmake
  # 设定编译类型为debug，调试时需要选择debug
  set(CMAKE_BUILD_TYPE Debug)
  # 设定编译类型为release，发布时需要选择release
  set(CMAKE_BUILD_TYPE Release)
  ```

- CMAKE_BINARY_DIR, PROJECT_BINARY_DIR, _BINARY_DIR

- CMAKE_SOURCE_DIR, PROJECT_SOURCE_DIR, _SOURCE_DIR

- CMAKE_C_COMPILER - C编译器

- CMAKE_CXX_COMPILER - C++编译器

- EXECUTABLE_OUTPUT_PATH - 可执行文件输出的存放路径

- LIBRARY_OUTPUT_PATH - 库文件输出的存放路径

## 使用CMake构建项目

### 目录结构

- 项目主目录包含一个CMakeLists.txt
  - 包含源文件的子文件夹包含一个CMakeLists.txt，主目录的CMakeLists.txt通过add_subdirectory添加子目录
  - 包含源文件的子文件夹不包含一个CMakeLists.txt，子目录编译规则体现在主目录的CMakeLists.txt中

### 编译流程

1. 手动编写CMakeLists.txt
2. 执行命令cmake PATH生成Makefile（PATH为顶层CMakeLists.txt所在目录）
3. 执行命令make进行编译

### 两种构建方式

- 内部构建(in-source build) - 不推荐

  ```cmake
  # 根据当前目录的CMakeLists.txt进行构建，并在当前目录生成Makefile和其他文件
  cmake .
  # 执行make命令，在当前目录生成target
  make
  ```

- 外部构建(out-of-source build) - 推荐

  ```cmake
  # 在当前目录创建build文件夹
  mkdir build
  # 进入到build文件夹中
  cd build
  # 根据上级目录的CMakeLists.txt进行构建，并在当前目录生成Makefile和其他文件
  cmake ..
  # 执行make命令，在当前目录生成target
  make
  ```

### 常用的CMakeList.txt示例

```cmake
cmake_minumum_required(VERSION 3.0)

project(main)

include_directories(include)

add_subdirectory(src)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLGS} -Wall")

set(CMAKE_BUILD_TYPE Debug)

add_executable(main xxx.cpp xxx.cpp ...)
```

## 实践 - 在VSCode中使用CMake

1. 配置lauch.json和task.json
2. 编写CMakeLists.txt
3. 按F5进行运行/调试

### launch.json

- 保存VSCode运行配置信息
- "program"指定可执行文件的路径
- "preLaunchTask"指定运行调试前要执行的tasks.json中的任务

```json
{
    // 使用 IntelliSense 了解相关属性。
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceRoot}\\out\\${fileBasenameNoExtension}.exe",
            // "program": "${fileDirname}\\build\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "D:\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "g++"
            // "preLaunchTask": "Build"
        }
    ]
}
```

### tasks.json

- 保存运行前要执行的任务的配置信息
- "label"用于标记一个特定的任务

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "cppbuild",
            "label": "g++",
            "command": "D:\\mingw64\\bin\\g++.exe",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${workspaceRoot}\\out\\${fileBasenameNoExtension}.exe",
                // "-fexec-charset=GBK"  // 如果使用内置终端，就注销掉这行
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        },
        {
            "type": "shell",
            "label": "cmake",
            "command": "cmake",
            "args": [
                ".."
            ]
        },
        {
            "label": "make",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "command": "make",
            "args": []
        },
        {
            "label": "Build",
            "dependsOrder": "sequence",
            "dependsOn": [
                "cmake",
                "make"
            ]
        }
    ]
}
```

