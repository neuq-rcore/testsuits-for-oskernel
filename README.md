# TestSuits for OS Kernel 2024 On LoongArch64 Platform (Final 2024)

## 简介

本仓库的测试集合主要是为全国大学生操作系统比赛，龙芯赛道提供比赛测试样例。
关于具体比赛访问 https://github.com/oscomp

## 测试样例（待定，后续继续补充）

- busybox
- busybox+lua相关
- lmbench相关
- iperf
- libc-bench
- libc-test
- netperf
- time-test

### 注意

time-test 为测试Kernel的time函数是否准确，其结果只作为专家评审的参考，不计入总分，但time-test必须成功执行成绩才有效。

- `lua`脚本和其他测试脚本要依赖`busybox`的`sh`功能。所以OS kernel首先需要支持`busybox`的`sh`功能。
- 部分脚本会需要特定的OS功能（syscall, device file等），OS kernel需要一步一步地添加功能，以支持不同程序的不同执行方式。

在libc-test样例中，包含动态链接的样例程序entry-dynamic.exe。此执行文件的动态链接解释器为/lib/ld-musl-loongarch64-sf.so.1，此文件为libc.so的动态链接。
由于Fat32文件系统不支持动态链接功能，因此比赛时各队伍请将/lib/ld-musl-loongarch64-sf.so.1当作/libc.so处理。

## 构建测试用例

### 获取编译工具链

从[该仓库](https://github.com/LoongsonLab/oscomp-toolchains-for-oskernel)中下载对应的工具链，具体链接为https://github.com/LoongsonLab/oscomp-toolchains-for-oskernel/releases/download/gcc-13.2.0-loongarch64/gcc-13.2.0-loongarch64-linux-gnu.tgz

### 常规决赛测试用例

```shell
# 编译测试样例，所有测试用例将会出现在sdcard/目录，将该目录拷贝至参赛OS的文件系统中即可
make all
```

### Linux Test Protect 编译
LTP(Linux Test Project)是由SGI，OSDL和Bull发起的联合项目，由IBM，红帽，甲骨文等公司开发和维护。
该项目的目标是向开源社区提供测试，以验证Linux的可靠性，健壮性和稳定性。

LTP测试套件包含一系列用于测试Linux内核和相关功能的工具。目标是通过将测试自动化带入测试工作来改进Linux内核和系统库。

目前我们主要测试linux的系统调用syscall。

```shell

cd linux-test-project
# --prefix设置为你自己的构建路径
./configure --prefix=/home/airxs/user/os/gitlab/github/linux-test-project/local/ltp \
      AR=loongarch64-linux-gnu-ar RANLIB=loongarch64-linux-gnu-ranlib --host=loongarch64-linux-gnu \
      --target=loongarch64-linux-gnu CFLAGS=-static LDFLAGS=-static

cd testcases/kernel/syscalls
make -j
```
然后在syscalls/下，各个系统调用测试文件夹内生成可执行文件，然后复制到ramdisk中测试。

