### Clib

位于 /usr/arm-none-eabi/lib

| 文件               | 说明                                                   |
| ------------------ | ------------------------------------------------------ |
| `libc.a`           | newlib 库                                              |
| `libc_nano.a`      | newlib nano 库                                         |
| `librdimon.a`      | newlib 库 带有 semihosting                             |
| `librdimon_nano.a` | newlib nano 库 带有 semihosting                        |
| `*.specs`          | 在链接时使用的文件, 在链接参数里指定, 用于链接不同的库 |

使用时需要实现一些桩函数, 除非指定了 nosys.specs 或使用 semihosting 库

### Semihosting

半主机功能, 允许 printf 等函数输出到 openocd 终端上

1. 需要使用特定的标准库 (无需打桩)
2. 在程序中初始化半主机功能
3. 在 openocd 中启用半主机功能