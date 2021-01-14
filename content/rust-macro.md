+++
title = "初尝 Rust macro_rules!"
weight = 1
order = 1
date = 2020-06-19
insert_anchor_links = "right"
[taxonomies]
categories = ["编程"]
tags=["rust"]
+++
> 前言:笔者在学习编译原理课程的时需要写一个简单的parser，由于太懒，将作业拖到最后三天才真正开始写，于是就选择了糊代码的递归下降法写parser。由于没有思考就直接上手糊，导致代码非常冗余难看，有许多重复的部分。虽然代码糊完了，可是实在不忍心看见如此丑陋的代码，于是便想着用宏去解决那些重复的代码，由此就有了这篇笔记。虽然现在也很丑陋，但是比之前要好一点了。
什么是宏,在c中，我们也接触过宏,例如
```c
#define a 5
```
我所理解的宏就是捕获->展开。捕获即捕获你所定义的，上面的代码就是捕获`a`，展开什么，即将你捕获的东西替换成后边的语法。再举一个c的例子
```c
# define a a+5
```
从语法层面则是:
```
Ident(a) ----> +
              / \
      Ident(a)    5
```
需要注意的是，上述的展开是不一定正确的，例如
```c
a*3
```
会被解析为
```
     +
    / \
   a   *
      / \
     5   3
```
rust中有两种宏,一种是过程宏,一种是声明宏,rust的`macro_rules!`既是声明宏。rust中使用`macro_rules`声明一个宏。大体的框架是这样
```rust
macro_rules!{
    ()=>{
        
    }
}
```
rust 中可以捕获的种类有
- `ident` : 即Identifier,一个表示符号,如`x`
- `path`  : 一个受限的名字,如`Foo::Bar`
- `pat`   : 一个模式，如`Some(0u8)`,`(1,2)`
- `expr`  : Expression，一个表达式,如`1+1`,`if x>b{a}else{c}`,`f(3i32)`等
- `ty`  :Type,一个类型,如`Vec<i32,i32>`,`i32`
- `block`:一个大括号界定的语句序列，或者一个表达式。例如：`{ log(error, "hi"); return 12; }`
- `stmt`：一个单独语句。例如：let x = 3
- `block`：一个大括号界定的语句序列，或者一个表达式。例如：`{ log(error, "hi"); return 12; }`
- `item`：一个项。例如：`fn foo() { }`，`struct Bar`
- `meta`：一个“元数据项”，可以在属性中找到。例如：`cfg(target_os = "windows")`
- `tt`：一个单独的记号树

笔者在用宏的时候遇到了一个奇怪的问题(原因是我小白)。内容如下。首先声明了一个宏，并实现了他的Disply trait。
```rust
use core::fmt;
enum Test{
    Foo(String),
    Bar,
}

impl fmt::Display for Test {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            Test::Foo(id) => write!(f, "Foo({})", id),
            Test::Bar => write!(f, "Bar"),
        }
    }
}
```
而后我定义一个宏,以及一个funtion用于生成一个Enum variantes.
```rust
use {Test,Test::*};

fn gen_test()->Test{
    Bar
}
macro_rules!{
    ($test:expr)=>{
        match gen_test(){
            $test=>{format!("Equal,{}",$test)}
            _=>{format!("Not Equal,{}",$test)}
        }
    }
}
```
于是奇怪的事情发生了，报错如下:
```
arbitrary expressions aren't allowed in patterns
```
rust定义`expr`是不能出现在match的arm上的,纵使我们知道了这是一个Enum variants,但是此时他已经被定义为`expr`的属性了。那我们把它变成`pat`可以吗。好，这个问题算是解决了，新的问题又出现了，不能打印。
```
expected expression, found `Bar`
```
这就很让人难受了，于是我又机智得换成了`path`,心想这下解决了吧，好，没有报错。但是要知道`path`无法匹配类似`Foo()`这样的变量呀。

于是笔者就去[stackoverflow](https://stackoverflow.com/questions/61768445/how-to-format-a-patenum-in-rust-macro-rules/61769112#61769112)上问了一波.得到了以下的答案<br>
> I don't understand why you're using either pat or path: in your expansion, $test is just an expression / value (an RHS). pat is a pattern (an LHS, either target of an assignment or pattern of a match arm) and path is a qname.

此处

- `LHS`:left-hand side,即赋值变量左侧
- `RHS`:right-hand side,即赋值变量右侧
即和expr 不能出现在match臂上的原因相同，即pat作为LHS,是没办法被作为RHS所应用的。

p.s:第一次在stackoverflow上问问题，社区氛围太友好了，感动
