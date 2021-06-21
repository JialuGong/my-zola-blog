+++
title = "多态，以编程语言为例"
weight = 1
order = 1
date = 2021-06-21
insert_anchor_links = "right"
[taxonomies]
tags=["pl","polymorphism"]
+++
> In programming languages and type theory, polymorphism is the provision of a single interface to entities of different types or the use of a single symbol to represent multiple different types.


又名： https://zh.wikipedia.org/wiki/Polymorphism_(computer_science)
<!-- more -->
第一次听说多态是在Java的学习，多态被认为是Java的三大特性之一（另外两个是封装和继承）。在课程学习中或者面试八股文中，多态的意思如下：相同的事物，调用其相同的方法，参数也相同时，但表现的行为却不同。它有三个必要条件，分别是：
1. 继承：在多态中必须存在有继承关系的子类和父类。
2. 重写：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。
3. 向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法。
然而，根据Wiki，多态的作用却不尽显于此，对于学过JAVA的同学，多态含义仅仅是适用于JAVA的狭义多态。本文旨在基于Wiki对于多态的定义，结合常用的语言，来使多态的概念更为清楚和便于理解。
如引文所说，“多态，在计算机科学中，在编程语言和类型理论中，多态性是为不同类型的实体提供单个接口，或者使用单个符号来表示多种不同类型”，可能有点拗口，我们可以通过多态的分类来更加清楚地理解它。

多态（polymorphis）普遍分为以下三种：
1. Ad hoc polymorphis（国内普遍译为特设多态）：对任意一组单独指定的类型定义一个通用接口
2. Parametric polymorphism(参数多态)：当一个或多个类型不是由名称指定但可以通过抽象符号指定时
3. Subtyping（子类型多态）：当一个名字表示许多不同类的实例，并且这些实例都与一些通用超类有关

## Ad hoc
特设多态，可以简单的表述为`不同参数（类型和参数个数二元组），不同行为`。在wiki介绍中，有两种典型的Ad hoc多态：方法重载和操作符重载。
并不是所有的语言都支持方法重载和操作符重载，常见的动态语言例如Javascript并不支持方法重载，因为例如JS的动态语言出于其弱类型的特性，并不会对函数调用时的参数做检查。而大多的静态语言都支持，只是有些出于语法设计并未实现，例如rust并不支持方法重载。而我们并不熟练的母语c++实现了这两种重载方法。
```c++
int add(int a,int b){
    return a+b;
}
String add(String a,String b){

}
```
```c++
Point opertor+(const Point &b){
    Point p;
    p.x=this->x+b.x;
    p.y=this->y+b.y;
    return p;
}
```
在Ad hoc多态中，**我们编写了一系列的函数，这些函数拥有相同的方法名，却拥有不同的参数，对于不同的参数，我们为这些参数一一书写了函数的实现方法**。

## 参数多态
我们可以用另外一个熟悉的词来解释参数多态——范形。对于范形函数，与Ad hoc相同的是，我们依旧是同样的函数名，不同类型的参数（拥有相同的参数个数），但是我们并不为这些不同的参数一一实现方法，而是仅仅编写一次。同样，我们用c++模版举例：
```c++
inline T const& Max (T const& a, T const& b) 
{ 
    return a < b ? b:a; 
} 
```
rust也支持函数中使用范形，但是以一种更为规范的方法，下文将进行说明。
不仅是函数，同样，还有创建一个结构体等，以rust为例：
```rust
struct Point<T,U>{
    x:T,
    y:U,
}
```

## 子类型多态
子类型多态的提出，是为了限制可以在特定情况下使用多态性的类型范围。下面是Wiki的一个例子。
```c++
abstract class Animal {
    abstract String talk();
}

class Cat extends Animal {
    String talk() {
        return "Meow!";
    }
}

class Dog extends Animal {
    String talk() {
        return "Woof!";
    }
}

static void letsHear(final Animal a) {
    println(a.talk());
}

static void main(String[] args) {
    letsHear(new Cat());
    letsHear(new Dog());
}
```