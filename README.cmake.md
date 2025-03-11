# Nginx CMake 构建指南

本文档提供了使用CMake构建Nginx的指南。

## 先决条件

在使用CMake构建Nginx之前，请确保您的系统上已安装以下软件：

- CMake (版本 3.17 或更高)
- C编译器 (如GCC, Clang或MSVC)
- PCRE库 (用于正则表达式支持)
- OpenSSL库 (用于SSL/TLS支持)
- zlib库 (用于压缩支持)

## 构建步骤

### 1. 创建构建目录

```bash
mkdir build
cd build
```

### 2. 配置项目

在Windows上使用MSVC:

```bash
cmake .. -G "Visual Studio 16 2019" -A x64
```

在Linux/MacOS上:

```bash
cmake ..
```

您也可以使用CMake的图形界面:

```bash
cmake-gui ..
```

### 3. 构建项目

在Windows上:

```bash
cmake --build . --config Release
```

在Linux/MacOS上:

```bash
cmake --build .
```

或者直接使用生成的构建系统:

```bash
make
```

### 4. 自定义构建选项

您可以通过设置以下CMake变量来自定义构建:

- `CMAKE_BUILD_TYPE`: 设置构建类型 (Debug/Release/RelWithDebInfo)
- `CMAKE_INSTALL_PREFIX`: 设置安装路径

例如:

```bash
cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/usr/local/nginx
```

## 目录结构

CMake项目结构如下:

```
nginx/
├── CMakeLists.txt          # 主CMake配置文件
├── cmake/                  # CMake模块和工具
│   ├── CMakeLists.txt
│   └── auto_group_files.cmake
├── src/                    # 源代码
│   ├── CMakeLists.txt      # src目录的构建配置
│   ├── core/
│   ├── event/
│   ├── http/
│   ├── mail/
│   ├── misc/
│   ├── os/
│   └── stream/
```

## 注意事项

1. 该CMake构建系统仍在开发中，可能无法涵盖Nginx的所有功能。
2. 某些平台特定的功能可能需要额外的配置。
3. 与原始的构建系统相比，部分高级配置选项可能不可用。

## 问题排查

如果您在构建过程中遇到问题:

1. 确保您已安装所有必要的依赖项。
2. 检查CMake错误输出，查找可能的问题。
3. 尝试清理构建目录并重新开始。

```bash
rm -rf build/
mkdir build
cd build
cmake ..
```

4. 查看CMake生成的配置文件 `CMakeCache.txt` 以检查检测到的库和路径是否正确。 