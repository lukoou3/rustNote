
In order to increase fluency in a programming language, one has to read a lot of it.

要想提高一门编程语言的熟练度，就必须大量阅读相关内容。

But how can you read a lot of it if you don’t know what it means?

但如果你都不知道它是什么意思，又怎么能大量阅读呢？

In this article, instead of focusing on one or two concepts, I’ll try to go through as many Rust snippets as I can, and explain what the keywords and symbols they contain mean.

在本文中，我不会只关注一两个概念，而是会尽可能多地讲解一些Rust代码片段，并解释其中包含的关键字和符号的含义。

Ready? Go!

准备好了吗？开始！

## Variable bindings(变量绑定)

### The let keyword(let关键字)

`let`introduces a variable binding:

`let`引入了变量绑定：

```rust
let x; // declare "x"
x = 42; // assign 42 to "x"
```

This can also be written as a single line:

这也可以写成一行：

```rust
let x = 42;
```

### Type annotation(类型注解)

You can specify the variable’s type explicitly with`:`, that’s a type annotation:

你可以使用`:`来显式指定变量的类型，这就是类型注解：

```rust
let x: i32; // `i32` is a signed 32-bit integer
x = 42;

// there's i8, i16, i32, i64, i128
//    also u8, u16, u32, u64, u128 for unsigned
```

This can also be written as a single line:

这也可以写成一行：

```rust
let x: i32 = 42;
```

### Uninitialized variables(未初始化的变量)

If you declare a name and initialize it later, the compiler will prevent you from using it before it’s initialized.

如果你声明了一个变量名但稍后才初始化它，编译器会阻止你在它初始化之前使用该变量。

```rust
let x;
foobar(x); // error: borrow of possibly-uninitialized variable: `x`
x = 42;
```

However, doing this is completely fine:

但是，这样做是完全没问题的：

```rust
let x;
x = 42;
foobar(x); // the type of `x` will be inferred from here
```

### Throwing values away(丢弃值)

The underscore `_` is a special name - or rather, a “lack of name”. It basically means to throw away something:

下划线`_`是一个特殊的名称——或者更确切地说，是“没有名称”。它基本上意味着丢弃某些东西：

```rust
// this does *nothing* because 42 is a constant
let _ = 42;

// this calls `get_thing` but throws away its result
let _ = get_thing();
```

Names that start with an underscore are regular names, it’s just that the compiler won’t warn about them being unused:

以下划线开头的名称是常规名称，只是编译器不会就它们未被使用发出警告：

```rust
// we may use `_x` eventually, but our code is a work-in-progress
// and we just wanted to get rid of a compiler warning for now.
let _x = 42;
```

### Shadowing bindings(绑定遮蔽)

Separate bindings with the same name can be introduced - you can shadow a variable binding:

可以引入具有相同名称的单独绑定——你可以遮蔽变量绑定：

```rust
let x = 13;
let x = x + 3;
// using `x` after that line only refers to the second `x`,
//
// although the first `x` still exists (it'll be dropped
// when going out of scope), you can no longer refer to it.
```

## Tuples(元组)

Rust has tuples, which you can think of as “fixed-length collections of values of different types”.

Rust 有元组，你可以将其视为“不同类型值的固定长度集合”。

```rust
let pair = ('a', 17);
pair.0; // this is 'a'
pair.1; // this is 17
```

If we really wanted to annotate the type of `pair`, we would write:

如果我们确实想标注`pair`的类型，我们会这样写：

```rust
let pair: (char, i32) = ('a', 17);
```

### Destructuring tuples(解构元组)

Tuples can be destructured when doing an assignment, which means they’re broken down into their individual fields:

在进行赋值时，可以对元组进行解构，这意味着会将元组拆分为其各个字段：

```rust
let (some_char, some_int) = ('a', 17);
// now, `some_char` is 'a', and `some_int` is 17
```

This is especially useful when a function returns a tuple:

当函数返回元组时，这一点尤其有用：

```rust
let (left, right) = slice.split_at(middle);

```

Of course, when destructuring a tuple, `_` can be used to throw away part of it:

当然，在解构元组时，`_`可用于舍弃其中的部分内容：

```rust
let (_, right) = slice.split_at(middle);

```

## Statements(语句)

The semi-colon marks the end of a statement:

分号标志着一个语句的结束：

```rust
let x = 3;
let y = 5;
let z = y + x;
```

Which means statements can span multiple lines:

这意味着语句可以跨多行：

```rust
let x = vec![1, 2, 3, 4, 5, 6, 7, 8]
    .iter()
    .map(|x| x + 3)
    .fold(0, |x, y| x + y);
```

(We’ll go over what those actually mean later).

（我们稍后会详细解释这些的实际含义）。

## Functions(函数)

`fn` declares a function. Here’s a void function:

`fn` 用于声明函数。这是一个无返回值函数：

```rust
fn greet() {
    println!("Hi there!");
}
```

And here’s a function that returns a 32-bit signed integer. The arrow indicates its return type:

这是一个返回32位有符号整数的函数。箭头指示了它的返回类型：

```rust
fn fair_dice_roll() -> i32 {
    4
}
```

## Blocks(块)

A pair of brackets declares a block, which has its own scope:

一对括号声明一个块，该块有自己的作用域：

```rust
// This prints "in", then "out"
fn main() {
    let x = "out";
    {
        // this is a different `x`
        let x = "in";
        println!("{}", x);
    }
    println!("{}", x);
}
```

### Blocks are expressions(块是表达式)

Blocks are also expressions, which mean they evaluate to a value.

代码块也是表达式，这意味着它们会计算出一个值。

```rust
// this:
let x = 42;

// is equivalent to this:
let x = { 42 };
```

Inside a block, there can be multiple statements:

一个代码块中可以包含多个语句：

```rust
let x = {
    let y = 1; // first statement
    let z = 2; // second statement
    y + z // this is the *tail* - what the whole block will evaluate to
};
```

### Implicit return(隐式返回)

And that’s why “omitting the semicolon at the end of a function” is the same as returning, ie. these are equivalent:

这就是为什么“省略函数末尾的分号”等同于返回，也就是说，以下两种写法是等效的：

```rust
fn fair_dice_roll() -> i32 {
    return 4;
}

fn fair_dice_roll() -> i32 {
    4
}
```

### 万物皆表达式(万物皆表达式)

`if` conditionals are also expressions:

`if`条件语句也是表达式：

```rust
fn fair_dice_roll() -> i32 {
    if feeling_lucky {
        6
    } else {
        4
    }
}
```

A `match` is also an expression:

