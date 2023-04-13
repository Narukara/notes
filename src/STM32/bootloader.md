## bootloader

### bootloader 的作用

- 从外部存储器（如 SPI flash）拷贝固件到可执行的区域（如 RAM）
- 解藕固件的各个部分（bootloader 和 app），通过 UART/USB/OTA 更新固件（app）
- 安全启动、验证加密签名、捕获 app 的 reboot loop 等

### 实现方法

1. 分割 flash，将 bootloader 和 app 定位到不同的区域

```
MEMORY
{
    bootrom (rx) : ORIGIN = 0x08000000, LENGTH = 16K
    approm (rx) : ORIGIN = 0x08004000, LENGTH = 112K
    ram (rwx) : ORIGIN = 0x20000000, LENGTH = 32K
}
```

- `bootrom` 里是 bootloader 的中断向量表、text、data
- `approm` 里是 app 的中断向量表、text、data
- `ram` 是共享的

2. 在 bootloader 中引导 app

```c
int main() {
    uint32_t* p = (uint32_t*)&__approm_start__;	// 获取 app 的中断向量表地址
    uint32_t msp = p[0];						// app 的 msp
    uint32_t entry = p[1];						// app 的 入口地址
    __asm("msr msp, %0;" ::"r"(msp)); 	 		// 设置 msp
    ((void (*)())entry)();             			// 跳转到 app 的 Reset_Handler
}
```

- bootloader 先启动（因为它的中断向量在 `0x08000000`），经过 `Reset_Handler` 进入 `main()`
- 在 bootloader 的 `main()` 里读 app 的中断向量表，引导 app 启动

3. 在 app 的 Reset_Handler 里重定位中断向量表，以便后续使用 app 的中断 Handler
4. bootloader 和 app 分别编译连接，可以分别下载，或者合并 bin 文件一起下载
