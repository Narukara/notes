## trait 对象

泛型类型参数一次只能替代一个具体类型，而 trait 对象则允许在运行时替代多种具体类型。

```rust
trait Show {
    fn show(&self) -> String;
}

fn show_all(v: Vec<Box<dyn Show>>) {	// trait 对象
    for i in v {
        println!("{}", i.show());
    }
}
```

trait bound 执行静态分发，trait 对象执行动态分发，即在运行时才能确定调用了什么方法。使用 trait 对象获得了额外的灵活性，但相比 trait bound 有性能损失

#### 对象安全

只有对象安全（object-safe）的 trait 才支持 trait 对象的使用。具体要求是：

- 返回值不是 `Self`
- 没有泛型类型的参数