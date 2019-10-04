# 概叙

处理微控制器涉及多种工具，因为我们将处理与您的笔记本电脑不同的架构，我们将不得不在“远程”设备上运行和调试程序。

文档
工具并不是一切。没有文档，几乎不可能使用微控制器。

我们将在本书中提到所有这些文件：

HEADS UP所有这些链接指向PDF文件，其中一些长度为数百页，大小为几MB。

* [STM32F3DISCOVERY用户手册](http://www.st.com/resource/en/user_manual/dm00063382.pdf)
* [STM32F303VC数据表](https://www.st.com/resource/en/datasheet/stm32f303vc.pdf)
* [STM32F303VC参考手册](https://www.st.com/content/ccc/resource/technical/document/reference_manual/4a/19/6e/18/9d/92/43/32/DM00043574.pdf/files/DM00043574.pdf/jcr:content/translations/en.DM00043574.pdf)
* [LSM303DLHC](https://www.st.com/resource/en/datasheet/lsm303dlhc.pdf)
* [L3GD20](https://www.st.com/resource/en/datasheet/l3gd20.pdf)
  
## 工具

我们将使用下面列出的所有工具。如果未指定最低版本，则任何最新版本都应该有效，但我们已经列出了我们测试过的版本。

Rust 1.30,1.30-beta，nightly-2018-09-13，或更新的工具链。

itmdump v0.2.1

OpenOCD> = 0.8。经测试的版本：v0.9.0和v0.10.0

arm-none-eabi-gdb。版本7.12或更高版本强烈推荐。经测试的版本：7.10,7.11,7.12和8.1

cargo-binutils。版本0.1.4或更新版本。

minicom在Linux和macOS上。测试版本：2.7。读者报告picocom也有效，但我们将minicom在本文中使用。

PuTTY 在Windows上。

如果您的笔记本电脑具有蓝牙功能并且您具有蓝牙模块，则可以另外安装这些工具以使用蓝牙模块。所有这些都是可选的：

Linux，只有你没有像Blueman这样的蓝牙管理器应用程序。

* bluez
* hcitool
* rfcomm
* rfkill

macOS / OSX / Windows用户只需要操作系统附带的默认蓝牙管理器。

接下来，按照与操作系统无关的安装说明获取一些工具：

## rustc 和cargo

按照https://rustup.rs上的说明安装rustup 。

然后，安装或切换到测试版频道。

$ rustup default beta
注意确保您的工具链比新工具链更新1.30-beta.1。rustc -V 应该返回比下面显示的日期更新的日期：

$ rustc -V
rustc 1.30.0-beta.1 (14f51b05d 2018-09-18)
itmdump
 
$ cargo install itm --vers 0.3.1

$ itmdump -V
itmdump 0.3.1
cargo-binutils

$ rustup component add llvm-tools-preview

$ cargo install cargo-binutils --vers 0.1.4

$ cargo size -- -version
LLVM (http://llvm.org/):
  LLVM version 8.0.0svn
  Optimized build.
  Default target: x86_64-unknown-linux-gnu
  Host CPU: skylake

## OS特定说明

现在按照您正在使用的操作系统的具体说明进行操作：

* [Linux](https://rust-embedded.github.io/03-setup/linux.html)
* [windows](https://rust-embedded.github.io/03-setup/windows.html)
* [macOS](https://rust-embedded.github.io/03-setup/macos.html)