匹配也是一种表达式：

```rust
fn fair_dice_roll() -> i32 {
    match feeling_lucky {
        true => 6,
        false => 4,
    }
}
```

## Field access and method calling(字段访问和方法调用)

Dots are typically used to access fields of a value:

点通常用于访问值的字段：

```rust
let a = (10, 20);
a.0; // this is 10

let amos = get_some_struct();
amos.nickname; // this is "fasterthanlime"
```

Or call a method on a value:

或者对一个值调用方法：

```rust
let nick = "fasterthanlime";
nick.len(); // this is 14
```

## Modules, use syntax(模块，use语法)

The double-colon,`::`, is similar but it operates on namespaces.

双冒号，`::`，与之类似，但它作用于命名空间。

In this example, `std` is acrate(a library), `cmp` is amodule(a source file), and `min` is afunction:

在这个示例中，`std`是一个crate（一个库），`cmp`是一个模块（一个源文件），而`min`是一个函数：

```rust
let least = std::cmp::min(3, 8); // this is 3
```

`use` directives can be used to “bring in scope” names from other namespace:

`use`指令可用于从其他命名空间“引入作用域”名称：

```rust
use std::cmp::min;

let least = min(7, 1); // this is 1
```

Within `use` directives, curly brackets have another meaning: they’re “globs”. If we want to import both `min` and `max`, we can do any of these:

在`use`指令中，花括号有另一种含义：它们是“通配符”。如果我们想同时导入`min`和`max`，可以采用以下任意一种方式：

```rust
// this works:
use std::cmp::min;
use std::cmp::max;

// this also works:
use std::cmp::{min, max};

// this also works!
use std::{cmp::min, cmp::max};
```

A wildcard (`*`) lets you import every symbol from a namespace:

通配符（`*`）允许你从命名空间导入所有符号：

```rust
// this brings `min` and `max` in scope, and many other things
use std::cmp::*;
```

### Types are namespaces too(类型也是命名空间)

Types are namespaces too, and methods can be called as regular functions:

类型也是命名空间，并且方法可以作为常规函数来调用：

```rust
let x = "amos".len(); // this is 4
let x = str::len("amos"); // this is also 4
```

### The libstd prelude(标准库预导入)

`str` is a primitive type, but many non-primitive types are also in scope by default.

`str`是一种基本类型，但默认情况下许多非基本类型也在作用域内。

```rust
// `Vec` is a regular struct, not a primitive type
let v = Vec::new();

// this is exactly the same code, but with the *full* path to `Vec`
let v = std::vec::Vec::new();
```

This works because Rust inserts this at the beginning of every module:

这之所以有效，是因为Rust会在每个模块的开头插入这个：

```rust
use std::prelude::v1::*;
```

(Which in turns re-exports a lot of symbols, like`Vec`,`String`,`Option`and`Result`).

（这进而会重新导出许多符号，例如`Vec`、`String`、`Option`和`Result`）。


## Structs(结构体)

Structs are declared with the `struct` keyword:

结构体使用`struct`关键字声明：

```rust
struct Vec2 {
    x: f64, // 64-bit floating point, aka "double precision"
    y: f64,
}
```

They can be initialized usingstruct literals:

它们可以使用结构体字面量进行初始化：

```rust
let v1 = Vec2 { x: 1.0, y: 3.0 };
let v2 = Vec2 { y: 2.0, x: 4.0 };
// the order does not matter, only the names do
```

### Struct update syntax(结构体更新语法)

There is a shortcut for initializing the rest of the fields from another struct:

从另一个结构体初始化其余字段有一个快捷方式：

```rust
let v3 = Vec2 {
    x: 14.0,
    ..v2
};
```

This is called “struct update syntax”, can only happen in last position, and cannot be followed by a comma.

这被称为“结构体更新语法”，只能出现在最后一个位置，且后面不能跟逗号。

Note that the rest of the fields can mean all the fields:

请注意，其余字段可能指的是所有字段：

```rust
let v4 = Vec2 { ..v3 };
```

### Destructuring structs(解构结构体)

Structs, like tuples, can be destructured.

结构体和元组一样，可以被解构。

Just like this is a valid let pattern:

就像这是一个有效的let模式：

```rust
let (left, right) = slice.split_at(middle);
```

So is this:So is this:

这也是如此：

```rust
let v = Vec2 { x: 3.0, y: 6.0 };
let Vec2 { x, y } = v;
// `x` is now 3.0, `y` is now `6.0`
```

And this:And this:

还有这个：

```rust
let Vec2 { x, .. } = v;
// this throws away `v.y`
```

## Patterns and destructuring(模式与解构)

### Destructuring with if let(使用 if let 进行解构)

`let` patterns can be used as conditions in `if`:

`let`模式可用作`if`中的条件：

```rust
struct Number {
    odd: bool,
    value: i32,
}

fn main() {
    let one = Number { odd: true, value: 1 };
    let two = Number { odd: false, value: 2 };
    print_number(one);
    print_number(two);
}

fn print_number(n: Number) {
    if let Number { odd: true, value } = n {
        println!("Odd number: {}", value);
    } else if let Number { odd: false, value } = n {
        println!("Even number: {}", value);
    }
}

// this prints:
// Odd number: 1
// Even number: 2
```

### Match arms are patterns(匹配分支是模式)

`match` arms are also patterns, just like `if let`:

`match`分支也是模式，就像`if let`一样：

```rust
fn print_number(n: Number) {
    match n {
        Number { odd: true, value } => println!("Odd number: {}", value),
        Number { odd: false, value } => println!("Even number: {}", value),
    }
}

// this prints the same as before
```

### Exhaustive matches(穷尽匹配)

A `match` has to be exhaustive: at least one arm needs to match.

match 必须是穷尽的：至少有一个分支需要匹配。

```rust
fn print_number(n: Number) {
    match n {
        Number { value: 1, .. } => println!("One"),
        Number { value: 2, .. } => println!("Two"),
        Number { value, .. } => println!("{}", value),
        // if that last arm didn't exist, we would get a compile-time error
    }
}
```

If that’s hard, `_` can be used as a “catch-all” pattern:

如果这很难，`_`可以用作“万能”模式：

```rust
fn print_number(n: Number) {
    match n.value {
        1 => println!("One"),
        2 => println!("Two"),
        _ => println!("{}", n.value),
    }
}
```

## Methods(方法)

You can declare methods on your own types:

你可以在自己的类型上声明方法：

```rust
struct Number {
    odd: bool,
    value: i32,
}

impl Number {
    fn is_strictly_positive(self) -> bool {
        self.value > 0
    }
}
```

