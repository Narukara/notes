`HashMap<K, V>` 以键值对的形式将数据存储在堆上。

#### 创建

`HashMap` 没有被 prelude 自动引用，需要 `use` 导入

```rust
use std::collections::HashMap;
```

1. `new` 方法创建

```rust
let mut m = HashMap::new();
```

2. 在键值对元组的 vector 上使用迭代器和 `collect` 方法

```rust
let teams = vec!["Blue".to_string(), "Yellow".to_string()];
let initial_scores = vec![10, 50];

let mut scores: HashMap<_, _> = teams.into_iter().zip(initial_scores.into_iter()).collect();
```

`collect` 可以输出很多种数据结构，因此 `HashMap<_, _>` 类型注解是必须的，但是键值对的类型可以用下划线占位，编译器能推断出。

#### 更新

1. `insert` 插入键值对，如果已存在则覆盖

```rust
m.insert("hi".to_string(), 10);
```

2. 只在不存在时插入

```rust
m.entry("hi".to_string()).or_insert(10);
```

`entry` 方法返回 `Entry` 枚举，表示可能存在的值，其上的 `or_insert` 方法在值不存在时将参数作为值插入，并返回值的可变引用

3. 根据旧值更新一个值

```rust
/* 统计单词个数 */
let text = "hello world wonderful world";
let mut map = HashMap::new();
for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}
```

#### 访问

使用 `get` 方法，返回 `Option<V>`

```rust
let v = m.get(&String::from("hi"));
```

#### 遍历

```rust
for (k, v) in &m {
    // ...
}
```

使用 `&mut m` 后，仅 `v` 会是可变引用。直接使用 `m` 会导致 `m` 被移动走。遍历的顺序是随机的。
