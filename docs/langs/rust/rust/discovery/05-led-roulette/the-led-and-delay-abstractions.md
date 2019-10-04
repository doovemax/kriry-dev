# The LED与延迟抽象

现在，我将介绍两个高级抽象，我们将用它来实现LED轮盘应用程序。

辅助包，`aux5`，暴露了一个名为`init`的初始化函数。调用此函数时，返回在元组中打包的两个值：`Delay`值和`Leds`值。

`Delay`可用于阻止程序达到指定的毫秒数。

`Leds`实际上是八个`Led`的数组。每个`Led`代表F3板上的一个LED，并暴露两个方法：`on`和`off`，分别用于打开或关闭LED。

让我们通过修改起始代码来尝试这两个抽象，如下所示：

```rust
#![deny(unsafe_code)]
#![no_main]
#![no_std]

use aux5::{entry, prelude::*, Delay, Leds};

#[entry]
fn main() -> ! {
    let (mut delay, mut leds): (Delay, Leds) = aux5::init();

    let half_period = 500_u16;

    loop {
        leds[0].on();
        delay.delay_ms(half_period);

        leds[0].off();
        delay.delay_ms(half_period);
    }
}
```

现在构建它：

```bash
$ cargo build --target thumbv7em-none-eabihf
```

> **注意** 在启动GDB会话之前，可能会忘记重建程序;这种遗漏会导致调试会话非常混乱。为了避免这个问题，您可以调用 `cargo run` 而不是 `cargo build`;`cargo run`将构建并启动调试会话，确保您永远不会忘记重新编译您的程序。

现在，我们将重复上一节中所做的 flashing 过程：

```bash
$ # this starts a GDB session of the program; no need to specify the path to the binary
$ arm-none-eabi-gdb -q target/thumbv7em-none-eabihf/debug/led-roulette
Reading symbols from target/thumbv7em-none-eabihf/debug/led-roulette...done.
(gdb) target remote :3333
Remote debugging using :3333
(..)

(gdb) load
Loading section .vector_table, size 0x188 lma 0x8000000
Loading section .text, size 0x3fc6 lma 0x8000188
Loading section .rodata, size 0xa0c lma 0x8004150
Start address 0x8000188, load size 19290
Transfer rate: 19 KB/sec, 4822 bytes/write.

(gdb) break main
Breakpoint 1 at 0x800018c: file src/05-led-roulette/src/main.rs, line 9.

(gdb) continue
Continuing.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, main () at src/05-led-roulette/src/main.rs:9
9           let (mut delay, mut leds): (Delay, Leds) = aux5::init();
```

好。让我们逐条执行代码。这次，我们将使用`next`命令而不是`step`。区别在于`next`命令将跳过函数调用而不是进入它们内部。

```bash
(gdb) next
11          let half_period = 500_u16;

(gdb) next
13          loop {

(gdb) next
14              leds[0].on();

(gdb) next
15              delay.delay_ms(half_period);
```

执行 `leds[0].on()` 语句后，您应该看到一个红色LED指示灯，指向北，亮起。

让我们继续执行这个程序：

```bash
(gdb) next
17              leds[0].off();

(gdb) next
18              delay.delay_ms(half_period);
```

`delay_ms` 调用将阻止程序半秒，但您可能不会注意到，因为`next`命令也需要一些时间来执行。但是，在执行`leds[0].off()`语句后，您应该看到红色LED熄灭。

你已经可以猜出这个程序的作用了。使用continue命令让它不间断地运行。

```bash
(gdb) continue
Continuing.
```

现在，让我们做一些更有趣的事情。我们将使用GDB修改程序的行为。

首先，让我们通过按`Ctrl+C`来停止无限循环。你可能会在`Led::on`、`Led::off` 或 `delay_ms`中的某个地方结束：

```bash
Program received signal SIGINT, Interrupt.
0x080033f6 in core::ptr::read_volatile (src=0xe000e010) at /checkout/src/libcore/ptr.rs:472
472     /checkout/src/libcore/ptr.rs: No such file or directory.
```

在我的例子中，程序在read_volatile函数内停止执行。 GDB输出显示了一些有趣的信息：`core::ptr::read_volatile (src=0xe000e010)`。这意味着该函数来自 `core` crate，并且使用参数 `src = 0xe000e010` 调用它。