And use them like usual:

并且像往常一样使用它们：

```rust
fn main() {
    let minus_two = Number {
        odd: false,
        value: -2,
    };
    println!("positive? {}", minus_two.is_strictly_positive());
    // this prints "positive? false"
}
```

## Immutability(不可变性)

Variable bindings are immutable by default, which means their interior can’t be mutated:

变量绑定默认是不可变的，这意味着它们的内部不能被修改：

```rust
fn main() {
    let n = Number {
        odd: true,
        value: 17,
    };
    n.odd = false; // error: cannot assign to `n.odd`,
                   // as `n` is not declared to be mutable
}
```

And also that they cannot be assigned to:

并且它们也不能被赋值给：

```rust
fn main() {
    let n = Number {
        odd: true,
        value: 17,
    };
    n = Number {
        odd: false,
        value: 22,
    }; // error: cannot assign twice to immutable variable `n`
}

```

`mut` makes a variable binding mutable:

`mut`使变量绑定具有可变性：

```rust
fn main() {
    let mut n = Number {
        odd: true,
        value: 17,
    }
    n.value = 19; // all good
}
```

## Traits(特质)

Traits are something multiple types can have in common:

特征是多种类型可以共有的东西：

```rust
trait Signed {
    fn is_strictly_negative(self) -> bool;
}
```

### Orphan rules(孤儿规则)

You can implement:You can implement:

你可以实现：

* one of your traits on anyone’s type
在任何人的类型上实现你的一个特征    
* anyone’s trait on one of your types
在你的某个类型上实现任意人的特性    
* but not a foreign trait on a foreign type
但不能在外部类型上实现外部 trait


These are called the “orphan rules”.

这些被称为“孤儿规则”。

Here’s an implementation of our trait on our type:

以下是我们的特质在我们的类型上的实现：

```rust
impl Signed for Number {
    fn is_strictly_negative(self) -> bool {
        self.value < 0
    }
}

fn main() {
    let n = Number { odd: false, value: -44 };
    println!("{}", n.is_strictly_negative()); // prints "true"
}
```

Our trait on a foreign type (a primitive type, even):

我们的 trait 用于一个外部类型（甚至是基本类型）：

```rust
impl Signed for i32 {
    fn is_strictly_negative(self) -> bool {
        self < 0
    }
}

fn main() {
    let n: i32 = -44;
    println!("{}", n.is_strictly_negative()); // prints "true"
}
```

A foreign trait on our type:

我们类型上的外来特性：

```rust
// the `Neg` trait is used to overload `-`, the
// unary minus operator.
impl std::ops::Neg for Number {
    type Output = Number;

    fn neg(self) -> Number {
        Number {
            value: -self.value,
            odd: self.odd,
        }
    }
}

fn main() {
    let n = Number { odd: true, value: 987 };
    let m = -n; // this is only possible because we implemented `Neg`
    println!("{}", m.value); // prints "-987"
}
```

### The Self type(Self 类型)

An `impl` block is alwaysfora type, so, inside that block, `Self` means that type:

一个`impl`块总是用于某个类型，因此，在该块内部，`Self`指的就是那个类型：

```rust
impl std::ops::Neg for Number {
    type Output = Self;

    fn neg(self) -> Self {
        Self {
            value: -self.value,
            odd: self.odd,
        }
    }
}
```

### Marker traits(标记 trait)

Some traits are markers - they don’t say that a type implements some methods, they say that certain things can be done with a type.

有些特性是标记——它们并非表明某种类型实现了某些方法，而是表明可以对某种类型执行某些操作。

For example,`i32`implements trait`Copy`(in short,`i32`is`Copy`), so this works:

例如，`i32`实现了 trait`Copy`（简而言之，`i32`是`Copy`），因此这样是可行的：

```rust
fn main() {
    let a: i32 = 15;
    let b = a; // `a` is copied
    let c = a; // `a` is copied again
}
```

And this also works:And this also works:

而且这也适用：

```rust
fn print_i32(x: i32) {
    println!("x = {}", x);
}

fn main() {
    let a: i32 = 15;
    print_i32(a); // `a` is copied
    print_i32(a); // `a` is copied again
}
```

But the `Number` struct is not `Copy`, so this doesn’t work:

但`Number`结构体不具备`Copy`特性，因此这行不通：

```rust
fn main() {
    let n = Number { odd: true, value: 51 };
    let m = n; // `n` is moved into `m`
    let o = n; // error: use of moved value: `n`
}
```

And neither does this:

而这个也不行：

```rust
fn print_number(n: Number) {
    println!("{} number {}", if n.odd { "odd" } else { "even" }, n.value);
}

fn main() {
    let n = Number { odd: true, value: 51 };
    print_number(n); // `n` is moved
    print_number(n); // error: use of moved value: `n`
}
```

But it works if `print_number` takes an immutable reference instead:

但如果`print_number`改为接受一个不可变引用，它就会生效：

```rust
fn print_number(n: &Number) {
    println!("{} number {}", if n.odd { "odd" } else { "even" }, n.value);
}

fn main() {
    let n = Number { odd: true, value: 51 };
    print_number(&n); // `n` is borrowed for the time of the call
    print_number(&n); // `n` is borrowed again
}
```

It also works if a function takes amutable reference - but only if our variable binding is also `mut`.

如果函数接受一个可变引用，它也能正常工作——但前提是我们的变量绑定也是`mut`。

```rust
fn invert(n: &mut Number) {
    n.value = -n.value;
}

fn print_number(n: &Number) {
    println!("{} number {}", if n.odd { "odd" } else { "even" }, n.value);
}

fn main() {
    // this time, `n` is mutable
    let mut n = Number { odd: true, value: 51 };
    print_number(&n);
    invert(&mut n); // `n is borrowed mutably - everything is explicit
    print_number(&n);
}
```

### Trait method receivers(特征方法接收者)

Trait methods can also take `self` by reference or mutable reference:

特征方法也可以通过引用或可变引用来获取`self`：

```rust
impl std::clone::Clone for Number {
    fn clone(&self) -> Self {
        Self { ..*self }
    }
}
```

When invoking trait methods, the receiver is borrowed implicitly:

调用特征方法时，接收者会被隐式借用：

```rust
fn main() {
    let n = Number { odd: true, value: 51 };
    let mut m = n.clone();
    m.value += 100;

    print_number(&n);
    print_number(&m);
}
```

To highlight this: these are equivalent:

为了强调这一点：以下内容是等价的：

```rust
let m = n.clone();

