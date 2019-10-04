# 验证安装

仅限Linux
验证权限
使用USB线将F3连接到笔记本电脑。务必将电缆连接到“USB ST-LINK”端口，即电路板边缘中心的USB端口。

F3现在应该显示为USB设备（文件）/dev/bus/usb。让我们看看它是如何枚举的：


$ lsusb | grep -i stm
Bus 003 Device 004: ID 0483:374b STMicroelectronics ST-LINK/V2.1
$ # ^^^        ^^^
在我的例子中，F3连接到＃3总线并被枚举为设备＃4。这意味着该文件/dev/bus/usb/003/004 是 F3。我们来看看它的权限：


$ ls -l /dev/bus/usb/003/004
crw-rw-rw- 1 root root 189, 20 Sep 13 00:00 /dev/bus/usb/003/004
权限应该是crw-rw-rw-。如果不是......那么检查你的udev规则并尝试重新加载它们：


$ sudo udevadm control --reload-rules
现在让我们重复串行模块的过程。

拔下F3并插入串行模块。现在，弄清楚它的相关文件是什么：


$ lsusb | grep -i ft232
Bus 003 Device 005: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
就我而言，就是这样/dev/bus/usb/003/005。现在，检查其权限：


$ ls -l /dev/bus/usb/003/005
crw-rw-r-- 1 root root 189, 21 Sep 13 00:00 /dev/bus/usb/003/005
和以前一样，权限应该是crw-rw-r--。

所有
第一个OpenOCD连接
首先，使用USB电缆将F3连接到笔记本电脑。将电缆连接到电路板边缘中心的USB端口，标有“USB ST-LINK”的端口。

将USB电缆连接到电路板后，应立即打开两个红色 LED指示灯。

接下来，运行以下命令：


$ # *nix
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg

$ # Windows
$ # NOTE cygwin users have reported problems with the -s flag. If you run into
$ # that you can call openocd from the `C:\OpenOCD\share\scripts` directory
$ openocd -s C:\OpenOCD\share\scripts -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
注意 Windows用户：C:\OpenOCD是您将OpenOCD安装到的目录。

重要事项 STM32F3DISCOVERY板有多个硬件版本。对于较旧的版本，您需要将“interface”参数更改为-f interface/stlink-v2.cfg（注意：-1结尾处的否）。或者，可以使用较旧的修订版-f board/stm32f3discovery.cfg 代替-f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg。

你应该看到这样的输出：


Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use '`transport select <transport>`'.
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
none separate
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : clock speed 950 kHz
Info : STLINK v2 JTAG v27 API v2 SWIM v15 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 2.915608
Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
（如果您不...则查看一般故障排除说明。）

openocd将阻止终端。没关系。

此外，其中一个红色LED（最靠近USB端口的LED）应开始在红灯和绿灯之间振荡。

而已！有用。你现在可以关闭/杀死openocd。