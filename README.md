# Getting Started

## Building Eclipse Cyclone DDS

In order to build Cyclone DDS you need a Linux, Mac or Windows 10 machine (or, with some caveats, a *BSD, QNX, OpenIndiana or a Solaris 2.6 one) with the following installed on your host:

  * C compiler (most commonly GCC on Linux, Visual Studio on Windows, Xcode on macOS);
  * Optionally GIT version control system;
  * [CMake](https://cmake.org/download/), version 3.16 or later;
  * Optionally [OpenSSL](https://www.openssl.org/), we recommend a fully patched and supported version but 1.1.1 will still work;
  * Optionally [Eclipse Iceoryx](https://iceoryx.io) version 2.0 for shared memory and zero-copy support;
  * Optionally [Bison](https://www.gnu.org/software/bison/) parser generator. A cached source is checked into the repository.

If you want to play around with the parser you will need to install the bison parser generator. On Ubuntu `apt install bison` should do the trick for getting it installed.
On Windows, installing chocolatey and `choco install winflexbison3` should get you a long way.  On macOS, `brew install bison` is easiest.

To obtain Eclipse Cyclone DDS, d
通过在终端输入指令克隆CycloneDDS库

    $ git clone https://github.com/eclipse-cyclonedds/cyclonedds.git
    $ cd cyclonedds
    $ mkdir build

Depending on whether you want to develop applications using Cyclone DDS or contribute to it you can follow different procedures:

### Build 配置

这里除了像 CMAKE_BUILD_TYPE 这样的标准选项外，还有一些使用 CMake 预定义变量（defines）指定的配置选项：

* `-DBUILD_EXAMPLES=ON`: 和样例一起build
* `-DBUILD_TESTING=ON`: build测试套件（会强制导出库中的所有符号） 
* `-DBUILD_IDLC=NO`: 禁用构建 IDL 编译器（会影响示例、测试以及 'ddsperf' 的构建）
* `-DBUILD_DDSPERF=NO`: 禁用构建用于性能测试的 'ddsperf' 工具
* `-DENABLE_SSL=NO`: 不查找 OpenSSL，移除 TLS/TCP 支持，并避免构建实现身份验证与加密的插件（默认为 AUTO，即检测到 OpenSSL 时自动启用）
* `-DENABLE_ICEORYX=NO`: 不查找 Iceoryx，禁用构建 PSMX Iceoryx 插件（默认为 AUTO，即检测到 Iceoryx 时自动启用）
* `-DENABLE_SECURITY=NO`: 不在核心代码中构建安全接口和钩子，也不构建相关插件（可以在没有 OpenSSL 的情况下启用安全特性，但此时需要从其他地方获取插件）
* `-DENABLE_LIFESPAN=NO`: 排除对有限生存期（finite lifespans）QoS 的支持
* `-DENABLE_DEADLINE_MISSED=NO`: 排除对有限截止时间（finite deadline）QoS 设置的支持
* `-DENABLE_TYPELIB=NO`: 排除对类型库（Type Library）的支持，此项同时需要配合设置'-DENABLE_TYPE_DISCOVERY=NO'和'-DENABLE_TOPIC_DISCOVERY=NO'来禁用类型发现与主题发现
* `-DENABLE_TYPE_DISCOVERY=NO`: 排除对类型发现和类型兼容性检查（即 XTypes 的绝大部分功能）的支持，此项同时需要配合设置 '-DENABLE_TOPIC_DISCOVERY=NO' 来禁用主题发现
* `-DENABLE_TOPIC_DISCOVERY=NO`: 排除对主题发现（Topic Discovery）的支持
* `-DENABLE_SOURCE_SPECIFIC_MULTICAST=NO`: 禁用对特定源组播（SSM）的支持（在 QNX 系统上构建时可能需要同时禁用此项与 -DENABLE_IPV6=NO）
* `-DENABLE_IPV6=NO`: 禁用 IPv6 支持（在 QNX 系统上构建时可能需要同时禁用此项与 -DENABLE_SOURCE_SPECIFIC_MULTICAST=NO）
* `-DBUILD_IDLC_XTESTS=NO`: 包含针对 IDL 编译器的测试集。该测试会在（测试）运行时使用 C 后端编译 idl 文件，并调用 C 编译器为生成的类型构建测试应用，再通过执行该应用来完成实际测试（不支持 Windows 系统）
* `-DENABLE_QOS_PROVIDER=NO`: 禁用对 QoS 提供程序（QoS Provider）的支持
### 对于软件开发者

To build and install the required libraries needed to develop your own applications using Cyclone
DDS requires a few simple steps.
There are some small differences between Linux and macOS on the one
hand, and Windows on the other.
对于linux 或 MacOS

    $ cd build
    $ cmake -DCMAKE_INSTALL_PREFIX=<install-location> ..
    $ cmake --build .

对于windows

    $ cd build
    $ cmake -G "<generator-name>" -DCMAKE_INSTALL_PREFIX=<install-location> ..
    $ cmake --build .

where you should replace `<install-location>` by the directory under which you would like to install Cyclone DDS and `<generator-name>` by one of the ways CMake [generators](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html) offer for generating build files.
For example, "Visual Studio 15 2017 Win64" would target a 64-bit build using Visual Studio 2017.

To install it after a successful build, do:

    $ cmake --build . --target install

which will copy everything to:

  * `<install-location>/lib`
  * `<install-location>/bin`
  * `<install-location>/include/ddsc`
  * `<install-location>/share/CycloneDDS`

Depending on the installation location you may need administrator privileges.

At this point you are ready to use Eclipse Cyclone DDS in your own projects.

Note that the default build type is a release build with debug information included (RelWithDebInfo), which is generally the most convenient type of build to use from applications because of a good mix between performance and still being able to debug things.  If you'd rather have a Debug or pure Release build, set `CMAKE_BUILD_TYPE` accordingly.

### Contributing to Eclipse Cyclone DDS

We very much welcome all contributions to the project, whether that is questions, examples, bug
fixes, enhancements or improvements to the documentation, or anything else really.
When considering contributing code, it might be good to know that build configurations for Azure pipelines are present in the repository and that there is a test suite built using a simple testing framework and CTest that can be built locally if desired.
To build it, set the cmake variable `BUILD_TESTING` to on when configuring, e.g.:

    $ cd build
    $ cmake -DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTING=ON ..
    $ cmake --build .
    $ ctest

## Documentation

The [documentation](https://cyclonedds.io/docs) is still rather limited and some parts of it are still only available in the form of text files in the `docs` directory.
This README is usually out-of-date and the state of the documentation is slowly improving, so it definitely worth hopping over to have a look.

## Building and Running the Roundtrip Example

We will show you how to build and run an example program that measures latency.
The examples are built automatically when you build Cyclone DDS, so you don't need to follow these steps to be able to run the program, it is merely to illustrate the process.

    $ mkdir roundtrip
    $ cd roundtrip
    $ cmake <install-location>/share/CycloneDDS/examples/roundtrip
    $ cmake --build .

On one terminal start the application that will be responding to pings:

    $ ./RoundtripPong

On another terminal, start the application that will be sending the pings:

    $ ./RoundtripPing 0 0 0
    # payloadSize: 0 | numSamples: 0 | timeOut: 0
    # Waiting for startup jitter to stabilise
    # Warm up complete.
    # Latency measurements (in us)
    #             Latency [us]                                   Write-access time [us]       Read-access time [us]
    # Seconds     Count   median      min      99%      max      Count   median      min      Count   median      min
        1     28065       17       16       23       87      28065        8        6      28065        1        0
        2     28115       17       16       23       46      28115        8        6      28115        1        0
        3     28381       17       16       22       46      28381        8        6      28381        1        0
        4     27928       17       16       24      127      27928        8        6      27928        1        0
        5     28427       17       16       20       47      28427        8        6      28427        1        0
        6     27685       17       16       26       51      27685        8        6      27685        1        0
        7     28391       17       16       23       47      28391        8        6      28391        1        0
        8     27938       17       16       24       63      27938        8        6      27938        1        0
        9     28242       17       16       24      132      28242        8        6      28242        1        0
       10     28075       17       16       23       46      28075        8        6      28075        1        0

The numbers above were measured on Mac running a 4.2 GHz Intel Core i7 on December 12th 2018.
From these numbers you can see how the roundtrip is very stable and the minimal latency is now down to 17 micro-seconds (used to be 25 micro-seconds) on this HW.

# Performance

Reliable message throughput is over 1MS/s for very small samples and is roughly 90% of GbE with 100
byte samples, and latency is about 30us when measured using [ddsperf](src/tools/ddsperf) between two Intel(R) Xeon(R) CPU E3-1270 V2 @ 3.50GHz (that's 2012 hardware ...) running Ubuntu 16.04, with the executables built on Ubuntu 18.04 using gcc 7.4.0 for a default (i.e., "RelWithDebInfo") build.

<img src="https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/assets/performance/20190730/throughput-async-listener-rate.png" alt="Throughput" height="400"><img src="https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/assets/performance/20190730/latency-sync-listener.png" alt="Throughput" height="400">

This is with the subscriber in listener mode, using asynchronous delivery for the throughput
test.
The configuration is a marginally tweaked out-of-the-box configuration: an increased maximum
message size and fragment size, and an increased high-water mark for the reliability window on the
writer side.
For details, see the [scripts](examples/perfscript) directory,
the
[environment details](https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/assets/performance/20190730/config.txt) and the [throughput](https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/assets/performance/20190730/sub.log) and [latency](https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/assets/performance/20190730/ping.log) data underlying the graphs.  These also include CPU usage ([throughput](https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/assets/performance/20190730/throughput-async-listener-cpu.png) and [latency](https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/assets/performance/20190730/latency-sync-listener-bwcpu.png)) and [memory usage](https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/assets/performance/20190730/throughput-async-listener-memory.png).

# Run-time configuration

The out-of-the-box configuration should usually be fine, but there are a great many options that can be tweaked by creating an XML file with the desired settings and defining the `CYCLONEDDS_URI` to point to it.
E.g. (on Linux):

    $ cat cyclonedds.xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cdds.io/config https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
        <Domain Id="any">
            <General>
                <Interfaces>
                    <NetworkInterface autodetermine="true" priority="default" multicast="default" />
                </Interfaces>
                <AllowMulticast>default</AllowMulticast>
                <MaxMessageSize>65500B</MaxMessageSize>
            </General>
            <Discovery>
                <EnableTopicDiscoveryEndpoints>true</EnableTopicDiscoveryEndpoints>
            </Discovery>
            <Internal>
                <Watermarks>
                    <WhcHigh>500kB</WhcHigh>
                </Watermarks>
            </Internal>
            <Tracing>
                <Verbosity>config</Verbosity>
                <OutputFile>cdds.log.${CYCLONEDDS_PID}</OutputFile>
            </Tracing>
        </Domain>
    </CycloneDDS>
    $ export CYCLONEDDS_URI=file://$PWD/cyclonedds.xml

(on Windows, one would have to use `set CYCLONEDDS_URI=file://...` instead.)

This example shows a few things:

* `Interfaces` can be used to override the interfaces selected by default.
  Members are
  * `NetworkInterface[@autodetermine]` tells Cyclone DDS to autoselect the interface it deems best.
  * `NetworkInterface[@name]` specifies the name of an interface to select (not shown above, alternative for autodetermine).
  * `NetworkInterface[@address]` specifies the ipv4/ipv6 address of an interface to select (not shown above, alternative for autodetermine).
  * `NetworkInterface[@multicast]` specifies whether multicast should be used on this interface.
    The default value 'default' means Cyclone DDS will check the OS reported flags of the interface and enable multicast if it is supported.
    Use 'true' to ignore what the OS reports and enable it anyway and 'false' to always disable multicast on this interface.
  * `NetworkInterface[@priority]` specifies the priority of an interface.
    The default value (`default`) means priority `0` for normal interfaces and `2` for loopback interfaces.
* `AllowMulticast` configures the circumstances under which multicast will be used.
  If the selected interface doesn't support it, it obviously won't be used (`false`); but if it does support it, the type of the network adapter determines the default value.
  For a wired network, it will use multicast for initial discovery as well as for data when there are multiple peers that the data needs to go to (`true`).
  On a WiFi network it will use it only for initial discovery (`spdp`), because multicast on WiFi is very unreliable.
* `EnableTopicDiscoveryEndpoints` turns on topic discovery (assuming it is enabled at compile time), it is disabled by default because it isn't used in many system and comes with a significant amount of overhead in discovery traffic.
* `Verbosity` allows control over the tracing, "config" dumps the configuration to the trace output (which defaults to "cyclonedds.log", but here the process id is appended).
  Which interface is used, what multicast settings are used, etc., is all in the trace.
  Setting the verbosity to "finest" gives way more output on the inner workings, and there are various other levels as well.
* `MaxMessageSize` controls the maximum size of the RTPS messages (basically the size of the UDP payload).
  Large values such as these typically improve performance over the (current) default values on a loopback interface.
* `WhcHigh` determines when the sender will wait for acknowledgements from the readers because it has buffered too much unacknowledged data.
  There is some auto-tuning, the (current) default value is a bit small to get really high throughput.

Background information on configuring Cyclone DDS can be found [here](docs/manual/config.rst) and a list of settings is [available](docs/manual/options.md).

# Trademarks

* "Eclipse Cyclone DDS", "Cyclone DDS", "Eclipse Iceoryx" and "Iceoryx" are trademarks of the Eclipse Foundation.
* "DDS" is a trademark of the Object Management Group, Inc.
* "ROS" is a trademark of Open Source Robotics Foundation, Inc.