let m = std::clone::Clone::clone(&n);
```

Marker traits like `Copy` have no methods:

像`Copy`这样的标记 trait 没有方法：

```rust
// note: `Copy` requires that `Clone` is implemented too
impl std::clone::Clone for Number {
    fn clone(&self) -> Self {
        Self { ..*self }
    }
}

impl std::marker::Copy for Number {}
```

Now, `Clone` can still be used:

现在，`Clone`仍然可以使用：

```rust
fn main() {
    let n = Number { odd: true, value: 51 };
    let m = n.clone();
    let o = n.clone();
}
```

But `Number` values will no longer be moved:

但`Number`值将不再被移动：

```rust
fn main() {
    let n = Number { odd: true, value: 51 };
    let m = n; // `m` is a copy of `n`
    let o = n; // same. `n` is neither moved nor borrowed.
}
```

### Deriving traits(派生特征)

Some traits are so common, they can be implemented automatically by using the `derive` attribute:

有些特性非常常见，可以通过使用`derive`属性来自动实现：

```rust
#[derive(Clone, Copy)]
struct Number {
    odd: bool,
    value: i32,
}

// this expands to `impl Clone for Number` and `impl Copy for Number` blocks.
```

## Generics(泛型)

### Generic functions(泛型函数)

Functions can be generic:

函数可以是泛型的：

```rust
fn foobar<T>(arg: T) {
    // do something with `arg`
}
```

They can have multiple type parameters, which can then be used in the function’s declaration and its body, instead of concrete types:

它们可以有多个类型参数，这些参数随后可用于函数的声明及其主体，而非具体类型：

```rust
fn foobar<L, R>(left: L, right: R) {
    // do something with `left` and `right`
}
```

### Type parameter constraints (trait bounds)(类型参数约束（特征边界）)

Type parameters usually have constraints, so you can actually do something with them.

类型参数通常有约束，这样你实际上就可以对它们进行一些操作。

The simplest constraints are just trait names:

最简单的约束只是特质名称：

```rust
fn print<T: Display>(value: T) {
    println!("value = {}", value);
}

fn print<T: Debug>(value: T) {
    println!("value = {:?}", value);
}
```

There’s a longer syntax for type parameter constraints:

类型参数约束有更长的语法：

```rust
fn print<T>(value: T)
where
    T: Display,
{
    println!("value = {}", value);
}
```

Constraints can be more complicated: they can require a type parameter to implement multiple traits:

约束可以更复杂：它们可能要求一个类型参数实现多个 trait：

```rust
use std::fmt::Debug;

fn compare<T>(left: T, right: T)
where
    T: Debug + PartialEq,
{
    println!("{:?} {} {:?}", left, if left == right { "==" } else { "!=" }, right);
}

fn main() {
    compare("tea", "coffee");
    // prints: "tea" != "coffee"
}
```

### Monomorphization(单态化)

Generic functions can be thought of as namespaces, containing an infinity of functions with different concrete types.

泛型函数可以被视为命名空间，包含无数个具有不同具体类型的函数。

Same as with crates, and modules, and types, generic functions can be “explored” (navigated?) using `::`

与 crate、模块和类型一样，泛型函数也可以使用 `::` 进行“探索”（或者说导航？）。

```rust
fn main() {
    use std::any::type_name;
    println!("{}", type_name::<i32>()); // prints "i32"
    println!("{}", type_name::<(f64, char)>()); // prints "(f64, char)"
}
```

This is lovingly called turbofish syntax, because `::<>` looks like a fish.

这被亲切地称为涡轮鱼语法，因为`::<>`看起来像一条鱼。

### Generic structs(泛型结构体)

Structs can be generic too:

结构体也可以是泛型的：

```rust
struct Pair<T> {
    a: T,
    b: T,
}

fn print_type_name<T>(_val: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let p1 = Pair { a: 3, b: 9 };
    let p2 = Pair { a: true, b: false };
    print_type_name(&p1); // prints "Pair<i32>"
    print_type_name(&p2); // prints "Pair<bool>"
}
```

### Example: `Vec<T>`(示例：`Vec<T>`)

The standard library type `Vec`(a heap-allocated array), is generic:

标准库类型`Vec`（一种堆分配数组）是泛型的：

```rust
fn main() {
    let mut v1 = Vec::new();
    v1.push(1);
    let mut v2 = Vec::new();
    v2.push(false);
    print_type_name(&v1); // prints "Vec<i32>"
    print_type_name(&v2); // prints "Vec<bool>"
}
```

Speaking of`Vec`, it comes with a macro that gives more or less “vec literals”:

说到`Vec`，它带有一个宏，这个宏大致能提供“vec字面量”：

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = vec![true, false, true];
    print_type_name(&v1); // prints "Vec<i32>"
    print_type_name(&v2); // prints "Vec<bool>"
}
```

## Macros(宏)

All of `name!()`,`name![] `or `name!{}` invoke a macro. Macros just expand to regular code.

`name!()`、`name![]`或`name!{}`都会调用宏。宏只会展开为常规代码。

In fact,`println`is a macro:

实际上，`println`是一个宏：

```rust
fn main() {
    println!("{}", "Hello there!");
}
```

This expands to something that has the same effect as:

这会扩展为具有相同效果的内容：

```rust
fn main() {
    use std::io::{self, Write};
    io::stdout().lock().write_all(b"Hello there!\n").unwrap();
}
```

## The panic! macro(panic! 宏)

`panic` is also a macro. It violently stops execution with an error message, and the file name / line number of the error, if enabled:

`panic`也是一个宏。它会强制停止执行并显示一条错误消息，如果启用的话，还会显示错误的文件名/行号：

```rust
fn main() {
    panic!("This panics");
}
// output: thread 'main' panicked at 'This panics', src/main.rs:3:5
```

## Functions that panic(会引发恐慌的函数)

Some methods also panic. For example, the `Option` type can contain something, or it can contain nothing. If`.unwrap()`is called on it, and it contains nothing, it panics:

一些方法也会引发 panic。例如，`Option`类型可以包含某个值，也可以不包含任何值。如果对其调用`.unwrap()`且它不包含任何值，就会引发 panic：

```rust
fn main() {
    let o1: Option<i32> = Some(128);
    o1.unwrap(); // this is fine

    let o2: Option<i32> = None;
    o2.unwrap(); // this panics!
}

// output: thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src/libcore/option.rs:378:21
```

## Enums(sum types)(枚举(和类型))

`Option` is not a struct - it’s an `enum`, with two variants.

`Option`不是结构体——它是一个`enum`，有两个变体。

