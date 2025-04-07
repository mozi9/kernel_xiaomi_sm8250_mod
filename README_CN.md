# 关于这个仓库
中文 | [English](README.md)

该repo (`android14-lineage21-mod` 分支)主要基于[Lineage OS 21 xiaomi sm8250 kernel source](https://github.com/LineageOS/android_kernel_xiaomi_sm8250)。

原来这个repo(即`android12-stable-mod`/`android14-stable-mod`分支)是fork自[UtsavBalar1231的仓库](https://github.com/UtsavBalar1231/kernel_xiaomi_sm8250)，但切到`android14-stable`分支的时候，发现那套代码有睡死问题（202408的几个release），所以现在切到了基于lineage21的代码来搞。MIUI特性的代码以及部分的设备驱动抠自UtsavBalar1231的仓库。

所以仍然感谢 [@UtsavBalar1231](https://github.com/UtsavBalar1231/)！

维护和编译这个内核的主要目的是想修复[电量卡在1%的问题](https://github.com/liyafe1997/Xiaomi-fix-battery-one-percent)，以及提供带KernelSU的预编译好的内核。以及再提供一个更直观和易用的编译脚本和README，方便大家自己折腾和修改，编译自己的内核！

（其中受“1%电量bug”影响的设备有：alioth, apollo, lmi, thyme, umi, pipa，因为它们都用了PM8150即高通的GEN4电量计。其它不受此bug影响的设备大可把这个内核当成个带KernelSU的官核平替，如果你想找一个带KernelSU的内核的话。并且据大家测试，该内核不带KernelSU版本可以应用[APatch](https://github.com/bmax121/APatch)）

Release里的编译好的内核成品由`android14-lineage21-mod`分支编译，应当能在原版MIUI和第三方的基于AOSP的各种Android11-14的ROM上使用。欢迎大家尝试并反馈（提Issue或Pull Requests）！酷友们到[酷安的这个帖子](https://www.coolapk.com/feed/56813047)讨论或反馈，也可以给我私信反馈！

注意：该内核的zip包不包含`dtbo.img`，并且不会刷你的dtbo分区。推荐使用原厂的`dtbo`，或者来自第三方系统包自带的dtbo（如果原作者确认那好用的话）。因为该源码build出来的`dtbo.img`有些小问题，比如在锁屏界面上尝试熄屏时，屏幕会突然闪一下到最高亮度。如果你刷过其它第三方内核，或者遇到一些奇怪的问题，建议检查一下你的`dtbo`是否被替换过。

度盘备用下载链接：https://pan.baidu.com/share/init?surl=11ocz7ggZ79gzRfWvsdbJA&pwd=ty58 （建议优先从Github Release下载）

欢迎加入内测QQ群: 459094061

支持的设备:
| 设备代号  | 设备名称                           |
|-----------|----------------------------------|
| psyche    | 小米12X                           |
| thyme     | 小米10S                           |
| umi       | 小米10                            |
| munch     | 红米K40S                          |
| lmi       | 红米K30 Pro                       |
| cmi       | 小米10 Pro                        |
| cas       | 小米10 Ultra                      |
| apollo    | 小米10T / 红米K30S Ultra          |
| alioth    | 小米11X / POCO F3 / 红米K40       |
| elish     | 小米平板5 Pro                     |
| enuma     | 小米平板5 Pro 5G                  |
| dagu      | 小米平板5 Pro 12.4                |
| pipa      | 小米平板6                         |

该内核的其他特性/改进:
1. 支持USB串口驱动（CH340/FTDI/PL2303/OTI6858/TI/SPCP8X5/QT2/UPD78F0730/CP210X）
2. 支持EROFS
3. F2FS开启了realtime discard以更好的TRIM闪存
4. 支持 CANBus 和 USB CAN （如 CANable）适配器（一些折腾嵌入式的可能会喜欢这个）
5. zRAM 支持 LZ4、LZ4HC、ZSTD 压缩算法

# 如何编译

1. 准备基础编译环境

你需要具备常用的基础工具链，例如 `git`、`make`、`curl`、`bison`、`flex`、`zip` 等，以及其他一些必要的软件包。

在 Debian/Ubuntu 系统中，你可以执行以下命令安装：

```bash
sudo apt install build-essential git curl wget bison flex zip bc cpio libssl-dev ccache
```

另外，还需要安装 `python`（仅有 `python3` 不够），你可以安装 `python-is-python3` 来解决：

```bash
sudo apt install python-is-python3
```

在 RHEL/RPM 系列系统中，你可以执行：

```bash
sudo yum groupinstall 'Development Tools'
sudo yum install wget bc openssl-devel ccache
```

注意：`build.sh` 脚本中启用了 `ccache` 以加快编译速度。`CCACHE_DIR` 已设置为 `$HOME/.cache/ccache_mikernel`，如有需要你可以修改或删除这个设置。

2. 下载 [proton-clang] 编译工具链

你需要具备 `aarch64-linux-gnu`、`arm-linux-gnueabi`、`clang` 等工具。[Proton Clang](https://github.com/kdrag0n/proton-clang/) 是一个很好的预编译 Clang 交叉编译工具链。

默认的工具链路径是 `$HOME/proton-clang/proton-clang-20210522/bin`，这个路径在 `build.sh` 中设置好了。如果你放在其他位置，请修改 `build.sh` 中的 `TOOLCHAIN_PATH`。

```bash
mkdir proton-clang
cd proton-clang
wget https://github.com/kdrag0n/proton-clang/archive/refs/tags/20210522.zip
unzip 20210522.zip
cd ..
```

3. 编译

不带 KernelSU 的编译命令：

```bash
bash build.sh 目标设备代号
```

带 KernelSU 的编译命令：

```bash
bash build.sh 目标设备代号 ksu
```

例如，为 lmi（Redmi K30 Pro / POCO F2 Pro）编译，不带 KernelSU：

```bash
bash build.sh lmi
```

例如，为 umi（小米10）编译，使用 [KernelSU](https://github.com/tiann/KernelSU)：

```bash
bash build.sh umi ksu
```

例如，为 umi（小米10）编译，使用 [RKSU](https://github.com/rsuntk/KernelSU)：

```bash
bash build.sh umi rksu
```

此外，还有一个 `buildall.sh` 脚本，可以一次性为所有支持的设备进行编译。
