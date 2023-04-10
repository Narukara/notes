## unsafe Rust

在 `unsafe` 块中，允许进行这五类特殊操作：

- 解引用裸指针
- 调用不安全的函数或方法
- 访问或修改可变静态变量
- 实现不安全 trait
- 访问 `union` 的字段

注意：`unsafe` 并不会关闭借用检查器或禁用任何其他 Rust 安全检查

创建不安全代码的安全抽象是 Rust 中一种重要的设计模式。将底层的、Rust 编译器不能理解的操作在 `unsafe` 块中实现，封装为安全的函数等抽象。

#### 裸指针

类型为 `*const T` 或 `*mut T`，分别是（底层）不可变或可变的。裸指针允许忽略借用规则、不保证指向有效的内存、允许为空、不能实现任何自动清理功能

```rust
let mut a = 10;
let p: *mut i32 = &mut a;	// 通过引用创建裸指针
let p = 0x0800_0000 as *mut i32;	// 通过地址创建裸指针
```

裸指针可以在安全代码中创建，但只能在 `unsafe` 中解引用

#### 调用不安全函数

```rust
unsafe fn dangerous() {}
```

调用不安全函数需要在 `unsafe` 块中进行。不安全函数内部看作 `unsafe` 块，可以在其中进行各种 `unsafe` 操作

#### 外部函数接口

为了与其他语言编写的代码交互，可以通过外部函数接口（FFI）。外部函数总被认为是不安全的：

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}
```

`"C"` 定义了外部函数所使用的 应用二进制接口（ABI）。ABI 定义了如何在汇编语言层面调用此函数。`"C"` ABI 是最常见的，并遵循 C 编程语言的 ABI。

另外，可以将 Rust 函数提供给其他语言使用：

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

`#[no_mangle]` 注解要求 Rust 编译器不要 mangle 此函数的名称

#### 静态变量

静态变量即全局变量：

```rust
static MY_NAME: String = String::new();	// 不可变的
static mut AGE: u32 = 1;	// 可变的
```

访问不可变静态变量是安全的，任何对可变静态变量的访问都是不安全的

#### 不安全 trait

当 trait 中至少有一个方法中包含编译器无法验证的不变式（invariant）时 trait 是不安全的

```rust
unsafe trait Foo {...}
unsafe impl Foo for i32 {...}
```

#### 访问联合体

联合体 `union` 主要用于和 C 代码中的联合体交互，访问联合体的字段是不安全的