```rust
enum Option<T> {
    None,
    Some(T),
}

impl<T> Option<T> {
    fn unwrap(self) -> T {
        // enums variants can be used in patterns:
        match self {
            Self::Some(t) => t,
            Self::None => panic!(".unwrap() called on a None option"),
        }
    }
}

use self::Option::{None, Some};

fn main() {
    let o1: Option<i32> = Some(128);
    o1.unwrap(); // this is fine

    let o2: Option<i32> = None;
    o2.unwrap(); // this panics!
}

// output: thread 'main' panicked at '.unwrap() called on a None option', src/main.rs:11:27
```

`Result` is also an enum, it can either contain something, or an error:

`Result`也是一个枚举，它要么包含某些内容，要么包含一个错误：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

It also panics when unwrapped and containing an error.

当它被解包且包含错误时，也会引发 panic。

## Lifetimes(生命周期)

Variables bindings have a “lifetime”:

变量绑定具有“生命周期”：

```rust
fn main() {
    // `x` doesn't exist yet
    {
        let x = 42; // `x` starts existing
        println!("x = {}", x);
        // `x` stops existing
    }
    // `x` no longer exists
}
```

Similarly, references have a lifetime:

同样，引用也有生命周期：

```rust
fn main() {
    // `x` doesn't exist yet
    {
        let x = 42; // `x` starts existing
        let x_ref = &x; // `x_ref` starts existing - it borrows `x`
        println!("x_ref = {}", x_ref);
        // `x_ref` stops existing
        // `x` stops existing
    }
    // `x` no longer exists
}
```

The lifetime of a reference cannot exceed the lifetime of the variable binding it borrows:

引用的生命周期不能超过它所借用的变量绑定的生命周期：

```rust
fn main() {
    let x_ref = {
        let x = 42;
        &x
    };
    println!("x_ref = {}", x_ref);
    // error: `x` does not live long enough
}
```

### Borrowing rules(借用规则)
one or more immutable borrows XOR one mutable borrow

一个或多个不可变借用 异或 一个可变借用

A variable binding can be immutably borrowed multiple times:

变量绑定可以被多次不可变借用：

```rust
fn main() {
    let x = 42;
    let x_ref1 = &x;
    let x_ref2 = &x;
    let x_ref3 = &x;
    println!("{} {} {}", x_ref1, x_ref2, x_ref3);
}
```

While borrowed, a variable binding cannot be mutated:

变量绑定在被借用时不能被修改：

```rust
fn main() {
    let mut x = 42;
    let x_ref = &x;
    x = 13;
    println!("x_ref = {}", x_ref);
    // error: cannot assign to `x` because it is borrowed
}
```

While immutably borrowed, a variable cannot bemutably borrowed:

当变量被不可变借用时，它不能被可变借用：

```rust
fn main() {
    let mut x = 42;
    let x_ref1 = &x;
    let x_ref2 = &mut x;
    // error: cannot borrow `x` as mutable because it is also borrowed as immutable
    println!("x_ref1 = {}", x_ref1);
}
```

### Functions generic over lifetimes(生命周期泛型函数)

References in function arguments also have lifetimes:

函数参数中的引用也具有生命周期：

```rust
fn print(x: &i32) {
    // `x` is borrowed (from the outside) for the
    // entire time this function is called.
}
```

Functions with reference arguments can be called with borrows that have different lifetimes, so:

带有引用参数的函数可以用具有不同生命周期的借用进行调用，因此：

* All functions that take references are generic
所有接受引用的函数都是泛型的    
* Lifetimes are generic parameters
生命周期是泛型参数

Lifetimes’ names start with a single quote,`'`:

生命周期的名称以单引号开头，`'`：

```rust
// elided (non-named) lifetimes:
fn print(x: &i32) {}

// named lifetimes:
fn print<'a>(x: &'a i32) {}
```

This allows returning references whose lifetime depend on the lifetime of the arguments:

这允许返回其生命周期依赖于参数生命周期的引用：

```rust
struct Number {
    value: i32,
}

fn number_value<'a>(num: &'a Number) -> &'a i32 {
    &num.value
}

fn main() {
    let n = Number { value: 47 };
    let v = number_value(&n);
    // `v` borrows `n` (immutably), thus: `v` cannot outlive `n`.
    // While `v` exists, `n` cannot be mutably borrowed, mutated, moved, etc.
}
```

### Lifetime elision(生命周期省略)

When there is a single input lifetime, it doesn’t need to be named, and everything has the same lifetime, so the two functions below are equivalent:

当只有一个输入生命周期时，无需对其命名，并且所有内容都具有相同的生命周期，因此下面两个函数是等效的：

```rust
fn number_value<'a>(num: &'a Number) -> &'a i32 {
    &num.value
}

fn number_value(num: &Number) -> &i32 {
    &num.value
}
```

### Structs generic over lifetimes(生命周期泛型结构体)

Structs can also begeneric over lifetimes, which allows them to hold references:

结构体也可以在生命周期上是泛型的，这使它们能够持有引用：

```rust
struct NumRef<'a> {
    x: &'a i32,
}

fn main() {
    let x: i32 = 99;
    let x_ref = NumRef { x: &x };
    // `x_ref` cannot outlive `x`, etc.
}
```

The same code, but with an additional function:

相同的代码，但增加了一个函数：

```rust
struct NumRef<'a> {
    x: &'a i32,
}

fn as_num_ref<'a>(x: &'a i32) -> NumRef<'a> {
    NumRef { x: &x }
}

fn main() {
    let x: i32 = 99;
    let x_ref = as_num_ref(&x);
    // `x_ref` cannot outlive `x`, etc.
}
```

The same code, but with “elided” lifetimes:

相同的代码，但带有“省略的”生命周期：

```rust
struct NumRef<'a> {
    x: &'a i32,
}

fn as_num_ref(x: &i32) -> NumRef<'_> {
    NumRef { x: &x }
}

fn main() {
    let x: i32 = 99;
    let x_ref = as_num_ref(&x);
    // `x_ref` cannot outlive `x`, etc.
}
```

### Implementations generic over lifetimes(生命周期泛型实现)

`impl` blocks can be generic over lifetimes too:

`impl`块也可以在生命周期上是泛型的：

```rust
impl<'a> NumRef<'a> {
    fn as_i32_ref(&'a self) -> &'a i32 {
        self.x
    }
}

fn main() {
    let x: i32 = 99;
    let x_num_ref = NumRef { x: &x };
    let x_i32_ref = x_num_ref.as_i32_ref();
    // neither ref can outlive `x`
}
```

But you can do elision (“to elide”) there too:

