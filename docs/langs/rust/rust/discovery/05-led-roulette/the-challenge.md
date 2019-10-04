# 挑战

你现在已经可以面对挑战了！您的任务是实现我在本章开头向您展示的应用程序。

这是GIF：

![LED轮盘](https://i.imgur.com/0k1r2Lc.gif)

此外，这可能有所帮助：

![时序图](https://rust-embedded.github.io/discovery/assets/timing-diagram.png)

这是一个时序图。它指示在任何给定时刻哪个LED亮起以及每个LED应该打开多长时间。在X轴上，我们有时间，以毫秒为单位。时序图显示了一个周期。这种模式每800毫秒重复一次。 Y轴标记每个LED有一个指向：北，东等。作为挑战的一部分，您必须弄清楚 `Leds` 数组中的每个元素如何映射到这些指向（提示：`cargo doc --open` `;-)`）。

在你尝试这个挑战之前，让我给你最后一个提示。我们的GDB会话总是在开头输入相同的命令。我们可以使用`.gdb`文件在GDB启动后立即执行一些命令。这样，您可以节省在每个GDB会话上手动输入它们的工作量。

将此 `openocd.gdb` 文件放在 Cargo 项目的根目录中，紧挨着 `Cargo.toml`：

```bash
$ cat openocd.gdb

target remote :3333
load
break main
continue
```

然后修改 `.cargo/config` 文件的第二行：

```bash
$ cat .cargo/config

[target.thumbv7em-none-eabihf]
runner = "arm-none-eabi-gdb -q -x openocd.gdb" # <-
rustflags = [
  "-C", "link-arg=-Tlink.x",
]
```

有了这个，你现在应该能够启动一个`gdb`会话，它会自动刷写程序并跳转到`main`的开头：

```bash
$ cargo run --target thumbv7em-none-eabihf
     Running `arm-none-eabi-gdb -q -x openocd.gdb target/thumbv7em-none-eabihf/debug/led-roulette`
Reading symbols from target/thumbv7em-none-eabihf/debug/led-roulette...done.
(..)
Loading section .vector_table, size 0x188 lma 0x8000000
Loading section .text, size 0x3b20 lma 0x8000188
Loading section .rodata, size 0xb0c lma 0x8003cc0
Start address 0x8003b1c, load size 18356
Transfer rate: 20 KB/sec, 6118 bytes/write.
Breakpoint 1 at 0x800018c: file src/05-led-roulette/src/main.rs, line 9.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, main () at src/05-led-roulette/src/main.rs:9
9           let (mut delay, mut leds): (Delay, Leds) = aux5::init();
(gdb)
```