您知道，显示函数参数的更明确的方法是使用info args命令：

```bash
(gdb) info args
src = 0xe000e010
```

无论你的程序停止在哪里，你都可以随时查看 `backtrace` 命令的输出（简称`bt`）来了解它是如何到达的：

```bash
(gdb) backtrace
#0  0x080033f6 in core::ptr::read_volatile (src=0xe000e010)
    at /checkout/src/libcore/ptr.rs:472
#1  0x08003248 in <vcell::VolatileCell<T>>::get (self=0xe000e010)
    at $REGISTRY/vcell-0.1.0/src/lib.rs:43
#2  <volatile_register::RW<T>>::read (self=0xe000e010)
    at $REGISTRY/volatile-register-0.2.0/src/lib.rs:75
#3  cortex_m::peripheral::syst::<impl cortex_m::peripheral::SYST>::has_wrapped (self=0x10001fbc)
    at $REGISTRY/cortex-m-0.5.7/src/peripheral/syst.rs:124
#4  0x08002d9c in <stm32f30x_hal::delay::Delay as embedded_hal::blocking::delay::DelayUs<u32>>::delay_us (self=0x10001fbc, us=500000)
    at $REGISTRY/stm32f30x-hal-0.2.0/src/delay.rs:58
#5  0x08002cce in <stm32f30x_hal::delay::Delay as embedded_hal::blocking::delay::DelayMs<u32>>::delay_ms (self=0x10001fbc, ms=500)
    at $REGISTRY/stm32f30x-hal-0.2.0/src/delay.rs:32
#6  0x08002d0e in <stm32f30x_hal::delay::Delay as embedded_hal::blocking::delay::DelayMs<u16>>::delay_ms (self=0x10001fbc, ms=500)
    at $REGISTRY/stm32f30x-hal-0.2.0/src/delay.rs:38
#7  0x080001ee in main () at src/05-led-roulette/src/main.rs:18
```

`backtrace`将会打印一系列函数调用，从当前函数直到main。

回到我们的主题。为了做我们想要的事情，首先，我们必须返回到`main`函数。我们可以使用`finish`命令来做到这一点。该命令恢复程序执行并在程序从当前函数返回后立即停止。我们需要调用他多次。

```bash
(gdb) finish
cortex_m::peripheral::syst::<impl cortex_m::peripheral::SYST>::has_wrapped (self=0x10001fbc)
    at $REGISTRY/cortex-m-0.5.7/src/peripheral/syst.rs:124
124             self.csr.read() & SYST_CSR_COUNTFLAG != 0
Value returned is $1 = 5

(gdb) finish
Run till exit from #0  cortex_m::peripheral::syst::<impl cortex_m::peripheral::SYST>::has_wrapped (
    self=0x10001fbc)
    at $REGISTRY/cortex-m-0.5.7/src/peripheral/syst.rs:124
0x08002d9c in <stm32f30x_hal::delay::Delay as embedded_hal::blocking::delay::DelayUs<u32>>::delay_us (
    self=0x10001fbc, us=500000)
    at $REGISTRY/stm32f30x-hal-0.2.0/src/delay.rs:58
58              while !self.syst.has_wrapped() {}
Value returned is $2 = false

(..)

(gdb) finish
Run till exit from #0  0x08002d0e in <stm32f30x_hal::delay::Delay as embedded_hal::blocking::delay::DelayMs<u16>>::delay_ms (self=0x10001fbc, ms=500)
    at $REGISTRY/stm32f30x-hal-0.2.0/src/delay.rs:38
0x080001ee in main () at src/05-led-roulette/src/main.rs:18
18              delay.delay_ms(half_period);
```

我们又回到了`main`。我们在这里有一个局部变量：half_period

```bash
(gdb) info locals
half_period = 500
delay = (..)
leds = (..)
```

现在，我们将使用set命令修改此变量：

```bash
(gdb) set half_period = 100

(gdb) print half_period
$1 = 100
```

如果让程序使用`continue`命令再次运行，您应该会看到LED将以更快的速度闪烁！

问题来了！如果你继续降低 `half_period` 的值会怎么样？在 `half_period` 的值是多少的时候你再也看不到LED闪烁了？

现在，轮到你写一个程序了。