但你也可以在那里进行省略（“省略”）：

```rust
impl<'a> NumRef<'a> {
    fn as_i32_ref(&self) -> &i32 {
        self.x
    }
}
```

You can elide even harder, if you never need the name:

如果你根本不需要这个名称，你可以省略得更彻底：

```rust
impl NumRef<'_> {
    fn as_i32_ref(&self) -> &i32 {
        self.x
    }
}
```

## The 'static lifetime(static生命周期)

There is a special lifetime, named `'static`, which is valid for the entire program’s lifetime.

有一种特殊的生命周期，名为`'static`，它在整个程序的生命周期内都是有效的。

String literals are`'static`:

字符串字面值是`'static`：

```rust
struct Person {
    name: &'static str,
}

fn main() {
    let p = Person {
        name: "fasterthanlime",
    };
}
```

But references to a `String` are not static:

但对`String`的引用不是静态的：

```rust
struct Person {
    name: &'static str,
}

fn main() {
    let name = format!("fasterthan{}", "lime");
    let p = Person { name: &name };
    // error: `name` does not live long enough
}
```

In that last example, the local `name` is not a `&'static str`, it’s a `String`. It’s been allocated dynamically, and it will be freed. Its lifetime is less than the whole program (even though it happens to be in `main`).

在最后一个例子中，本地的`name`不是一个`&'static str`，而是一个`String`。它是动态分配的，并且会被释放。它的生命周期短于整个程序（尽管它恰好位于`main`中）。


To store a non-`'static` string in `Person`, it needs to either:

要在`Person`中存储非`'static`字符串，需要：

A) Be generic over a lifetime:

A) 对生命周期进行泛型处理：

```rust
struct Person<'a> {
    name: &'a str,
}

fn main() {
    let name = format!("fasterthan{}", "lime");
    let p = Person { name: &name };
    // `p` cannot outlive `name`
}
```

or或


B) Take ownership of the string

取得字符串的所有权

```rust
struct Person {
    name: String,
}

fn main() {
    let name = format!("fasterthan{}", "lime");
    let p = Person { name: name };
    // `name` was moved into `p`, their lifetimes are no longer tied.
}
```

## Struct literal assignment shorthand(结构体字面量赋值简写)

in a struct literal, when a field is set to a variable binding of the same name:

在结构体字面值中，当一个字段被设置为同名的变量绑定时：

```rust
    let p = Person { name: name };
```

It can be shortened like this:

可以像这样缩写：

```rust
    let p = Person { name };
```

## Owned types vs reference types(所有权类型与引用类型)

For many types in Rust, there are owned and non-owned variants:

在Rust中，许多类型都有所有权变体和非所有权变体：

* Strings: `String` is owned, `&str` is a reference.
字符串：`String` 是有所有权的，`&str` 是引用。    

* Paths: `PathBuf` is owned, `&Path` is a reference.
路径：`PathBuf` 是有所有权的，`&Path` 是引用。    

* Collections: `Vec<T>` is owned, `&[T]` is a reference.
集合：`Vec<T>` 是有所有权的，`&[T]` 是引用。     


### Slices(切片)

Rust has slices - they’re a reference to multiple contiguous elements.

Rust 有切片——它们是对多个连续元素的引用。

You can borrow a slice of a vector, for example:

你可以借用向量的一部分，例如：

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let v2 = &v[2..4];
    println!("v2 = {:?}", v2);
}

// output:
// v2 = [3, 4]
```

### Operator overloading(运算符重载)

The above is not magical. The indexing operator (`foo[index]`) is overloaded with the`Index` and `IndexMut` traits.

上述内容并非魔法。索引运算符（`foo[index]`）通过`Index`和`IndexMut`特征进行了重载。


The `..` syntax is just range literals. Ranges are just a few structs defined in the standard library.

`..`语法只是范围字面量。范围只是标准库中定义的几个结构体。

They can be open-ended, and their rightmost bound can be inclusive, if it’s preceded by`=`.

它们可以是开放式的，如果最右侧边界前有`=`，则该边界可以是包含性的。

```rust
fn main() {
    // 0 or greater
    println!("{:?}", (0..).contains(&100)); // true
    // strictly less than 20
    println!("{:?}", (..20).contains(&20)); // false
    // 20 or less than 20
    println!("{:?}", (..=20).contains(&20)); // true
    // only 3, 4, 5
    println!("{:?}", (3..6).contains(&4)); // true
}
```

### Borrowing rules and slices(借用规则和切片)

Borrowing rules apply to slices.Borrowing rules apply to slices.

借用规则适用于切片。

```rust
fn tail(s: &[u8]) -> &[u8] {
  &s[1..]
}

fn main() {
    let x = &[1, 2, 3, 4, 5];
    let y = tail(x);
    println!("y = {:?}", y);
}
```

This is the same as:

这与以下内容相同：

```rust
fn tail<'a>(s: &'a [u8]) -> &'a [u8] {
  &s[1..]
}
```

This is legal:

这是合法的：

```rust
fn main() {
    let y = {
        let x = &[1, 2, 3, 4, 5];
        tail(x)
    };
    println!("y = {:?}", y);
}
```

but only because`[1, 2, 3, 4, 5]`is a `'static` array.

但这只是因为`[1, 2, 3, 4, 5]`是一个`'static`数组。


So, this is illegal:

所以，这是非法的：

```rust
fn main() {
    let y = {
        let v = vec![1, 2, 3, 4, 5];
        tail(&v)
        // error: `v` does not live long enough
    };
    println!("y = {:?}", y);
}
```

because a vector is heap-allocated, and it has a non-`'static` lifetime.

因为向量是在堆上分配的，并且它具有非`'static`生命周期。

### Fallible functions (Result)(易错函数（Result）)

Functions that can fail typically return a `Result`:

可能失败的函数通常会返回一个`Result`：

```rust
fn main() {
    let s = std::str::from_utf8(&[240, 159, 141, 137]);
    println!("{:?}", s);
    // prints: Ok("🍉")

    let s = std::str::from_utf8(&[195, 40]);
    println!("{:?}", s);
    // prints: Err(Utf8Error { valid_up_to: 0, error_len: Some(1) })
}
```

If you want to panic in case of failure, you can `.unwrap()`:

如果在失败时想要触发 panic，可以使用`.unwrap()`：

```rust
fn main() {
    let s = std::str::from_utf8(&[240, 159, 141, 137]).unwrap();
    println!("{:?}", s);
    // prints: "🍉"

    let s = std::str::from_utf8(&[195, 40]).unwrap();
    // prints: thread 'main' panicked at 'called `Result::unwrap()`
    // on an `Err` value: Utf8Error { valid_up_to: 0, error_len: Some(1) }',
    // src/libcore/result.rs:1165:5
}
```

