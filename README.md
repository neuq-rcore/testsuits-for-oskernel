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

- time-test 为测试Kernel的time函数是否准确，其结果只作为专家评审的参考，不计入总分，但time-test必须成功执行成绩才有效。
  - `lua`脚本和其他测试脚本要依赖`busybox`的`sh`功能。所以OS kernel首先需要支持`busybox`的`sh`功能。
  - 部分脚本会需要特定的OS功能（syscall, device file等），OS kernel需要一步一步地添加功能，以支持不同程序的不同执行方式。
- ext4文件系统镜像`sdcard.img`是用`mkfs.ext4`工具生成的。
  - [C或Rust语言实现的ext4相关的库和OS参考](https://github.com/oscomp/os-competition-info/blob/main/ref-info.md#ext4%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E5%8F%82%E8%80%83%E5%AE%9E%E7%8E%B0)

## 构建测试用例

### 获取编译工具链

从[该仓库](https://github.com/LoongsonLab/oscomp-toolchains-for-oskernel)中下载对应的工具链，具体链接为https://github.com/LoongsonLab/oscomp-toolchains-for-oskernel/releases/download/gcc-13.2.0-loongarch64/gcc-13.2.0-loongarch64-linux-gnu.tgz

### 常规决赛测试用例

```shell
# 编译测试样例，所有测试用例将会出现在sdcard/目录，将该目录拷贝至参赛OS的文件系统中即可
make all
```

### Linux Test Project 编译
LTP(Linux Test Project)是由SGI，OSDL和Bull发起的联合项目，由IBM，红帽，甲骨文等公司开发和维护。
该项目的目标是向开源社区提供测试，以验证Linux的可靠性，健壮性和稳定性。

LTP测试套件包含一系列用于测试Linux内核和相关功能的工具。目标是通过将测试自动化带入测试工作来改进Linux内核和系统库。

目前我们主要测试linux的系统调用syscall。

```shell

cd linux-test-project
# --prefix设置为你自己的构建路径，即编译后的二进制文件安装的路径
./configure --prefix=/root/ltp/output \
      AR=loongarch64-linux-gnu-ar RANLIB=loongarch64-linux-gnu-ranlib --host=loongarch64-linux-gnu \
      --target=loongarch64-linux-gnu 
make -j16
make install -j16

```
在上文prefix指定的路径`$(PREFIX)`下，并将本仓库[`scripts/ltp/skip_tests`](scripts/ltp/skip_tests)文件拷贝至`$(PREFIX)`路径，运行：
```shell
./runltp -f syscalls -S skip_tests
```
即可运行所有syscall测试用例，运行结果的日志为`$(PREFIX)/results/LTP_RUN_ON-2024xxxxx.log`，同学们可以忽略结果为CONF的测试用例。

### 烧录ext4 sdcard测试用例镜像

```shell
# 创建一个空的镜像文件，大小为2G
dd if=/dev/zero of=$PWD/sdcard.img bs=1M count=2048
# 将该镜像文件格式化为ext4格式
mkfs.ext4 sdcard.img
# 新建一个空目录
mkdir sdcard-testcases
# 将镜像文件挂载至该目录
sudo mount sdcard.img sdcard-testcases
# 将常规测试用例和ltp测试用例拷贝至上述目录
cp -r sdcard/ linux-test-project/output sdcard-testcases 
sudo umount sdcard-testcases
```
sdcard.img文件便是包含了所有测试用例的镜像

## 使用已编译完成的二进制测试用例

从release中下载对应的sdcard镜像，其中包含了所有已编译的测试用例

## 使用loongarch Linux运行测试用例对比实现

- 按照[此处](./loongarch-linux-qemu/README.md)下载并运行Linux镜像
- /root/sdcard中为常规决赛测试用例，/root/ltp-syscalls中为LTP测试用例
