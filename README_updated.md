# CycloneDDS 环境配置与通讯延迟测试指南

本指南介绍如何在 Linux 开发板与 Windows 主机上分别配置 Eclipse CycloneDDS 开发环境，使用 Visual Studio 进行跨平台远程开发/调试，并利用官方 Demo 测试主机与开发板之间的 DDS 通讯延迟。

---

## 目录
- [一、 Linux 开发板环境配置](#一-linux-开发板环境配置)
- [二、 Windows 主机环境配置](#二-windows-主机环境配置)
- [三、 使用 Visual Studio 连接开发板](#三-使用-visual-studio-连接开发板)
- [四、 通讯延迟测试 (Demo)](#四-通讯延迟测试-demo)
- [五、 常见问题与排查](#五-常见问题与排查)

---

## 一、 Linux 开发板环境配置
从 GitHub 拉取官方源码并进行编译：

```bash
$ git clone https://github.com/eclipse-cyclonedds/cyclonedds.git
$ cd cyclonedds
$ mkdir build
$ cd build
$ cmake -DBUILD_EXAMPLES=ON ..
$ cmake --build .
```

---

## 二、 Windows 主机环境配置

### 1. 前置软件准备
- **Visual Studio**：勾选 **“使用 C++ 的桌面开发”** 以及 **“使用 C++ 的 Linux 开发”** 工作负载。
- **CMake**：确保已安装并添加至系统环境变量 `PATH`。
- **Git for Windows**：用于拉取源码。

### 2. 编译与安装 CycloneDDS
打开 Windows 终端：

```cmd
:: 克隆源码
git clone https://github.com/eclipse-cyclonedds/cyclonedds.git
cd cyclonedds
mkdir build 
cd build
cmake -DBUILD_EXAMPLES=ON ..
cmake --build . --config Release
```

### 3. 配置系统环境变量
1. 将 `C:\cycloneddsin` 添加至系统的 **Path** 环境变量中。
2. 新增环境变量 `CycloneDDS_DIR`，值为 `C:/cyclonedds/lib/cmake/CycloneDDS`。

---

## 三、 使用 Visual Studio 连接开发板

通过 Visual Studio 的远程 C++ 开发功能，可以直接在 Windows 上编写代码并在 Linux 开发板上进行编译与调试。

### 1. 添加 SSH 远程连接
1. 打开 Visual Studio，点击上方菜单栏 **工具 (Tools) -> 选项 (Options)**。
2. 展开 **跨平台 (Cross Platform) -> 连接管理器 (Connection Manager)**。
3. 点击 **添加 (Add)**：
   - **主机名 (Host Name)**：输入开发板 IP 地址（例如 `192.168.1.10` 或共享网段 IP）。
   - **端口 (Port)**：`22`
   - **用户名 (User Name)**：开发板用户名
   - **身份验证类型 (Authentication type)**：选择密码或 SSH 密钥。
4. 点击 **连接 (Connect)** 完成连接认证。

### 2. 创建并配置远程 C++ CMake 项目
1. 在 VS 中新建项目，选择 **CMake 项目 (CMake Project)**。
2. 打开 `CMakePresets.json` 或 `CMakeSettings.json`：
   - 添加 Linux 远程配置模板（例如 `Linux-GCC-Release`）。
   - 将 **远程机器名 (Remote Machine Name)** 设置为刚才添加的 SSH 连接。
3. 在项目的 `CMakeLists.txt` 中添加 CycloneDDS 依赖：

```cmake
cmake_minimum_required(VERSION 3.16)
project(DDS_Demo)

find_package(CycloneDDS REQUIRED)

add_executable(dds_app main.cpp)
target_link_libraries(dds_app PRIVATE CycloneDDS::ddsc)
```

4. 选择远程目标进行构建与断点调试，VS 将自动将源码同步至开发板并调用开发板上的 GCC 进行编译。

---

## 四、 通讯延迟测试 (Demo)

CycloneDDS 源码自带了 `roundtrip` 延迟测试 Demo（以及 `throughput` 性能测试），可直接用于测量主机与开发板之间的 RTT（往返延迟）。

> **前置条件**：编译 CycloneDDS 时请确保开启了示例构建选项 `-DBUILD_EXAMPLES=ON`。

### 1. 准备配置文件（确保网络多播/网卡配置正确）
在主机与开发板上创建 `cyclonedds.xml`，显式指定所使用的网卡接口：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<CycloneDDS xmlns="https://cdds.io/config">
    <Domain id="any">
        <General>
            <NetworkInterfaceAddress>AUTO</NetworkInterfaceAddress>
        </General>
    </Domain>
</CycloneDDS>
```
*注：可通过将 `AUTO` 替换为具体网卡名称或 IP（如 `end0` / `192.168.137.100`）来绑定特定网口。*

在终端中导入配置文件：
- **Linux**：`export CYCLONEDDS_URI=file:///path/to/cyclonedds.xml`
- **Windows (CMD)**：`set CYCLONEDDS_URI=file://C:/path/to/cyclonedds.xml`

---

### 2. 运行延迟测试 (Roundtrip Example)

CycloneDDS 编译生成的二进制程序包含 `RoundtripPong` (接收应答端) 与 `RoundtripPing` (发送端)。

#### 步骤 A：在开发板上启动 Pong (订阅/应答端)
```bash
# 进入 CycloneDDS 编译目录下的 bin 文件夹
cd cyclonedds/build/bin

# 运行 roundtrip 测试的 pong 端
./RoundtripPong
```

#### 步骤 B：在 Windows 主机上启动 Ping (发送测试端)
在 Windows 命令行打开对应的编译输出目录，运行 `RoundtripPing`（后接三个参数：数据包大小 `payloadSize`、样本数 `numSamples`、超时时间 `timeOut`，传 `0 0 0` 即使用默认参数）：

```cmd
cd cycloneddsuildin\Release

:: 运行 ping 端发送测试包
RoundtripPing 0 0 0
```

---

### 3. 数据分析与延迟评估
`RoundtripPing` 运行后会在终端实时输出延迟统计指标：

```text
# Latency measurements (in us)
#             Latency [us]                                   Write-access time [us]       Read-access time [us]
# Seconds     Count   median      min      99%      max      Count   median      min      Count   median      min
        1      1593      313      176      452     1082       1593       34       15       1593        7        3
        2      1567      312      170      509     2628       1567       34       14       1567        7        3
        3      1591      301      181      715     2039       1591       34       14       1591        6        3
        4      1601      306      163      569     1766       1601       35       10       1601        7        1
        5      1720      284      165      448     2583       1720       28       16       1720        4        2
        6      1704      276      170      770     3304       1704       24       16       1704        4        2
        7      1665      285      170      652     4216       1665       59       14       1665        9        2
        8      1656      289      166      655     2304       1656       52       22       1656        7        3
        9      1570      295      170      870     5942       1570       53       16       1570        7        3
       10      1671      291      164      606     1213       1671       50       16       1671        7        2
```

| 统计指标 | 含义描述 |
| :--- | :--- |
| **min** | 最小往返时延 (Round Trip Time, RTT，单位：微秒 $\mu s$) |
| **median** | 往返时延中位数 |
| **99%** | 99% 的数据包往返时延均在此数值以内 |
| **max** | 最大往返时延 |

> **单向延迟 (One-Way Latency)** $pprox$ **RTT / 2**。  
> 在千兆局域网或直连网口环境下，CycloneDDS 的平均单向延迟通常维持在 **十至百微秒（$\mu s$）级别**。

---

## 五、 常见问题与排查

1. **`find_package(CycloneDDS)` 报错或找不到文件**：
   - 请检查 Windows 上是否正确配置了 `CycloneDDS_DIR` 系统环境变量；
   - Linux 环境下确保已执行 `sudo ldconfig` 刷新动态链接库缓存。
2. **Ping-Pong 测试无法收发数据**：
   - 检查 Windows 防火墙与 Linux iptables，确保 UDP 组播通信端口未被拦截；
   - 检查 `cyclonedds.xml` 中绑定的网卡或 IP 地址是否与当前通信网口一致。
