## libc

C 标准库有多种实现，如大多数 GNU/Linux 系统上的 `glibc`。`newlib` 是 RedHat 维护的针对嵌入式系统的 C 标准库。`arm-none-eabi-gcc` 工具链附带了 `newlib`。以及一个更轻量的变体 `newlib-nano`。位于 `/usr/arm-none-eabi/lib`：

| 文件               | 说明                                                   |
| ------------------ | ------------------------------------------------------ |
| `libc.a`           | newlib 库                                              |
| `libc_nano.a`      | newlib nano 库                                         |
| `librdimon.a`      | newlib 库 带有 semihosting                             |
| `librdimon_nano.a` | newlib nano 库 带有 semihosting                        |
| `*.specs`          | 在链接时使用的文件, 在链接参数里指定, 用于链接不同的库 |

使用时需要实现一些桩函数（System Calls）, 除非指定了 nosys.specs 或使用 semihosting 库

| 常用的链接参数     | 说明                |
| ------------------ | ------------------- |
| 无                 | 使用 newlib 库      |
| -specs=nano.specs  | 使用 newlib nano 库 |
| -specs=nosys.specs |                     |
| -nostdlib          | 不链接到 libc       |

### Semihosting

半主机功能, 允许 printf 等函数输出到 openocd 终端上

1. 需要使用特定的标准库（无需打桩）
2. 在程序中初始化半主机功能
3. 在 openocd 中启用半主机功能

### 桩函数（System Calls）

https://sourceware.org/newlib/libc.html#Syscalls 提供了 newlib 的桩函数列表和最小实现。

这些桩函数并不是全都要实现。链接时 ld 会提示缺少哪些函数，这主要取决于程序中用到了 libc 的哪些功能。

### 构造函数

可以给一些函数添加 `__attribute__((constructor))`，声明为构造函数。libc 会在 `__libc_init_array()` 中依次调用这些构造函数。`__libc_init_array()` 通常在 `Reset_Handler` 中执行，且先于 `main`。

例如：将 `_write` 实现为通过串口输出，为了保证串口可用，将初始化串口的函数声明为构造函数。

### 线程安全

大多数 newlib 函数是可重入的。对于其他函数，为了保证线程安全，OS 需要在上下文切换时正确地设置 `_impure_ptr` 变量，这个变量指向 `struct _reent` 结构体，其中保存了一个线程使用标准库的状态。

为了允许 `malloc` 可重入，需要实现 `malloc_lock` 和 `malloc_unlock`。通常建议使用递归互斥锁实现。

### 替换标准库函数

在自己的代码中定义与标准库函数签名相同的函数，可以替换掉标准库中对应的函数。