## Trait

#### 定义

trait 类似其他编程语言中的接口

```rust
trait Show {
    fn show(&self) -> String;
}
```

```rust
trait Comparable {
    fn bigger_than(&self, b: &Self) -> bool;	// trait impl 到什么类型上，Self 就是什么类型
}
```

可以定义方法的默认实现：

```rust
trait Show {
    fn show(&self) -> String {
        String::new()
    }
}
```

可以在一个 trait 中定义多个方法，它们之间可以互相调用。

#### 为类型实现 trait

```rust
struct Point {
    x: i32,
    y: i32,
}

impl Show for Point {
    fn show(&self) -> String {
        format!("({}, {})", self.x, self.y)
    }
}
```

为类型实现 trait 时，要么类型，要么 trait 要位于本地作用域内。不允许为外部类型实现外部 trait，例如在自己的程序中为标准库类型 `Vec<T>` 实现标准库 trait `Display`。这个限制称为“相干性”或“孤儿规则”。

甚至可以对泛型实现 trait。例如，标准库对所有 Display 的类型实现了 ToString

```rust
impl<T: Display> ToString for T {
    ...
}
```

```rust
impl<T: PartialOrd> Comparable for T {
    fn bigger_than(&self, b: &Self) -> bool {
        self > b
    }
}
```

#### trait 参数/返回值

用 trait 限定函数泛型参数的类型，以便于在函数中调用 trait 提供的方法。有两种语法：

1. `impl Trait` 语法

```rust
fn fun(item: &impl Show) {
    println!("{}", item.show());
}
```

也可以在返回值上使用

2. `trait` bound 语法

```rust
fn fun<T: Show>(item: &T) {
    println!("{}", item.show());
}
```

注意：这种语法通常不能在返回值上使用，原因见 `泛型.md`

指定多个 trait 时，用加号连接：

```rust
fn fun(item: &(impl Show + Display))
fn fun<T: Show + Display>(item: &T)
```

另外，可以用 `where` 语法避免过长的签名

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
...
```

注意：这里是通过“单态化”实现的，函数静态分发，泛型类型参数一次只能替代一个具体类型。

---

## 高级特性

#### 关联类型

关联类型是一个占位符类型，例如：

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

关联类型与泛型参数的区别在于：trait 有泛型参数时，在同一个类型上，可以多次以不同泛型类型实现这个 trait，且在调用 trait 方法的时候也不得不加上类型注解。

#### 运算符重载

可以重载 `std::ops` 中列出的运算符，例如加法：

```rust
impl Add for Point {
    type Output = Point;
    fn add(self, rhs: Self) -> Self::Output {
        Self {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
```

#### 默认泛型类型参数

定义 trait 时可以为泛型参数指定默认类型，例如：

```rust
pub trait Add<Rhs = Self> {		// 默认情况下，加法运算实现在两个相同的类型上
    type Output;
    fn add(self, rhs: Rhs) -> Self::Output;
}
```

#### 完全限定语法和消歧义

1. 如果 trait 和类型上的方法重名，可以通过以下语法消歧义（也可以使用后面的完全限定语法）：

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {..}

impl Wizard for Human {..}

impl Human {
    fn fly(&self) {..}
}
```

```rust
let person = Human;
Pilot::fly(&person);
Wizard::fly(&person);
person.fly();
```

2. 如果 trait 和类型上的关联函数重名，可以使用完全限定语法：

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {..}
}

impl Animal for Dog {..}
```

```rust
Dog::baby_name()
<Dog as Animal>::baby_name()
```

#### 父 trait

可以定义基于其他 trait 的 trait。例如定义一个 `OutlinePrint` trait，基于 `Display` trait，提供 `outline_print` 方法，在打印值的前后加上星号，那么只有在实现了 `Display` 的类型上才能实现 `OutlinePrint`

```rust
trait OutlinePrint: Display {
    fn outline_print(&self) {
        let output = self.to_string();
        println!("* {} *", output);
    }
}
```

#### newtype 模式

为了在外部类型上实现外部 trait，可以使用一种叫 newtype 模式的设计方法。即用元组结构体包装目标类型，然后在新类型上实现：

```rust
struct Wrapper(Vec<String>);
impl fmt::Display for Wrapper {..}
```

这样做的缺点是新类型不能直接使用原类型上的各种方法。可以考虑使用 `Deref` trait 解决这个问题。
