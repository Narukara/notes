`Vec<T>` 允许存储多个相同类型的值，且在内存（堆）上顺序排列。

#### 创建

1. `new` 函数创建空 Vec

```rust
let v1: Vec<String> = Vec::new();
```

2. `vec!` 宏创建

```rust
let v2 = vec!['b', 'a', 'c'];	// Vec<char>
```

#### 更新

```rust
let mut v = Vec::new();
v.push(1);	// pub fn push(&mut self, value: T)
v.pop();	// pub fn pop(&mut self) -> Option<T>
```

还有很多方法，可以查API文档

#### 所有权

`Copy` 类型的变量会被拷贝进入 Vec。非 `Copy` 类型的变量会被移动进入 Vec，其所有权交给 Vec。

如果将值的引用插入 Vec，这些值本身将不会被移动进 Vec。但是这些引用指向的值必须至少在 Vec 有效时也是有效的。生命周期与引用有效性部分将会更多地讨论这个问题。

#### 访问

常用的两种方式：

1. 通过下标获取引用 `&v[n]`，越界会导致 `panic`
2. 通过 `get(n)` 方法获取引用，会得到 `Option`

```rust
let v = vec![String::from("hi")];
let s1 = &v[0];				// &mut 得到可变引用
let s2 = match v.get(0) {	// get_mut 得到可变引用
    Some(s) => s,
    None => panic!(),
};
```

注意遵循借用规则，下面的代码不能通过编译：

```rust
let mut v = vec![1, 2, 3];
let i = &v[0];		// 不可变引用
v.push(4);			// 可变引用
println!("{i}");	// 添加元素可能导致内存重新分配，使原引用失效
```

另外，也可以用 `v[n]` 的方式直接访问其中元素，但不允许将其中元素移动出来。

#### slice

Vec 可以转换为 slice

#### 遍历

1. 使用 `for i in &v` 得到不可变引用
2. 使用 `for i in &mut v` 得到可变引用
3. 使用 `for i in v` 隐式调用 `into_iter()`，会导致移动