Or `.expect()`, for a custom message:

或者 .expect()`，用于自定义消息：

```rust
fn main() {
    let s = std::str::from_utf8(&[195, 40]).expect("valid utf-8");
    // prints: thread 'main' panicked at 'valid utf-8: Utf8Error
    // { valid_up_to: 0, error_len: Some(1) }', src/libcore/result.rs:1165:5
}
```

Or, you can `match`:

或者，您可以`match`：

```rust
fn main() {
    match std::str::from_utf8(&[240, 159, 141, 137]) {
        Ok(s) => println!("{}", s),
        Err(e) => panic!(e),
    }
    // prints 🍉
}
```

Or you can `if let`:

或者你可以使用`if let`：

```rust
fn main() {
    if let Ok(s) = std::str::from_utf8(&[240, 159, 141, 137]) {
        println!("{}", s);
    }
    // prints 🍉
}
```

Or you can bubble up the error:

或者你可以向上传递错误：

```rust
fn main() -> Result<(), std::str::Utf8Error> {
    match std::str::from_utf8(&[240, 159, 141, 137]) {
        Ok(s) => println!("{}", s),
        Err(e) => return Err(e),
    }
    Ok(())
}
```

Or you can use `?` to do it the concise way:

或者你可以使用`?`来以简洁的方式完成：

```rust
fn main() -> Result<(), std::str::Utf8Error> {
    let s = std::str::from_utf8(&[240, 159, 141, 137])?;
    println!("{}", s);
    Ok(())
}

```

## Dereferencing(解引用)

The`*` operator can be used todereference, but you don’t need to do that to access fields or call methods:

`*`运算符可用于解引用，但访问字段或调用方法时无需这样做：

```rust
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    let p = Point { x: 1.0, y: 3.0 };
    let p_ref = &p;
    println!("({}, {})", p_ref.x, p_ref.y);
}

// prints `(1, 3)`
```

And you can only do it if the type is `Copy`:

只有当类型为`Copy`时，你才能执行此操作：

```rust
struct Point {
    x: f64,
    y: f64,
}

fn negate(p: Point) -> Point {
    Point {
        x: -p.x,
        y: -p.y,
    }
}

fn main() {
    let p = Point { x: 1.0, y: 3.0 };
    let p_ref = &p;
    negate(*p_ref);
    // error: cannot move out of `*p_ref` which is behind a shared reference
}
```

```rust
// now `Point` is `Copy`
#[derive(Clone, Copy)]
struct Point {
    x: f64,
    y: f64,
}

fn negate(p: Point) -> Point {
    Point {
        x: -p.x,
        y: -p.y,
    }
}

