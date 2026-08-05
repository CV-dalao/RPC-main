# Krpc · 基于 C++ 的分布式 RPC 服务系统

<div align="center">

![C++](https://img.shields.io/badge/C%2B%2B-14-blue?logo=cplusplus&style=flat-square)
![CMake](https://img.shields.io/badge/CMake-%3E%3D3.0-green?logo=cmake&style=flat-square)
![Muduo](https://img.shields.io/badge/Network-Muduo-orange?style=flat-square)
![Protobuf](https://img.shields.io/badge/Serialize-Protobuf-purple?logo=google&style=flat-square)
![ZooKeeper](https://img.shields.io/badge/Registry-ZooKeeper-yellow?style=flat-square)
![License](https://img.shields.io/badge/License-GPLv3-red?style=flat-square)

一个基于 **Muduo + Protobuf + ZooKeeper** 的高性能 C++ 分布式 RPC 服务系统。

本文档记录了本项目在 **Windows + WSL 2 + Ubuntu 24.04** 中的完整环境搭建、编译与运行流程。

</div>

---

## 📷 运行结果

### 客户端运行结果

![Krpc 客户端运行结果](img/client.png)

### 服务端运行结果

![Krpc 服务端运行结果](img/server.png)

---

## 📑 目录

- [一、安装 WSL 2](#一安装-wsl-2)
- [二、创建用户](#二创建用户)
- [三、更新 Ubuntu 并安装基础 C++ 编译环境](#三更新-ubuntu-并安装基础-c-编译环境)
- [四、安装 CMake、Ninja 和 pkg-config](#四安装-cmakeninja-和-pkg-config)
- [五、将 Windows 项目复制到 WSL](#五将-windows-项目复制到-wsl)
- [六、安装 Protobuf、Muduo、glog 和 ZooKeeper](#六安装-protobufmuduoglog-和-zookeeper)
- [七、编译项目](#七编译项目)
- [八、重新生成 Krpcheader 的 pb 文件（新版 Protobuf）](#八重新生成-krpcheader-的-pb-文件新版-protobuf)
- [九、重新生成 user 的 pb 文件](#九重新生成-user-的-pb-文件)
- [十、重新编译核心库](#十重新编译核心库)
- [十一、运行服务端](#十一运行服务端)
- [十二、运行客户端](#十二运行客户端)

---

## 一、安装 WSL 2

首先在 Windows 的 **CMD 或 PowerShell** 中安装 WSL：

```bash
wsl --install
```

安装完成后检查：

```bash
wsl -l -v
```

> **提示**：如果系统提示"适用于 Linux 的 Windows 子系统没有已安装的分发"，说明 WSL 本体已安装，只是还没有安装 Ubuntu 等 Linux 发行版。

查看可安装的发行版：

```bash
wsl --list --online
```

安装 Ubuntu 24.04：

```bash
wsl --install -d Ubuntu-24.04
```

安装完成后启动：

```bash
wsl -d Ubuntu-24.04
```

---

## 二、创建用户

Ubuntu 第一次启动时要求创建 Linux 用户。

> **注意**：
> - Linux 用户名**不能以数字开头**；
> - 设置用户密码时，终端**不会显示字符或星号**，这是正常现象。

---

## 三、更新 Ubuntu 并安装基础 C++ 编译环境

进入 Ubuntu 后先更新软件包：

```bash
sudo apt update
sudo apt upgrade -y
```

安装 C++ 编译和调试工具：

```bash
sudo apt install -y \
  build-essential \
  gdb \
  git \
  curl \
  wget \
  unzip
```

其中：

| 工具 | 说明 |
| --- | --- |
| gcc | C 编译器 |
| g++ | C++ 编译器 |
| make | 构建工具 |
| gdb | 调试器 |
| git | 代码版本管理工具 |

检查安装：

```bash
g++ --version
gcc --version
make --version
gdb --version
```

---

## 四、安装 CMake、Ninja 和 pkg-config

```bash
sudo apt install -y cmake ninja-build pkg-config
```

检查版本：

```bash
cmake --version
ninja --version
pkg-config --version
```

---

## 五、将 Windows 项目复制到 WSL

> 假设 Windows 中的项目路径是 `D:\Krpc-main`（请根据你的实际路径调整）。

创建 Linux 项目目录：

```bash
mkdir -p ~/projects
```

复制项目：

```bash
cp -a "/mnt/d/Krpc-main" ~/projects/
```

进入项目：

```bash
cd ~/projects/Krpc-main
```

---

## 六、安装 Protobuf、Muduo、glog 和 ZooKeeper

### 1. 安装 Protobuf

```bash
sudo apt install -y \
  protobuf-compiler \
  libprotobuf-dev \
  libprotoc-dev
```

### 2. 安装 Muduo

**(1) 先安装 Muduo 所需依赖：**

```bash
sudo apt install -y \
  git \
  build-essential \
  cmake \
  make \
  libboost-dev
```

**(2) 创建第三方库目录：**

```bash
mkdir -p ~/libs
cd ~/libs
```

**(3) 下载 Muduo：**

```bash
git clone https://github.com/chenshuo/muduo.git
cd muduo
```

**(4) 编译 Muduo：**

```bash
cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/usr/local
cmake --build build -j$(nproc)
```

**(5) 安装：**

```bash
sudo cmake --install build
sudo ldconfig
```

### 3. 安装 glog 和 ZooKeeper 开发库

```bash
sudo apt install -y \
  libgoogle-glog-dev \
  libgflags-dev

sudo apt install -y libzookeeper-mt-dev
```

```bash
sudo apt install -y zookeeperd zookeeper-bin
```

> **提示**：`zookeeperd` 会把 ZooKeeper 作为系统服务安装并**自动启动**，所以后面运行项目时无需手动执行 `zkServer.sh start`。

---

## 七、编译项目

```bash
cd ~/projects/Krpc-main
rm -rf build
```

配置：

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
```

编译：

```bash
cmake --build build -j$(nproc)
```

---

## 八、重新生成 Krpcheader 的 pb 文件（新版 Protobuf）

如果你安装的是**最新版本的 Protobuf**，那么项目中 `./src/Krpcheader.proto` 和 `./example/user.proto` 生成的 `.pb.cc` / `.pb.h` 可能不适用，需要重新生成。

先创建临时输出目录：

```bash
rm -rf /tmp/krpc_pb
mkdir -p /tmp/krpc_pb
```

使用当前版本的 `protoc` 重新生成：

```bash
protoc \
  -I=src \
  --cpp_out=/tmp/krpc_pb \
  src/Krpcheader.proto
```

生成文件：

```bash
/tmp/krpc_pb/Krpcheader.pb.cc
/tmp/krpc_pb/Krpcheader.pb.h
```

备份旧文件：

```bash
cp src/Krpcheader.pb.cc src/Krpcheader.pb.cc.bak
cp src/include/Krpcheader.pb.h src/include/Krpcheader.pb.h.bak
```

替换旧文件：

```bash
cp /tmp/krpc_pb/Krpcheader.pb.cc src/Krpcheader.pb.cc
cp /tmp/krpc_pb/Krpcheader.pb.h src/include/Krpcheader.pb.h
```

> **注意**：`.pb.h` 需放在 `src/include/`（头文件搜索路径），`.pb.cc` 需放在 `src/`（CMake 的 `*.cc` 收集范围）。

---

## 九、重新生成 user 的 pb 文件

```bash
protoc \
  -I=example \
  --cpp_out=example \
  example/user.proto
```

---

## 十、重新编译核心库

重新清理并构建：

```bash
cd ~/projects/Krpc-main
rm -rf build
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j$(nproc)
```

---

## 十一、运行服务端

```bash
cd ~/projects/Krpc-main/bin
./server -i ./test.conf
```

> 启动成功后，服务端会向 ZooKeeper 注册服务并开始监听，等待 RPC 调用。

---

## 十二、运行客户端

```bash
cd ~/projects/Krpc-main/bin
./client -i ./test.conf
```

> 客户端会发起多线程并发调用，结束后输出总请求数、成功/失败次数、耗时与 QPS。