fn main() {
    let p = Point { x: 1.0, y: 3.0 };
    let p_ref = &p;
    negate(*p_ref); // ...and now this works
}
```

## Function types, closures(函数类型、闭包)

Closures are just functions of type `Fn`,`FnMut` or `FnOnce` with some captured context.

闭包只是具有某些捕获上下文的`Fn`、`FnMut`或`FnOnce`类型的函数。

Their parameters are a comma-separated list of names within a pair of pipes (`|`). They don’t need curly braces, unless you want to have multiple statements.

它们的参数是由竖线（`|`）对中的逗号分隔的名称列表。它们不需要花括号，除非你想包含多个语句。

```rust
fn for_each_planet<F>(f: F)
    where F: Fn(&'static str)
{
    f("Earth");
    f("Mars");
    f("Jupiter");
}

fn main() {
    for_each_planet(|planet| println!("Hello, {}", planet));
}

// prints:
// Hello, Earth
// Hello, Mars
// Hello, Jupiter
```

The borrow rules apply to them too:

借用规则也适用于他们：

```rust
fn for_each_planet<F>(f: F)
    where F: Fn(&'static str)
{
    f("Earth");
    f("Mars");
    f("Jupiter");
}

fn main() {
    let greeting = String::from("Good to see you");
    for_each_planet(|planet| println!("{}, {}", greeting, planet));
    // our closure borrows `greeting`, so it cannot outlive it
}
```

For example, this would not work:

例如，这样做是行不通的：

```rust
fn for_each_planet<F>(f: F)
    where F: Fn(&'static str) + 'static // `F` must now have "'static" lifetime
{
    f("Earth");
    f("Mars");
    f("Jupiter");
}

fn main() {
    let greeting = String::from("Good to see you");
    for_each_planet(|planet| println!("{}, {}", greeting, planet));
    // error: closure may outlive the current function, but it borrows
    // `greeting`, which is owned by the current function
}
```

But this would:

但这会：

```rust
fn main() {
    let greeting = String::from("You're doing great");
    for_each_planet(move |planet| println!("{}, {}", greeting, planet));
    // `greeting` is no longer borrowed, it is *moved* into
    // the closure.
}
```

### FnMut and borrowing rules(FnMut与借用规则)

An `FnMut` needs to be mutably borrowed to be called, so it can only be called once at a time.

调用`FnMut`需要进行可变借用，因此它一次只能被调用一次。

This is legal:

这是合法的：

```rust
fn foobar<F>(f: F)
    where F: Fn(i32) -> i32
{
    println!("{}", f(f(2)));
}

fn main() {
    foobar(|x| x * 2);
}

// output: 8
```

This isn’t:

这不行：

```rust
fn foobar<F>(mut f: F)
    where F: FnMut(i32) -> i32
{
    println!("{}", f(f(2)));
    // error: cannot borrow `f` as mutable more than once at a time
}

fn main() {
    foobar(|x| x * 2);
}
```

This is legal again:

这再次合法了：

```rust
fn foobar<F>(mut f: F)
    where F: FnMut(i32) -> i32
{
    let tmp = f(2);
    println!("{}", f(tmp));
}

fn main() {
    foobar(|x| x * 2);
}

// output: 8
```

`FnMut` exists because some closures mutably borrow local variables:

`FnMut`存在的原因是某些闭包会可变借用局部变量：

```rust
fn foobar<F>(mut f: F)
    where F: FnMut(i32) -> i32
{
    let tmp = f(2);
    println!("{}", f(tmp));
}

fn main() {
    let mut acc = 2;
    foobar(|x| {
        acc += 1;
        x * acc
    });
}

// output: 24
```

Those closures cannot be passed to functions expecting `Fn`:

这些闭包不能传递给期望`Fn`的函数：

```rust
fn foobar<F>(f: F)
    where F: Fn(i32) -> i32
{
    println!("{}", f(f(2)));
}

fn main() {
    let mut acc = 2;
    foobar(|x| {
        acc += 1;
        // error: cannot assign to `acc`, as it is a
        // captured variable in a `Fn` closure.
        // the compiler suggests "changing foobar
        // to accept closures that implement `FnMut`"
        x * acc
    });
}
```

`FnOnce` closures can only be called once. They exist because some closure move out variables that have been moved when captured:

`FnOnce` 闭包只能被调用一次。它们的存在是因为有些闭包会移出在捕获时已被移出的变量：

```rust
fn foobar<F>(f: F)
    where F: FnOnce() -> String
{
    println!("{}", f());
}

fn main() {
    let s = String::from("alright");
    foobar(move || s);
    // `s` was moved into our closure, and our
    // closures moves it to the caller by returning
    // it. Remember that `String` is not `Copy`.
}
```

This is enforced naturally, as `FnOnce` closures need to bemovedin order to be called.

这是自然强制执行的，因为需要将`FnOnce`闭包移动后才能调用。

So, for example, this is illegal:

例如，以下情况是不合法的：

```rust
fn foobar<F>(f: F)
    where F: FnOnce() -> String
{
    println!("{}", f());
    println!("{}", f());
    // error: use of moved value: `f`
}
```

And, if you need convincing that our closuredoesmove`s`, this is illegal too:

而且，如果你需要证据证明我们的闭包确实会移动`s`，那这也是不合法的：

```rust
fn main() {
    let s = String::from("alright");
    foobar(move || s);
    foobar(move || s);
    // use of moved value: `s`
}
```

But this is fine:

但这样是没问题的：

```rust
fn main() {
    let s = String::from("alright");
    foobar(|| s.clone());
    foobar(|| s.clone());
}
```

Here’s a closure with two arguments:

这是一个带有两个参数的闭包：

```rust
fn foobar<F>(x: i32, y: i32, is_greater: F)
    where F: Fn(i32, i32) -> bool
{
    let (greater, smaller) = if is_greater(x, y) {
        (x, y)
    } else {
        (y, x)
    };
    println!("{} is greater than {}", greater, smaller);
}

fn main() {
    foobar(32, 64, |x, y| x > y);
}
```

Here’s a closure ignoring both its arguments:

这是一个忽略其两个参数的闭包：

```rust
fn main() {
    foobar(32, 64, |_, _| panic!("Comparing is futile!"));
}
```

Here’s a slightly worrying closure:

这是一个略显令人担忧的收尾：

```rust
fn countdown<F>(count: usize, tick: F)
    where F: Fn(usize)
{
    for i in (1..=count).rev() {
        tick(i);
    }
}

fn main() {
    countdown(3, |i| println!("tick {}...", i));
}

// output:
// tick 3...
// tick 2...
// tick 1...
```
### The toilet closure(马桶闭包)

And here’s a toilet closure:

这是一个“马桶闭包”：

```rust
fn main() {
    countdown(3, |_| ());
}
```

It’s called that because `|_| ()` looks like a toilet.

之所以这么叫，是因为`|_| ()`看起来像个马桶。

## Loops, iterators(循环、迭代器)

Anything that is iterable can be used in a `for in` loop.

任何可迭代的对象都可以在`for in`循环中使用。

We’ve just seen a range being used, but it also works with a `Vec`:

我们刚刚看到了范围的用法，但它也适用于`Vec`：

```rust
fn main() {
    for i in vec![52, 49, 21] {
        println!("I like the number {}", i);
    }
}
```

Or a slice:

或者切片：

```rust
fn main() {
    for i in &[52, 49, 21] {
        println!("I like the number {}", i);
    }
}

// output:
// I like the number 52
// I like the number 49
// I like the number 21
```

Or an actual iterator:

或者一个实际的迭代器：

```rust
fn main() {
    // note: `&str` also has a `.bytes()` iterator.
    // Rust's `char` type is a "Unicode scalar value"
    for c in "rust".chars() {
        println!("Give me a {}", c);
    }
}

// output:
// Give me a r
// Give me a u
// Give me a s
// Give me a t
```

Even if the iterator items are filtered and mapped and flattened:

即使迭代器项经过了过滤、映射和扁平化处理：

```rust
fn main() {
    for c in "SuRPRISE INbOUND"
        .chars()
        .filter(|c| c.is_lowercase())
        .flat_map(|c| c.to_uppercase())
    {
        print!("{}", c);
    }
    println!();
}

// output: UB
```

## Returning closures(返回闭包)

You can return a closure from a function:

你可以从函数中返回一个闭包：

```rust
fn make_tester(answer: String) -> impl Fn(&str) -> bool {
    move |challenge| {
        challenge == answer
    }
}

fn main() {
    // you can use `.into()` to perform conversions
    // between various types, here `&'static str` and `String`
    let test = make_tester("hunter2".into());
    println!("{}", test("******"));
    println!("{}", test("hunter2"));
}
```

### Capturing into a closure(捕获到闭包中)

You can even move a reference to some of a function’s arguments, into a closure it returns:

你甚至可以将对函数某些参数的引用移到它返回的闭包中：

```rust
fn make_tester<'a>(answer: &'a str) -> impl Fn(&str) -> bool + 'a {
    move |challenge| {
        challenge == answer
    }
}

fn main() {
    let test = make_tester("hunter2");
    println!("{}", test("*******"));
    println!("{}", test("hunter2"));
}

// output:
// false
// true
```

Or, with elided lifetimes:

或者，使用省略的生命周期：

```rust
fn make_tester(answer: &str) -> impl Fn(&str) -> bool + '_ {
    move |challenge| {
        challenge == answer
    }
}
```

## Conclusion(结论)

And with that, we hit the 30-minute estimated reading time mark, and you should be able to read most of the Rust code you find online.

这样一来，我们就达到了预计30分钟的阅读时间，你应该能够读懂网上大多数的Rust代码了。

Writing Rust is a very different experience from reading Rust. On one hand, you’re not reading the solution to a problem, you’re actually solving it. On the other hand, the Rust compiler helps out a lot.

编写Rust代码与阅读Rust代码的体验大不相同。一方面，你不是在阅读某个问题的解决方案，而是在实际解决这个问题。另一方面，Rust编译器的帮助非常大。

The Rust compiler has high-quality diagnostics (which include suggestions) for all the mistakes featured in this article.

Rust编译器针对本文中提到的所有错误都提供了高质量的诊断信息（包括建议）。

