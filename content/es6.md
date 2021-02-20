+++
title = "ES6 笔记"
weight = 1
order = 1
date = 2021-01-16
insert_anchor_links = "right"
[taxonomies]
tags=["javascript"]
+++
### 1. let & conost
**let**<br>
`let`与`var`类似,区别在于`let`命令只在代码块内有效;`let`有以下特性
- 不存在变量提升<br>
`var`会存在变量提升的现象，如下面两段代码
```js
console.log(a)//Reference error
``` 
```js
console.log(a);//Undefined
var a=6;
```
而`let`所声明的变量必须在声明之后使用
```js
console.log(a)//Reference error
let a=6;
```
- 暂时性死区<br>
只要块级作用域存在`let`命令,它所声明的变量就"绑定"这个区域,不再受外部影响;在块代码内,如果区块存在let和const命令,则这些命令所声明的变量在声明之前(块内),是不可用的,这被称为`暂时性死区`(TDZ)
```js
var a=8;
{
    console.log(a)//8
}
```  
```js
var a=8;
{
    //TDZ开始
    console.log(a)//Reference error
    let a=8;
    //TDZ结束
}
```
- 不允许重复声明变量<br>
  `let`不允许在相同的作用域重复声明同一个变量
```js
{
    let a=9;
    let a=8;//error
    var a=7;//error
}
```
不容易发现的死区,例如
```js
function bar (x=y,y=3){
      console.log(`foo`)
}
```
**const**<br>
`const`实际上保证的并不是变量的值不该东，而是变量指向的内存地址不改动.对于js的6种基本变量,值就保存在变量指向的地址中,所以不可变动,而对于`object`之类的复合型数据变量,只能保证变量指向的地址中保存的对象指针不变动;<br>
同时`const`也变量不可提升,存在着暂时性死区<br>

**ES6声明变量的6种方法**<br>
es5中声明变量只有两种方法,分别是`var`和`function`,es6除了新添加了`const`和`let`，还添加了`import`和`class`

**关于顶层对象**<br>
顶层对象在浏览器中指`window`对象,在node中指`global`对象,在es5中,顶层对象的属性与全局变量是相同的,在es6中,`var`命令和`function`命令声明的全局变量依然是顶层对象属性,而`let`、`const`、`class`命令声明的全局变量不是顶层对象属性.

---

### 2. 变量解构
**数组的解构赋值**<br>
- 基本用法<br>
ES6允许按照一定的模式从数组对象中提取值,然后对变量进行赋值,即解构(Destructuring)
```js
let [a,b,c]=[1,2,3]
let [foo,[bar]]=[1,[2]]
```
只要等号两边的模式相同,左边的变量就会被赋予对应的值,如果未解构成功,则默认为`undefined`,只要某种数据结构具有`Iterator`接口,都可采用数组形式的解构赋值<br>
- 允许默认值<br>
  解构允许指定默认值
```js
let [foo=true]=[]//foo=true
let [x,y='b']=['a']//x='a',y='b'
let [x,y='b']=['a',undefined]//x='a',y='b'
let [x='a']=[null]//x=null
```
**对象的解构赋值**<br>
对象与数组的一个重要的不同是变量与属性必须同名才能取得正确的值.
```js
let {bar,foo}={foo:1,bar:2};//foo=1,bar=2
```
如果变量名和属性名不一致,则必须写成以下这样
```js
let {bar:x,foo}={foo:1,bar:2};//x=2; bar 为reference error
```
即将`:`左边的值赋给了右边的值,上面代码中`bar`是模式,`x`是变量.

**字符串的解构赋值**<br>
字符串可以被转换为一个类似数组的对象进行解构赋值
```js
let [a,b,c]='abc'
```
**数值和布尔值解构赋值**<br>
解构赋值时,如果等号右边是数值或者布尔值,则会先转为对象
```js
let {toString:s}=1;//s===Number.prototype.toString
```
**函数参数解构**
```js
function add([x,y]){
    return x+y;
}
```

---
### 3. 函数扩展
**函数参数默认值**<br>
- 基本用法<br>
ES6之前,不能直接为函数的参数指定默认值,只能在函数体内模拟
```js
function add(x,y){
    y=y||2;
    return x+y;
} 
```
而在ES6中,允许为函数参数设置默认值
```js
function add(x,y=2){

}
```
- 参数默认值的位置<br>
在函数调用时,如果函数非尾部的参数设置默认值,这个非尾部参数是无法省略的,需要借助`undefined`.如
```js
function add(x=3,y){
    return x+y;
}
add(,3)//error
add(undefined,3)
```
应用,利用参数默认值可以指定某一个参数不得省略,如果省略就抛出一个错误
```js
function throwIfMissing(){
    throw new Error(`Missing parameter`)
}

function foo(mustBeProvided=throwIfMissing){
    return mustBeProvided;
}
```
- 函数的length属性<br>
函数的`length`属性将返回没有指定默认值的参数的个数,注意,若默认值不是尾参数,则返回第一个默认参数前面参数的个数
```js
(function foo(x,y=2,z){}).length\\1
```
**严格模式**<br>
如果函数参数使用了默认值、解构赋值或者扩展运算符,那么函数内部就不能显式设定为严格模式<br>
**name属性**<br>
函数的`name`属性将返回函数的名字,在ES6如果将一个匿名函数赋给一个变量,则改变量会变成这个函数的名字.注意若将一个具名函数赋值给一个变量,则`name`属性都返回这个具名函数原本的名字;
```js
const add =()=>{}
const a=add;
a.name;//add
```
**箭头函数**
- 基本用法<br>
当箭头后非代码块时候,为一句表达式时,能够隐式地返回计算结果,如
```js
const add=(x,y)=>x+y;
```
当返回值是一个对象时,需要在对象外添加圆括号,因为大括号会被解析为代码块
```js
const genObj=()=>({a:1,b:2})
```
当语句多于一句时候,需要添加大括号,并用`return`显示返回结果
- 注意事项<br>
    - 箭头函数内的this对象指向的是其定义时所在的父区域
    ```js
    let a=9;
    function whichA(){
        let a=8;
        const getA=()=>{
            return this.a;
        }
        returen getA();
    }
    whichA();//8
    ``` 
    - 不可以使用`yield`命令
    - 不可以当作构造函数,不可以使用new命令
    - 不可以使用`arguments`对象,其`arguments`对象是定义时父区域的`arguments`对象.如果使用需要用`rest`参数代替
    ```js
    function add(){
        const sum=()=>{
            for (const i in arguments) {
                console.log(arguments[i])
            }
        }
        return sum;
    }
    add(1,2,3)();//1,2,3
    add()(1,2,3);//
  ```
---
### 5.数组扩展
**扩展运算符**<br>
- 扩展运算符<br>
扩展运算符`...`将一个数组转化为用逗号分隔的参数序列;
```js
...[1,2,3]//1,2,3
```
- `apply`方法<br>
`apply`方法也能够将一组数组转化为由逗号分割的参数序列
```js
function add (x,y){
    return x+y;
}
let ans=add.apply(null,[1,2]);//3
```
求最大值
```js
//es5;
Math.max.apply(null,[1,2,3]);
//es6;
Math.max(...[1,2,3]);
```
将一个数组添加到另一个数组尾部
```js
let arr1=[1,2,3];
let arr2=[4,5,6];
//es5
let arr3=Array.prototype.push.apply(arr1,arr2);
//es6
let arr4=arr1.push(...arr2);
```
```js
//es5
new (Date.bind.apply(Date,[null,2015,1,1]));
//es6
new Date(...[2015,0,1])
```
- 应用<br>
  - 合并数组
  ```js
  let more=[1,2,3]
  //es5;
  [1,2].cotanct(more)
  [1,2,...more]
  ```
  - 与解构赋值结合生成数组,注意,用扩展运算符对数组赋值,只能将参数放在最后一位,否则报错
  ```js
  let a,b;
  //es5
   a=list[0];
   b=list.slice(1);
  //es6;
  [a,...b]=list
  ```
  - 将字符串转化为数组,并且支持unicode编码
  ```js
  let s='😊🥺😉😍😘😚'
  console.log([...s])//[ '😊', '🥺', '😉', '😍', '😘', '😚' ]
  ```
  - 实现了`Iterator`接口的对象,任何实现了`Iterator`接口的对象都可以用扩展运算符转化为真正的数组,而没有实现`Iterator`接口的类似数组的对象无法通过扩展运算符变为数组
  - Map和Set结构,Ｇenerator函数<br>扩展运算符内部调用的是数据结构的`Iterator`接口,因此实现了`Iterator`接口的对象可以使用扩展运算符,如`Map`,`Set`;
  ```js
  let map=new Map([[1,'first'],[2,'second']]);
  [...map]
  ```
  `Generator`函数运行后会返回一个遍历器对象,因此也可以使用扩展运算符
  ```js
    const go =function *(){
        let i = 0;
        while (i < 3) {
            yield i++;
        }
    }
    console.log([...go()])//[0,1,2]
  ```
- Array.from()<br>`Array.from()`可将两类对象转为真正的数组:
    - 类似数组的对象(array-like object)
  ```js
  let arrayLike={
      '0':1,
      '1':2,
      length:2,
  }
  //es5
  let arr1=[].slice.call(arrayLike);
  //es6
  let arr2=Array.from(arrayLike);
  ```
    - 可遍历的对象,实现了`Iterator`接口的对象<br>
  此外,`Array.from()`还可以第二个参数还可以接收一个匿名函数,实现类似于数组`map`的方法
  ```js
  Array.from(arrayLike,x=>x*x);
  Array.from(arrayLike).map(x=>x*x);
  ```
- Array.of()<br>
`Array.of()`将一组参数转化为数组,使用`Array.of()`来解决`Array()`该构造函数在参数不同的情况下的行为差异
```js
Array.of(4)//[4]
Array(4)//[,,,]
Array.of(1,2)//[1,2]
Array(1,2)//[1,2]
```
---
### 6.对象扩展
**属性表示方式**<br>
ES6允许直接将变量和函数作为对象的属性和方法
```js
let a=1;
let obj={a}//obj={a:1}
//等同于
let obj={a:a};
```
函数简写
```js
let obj={
    method(x,y){
       return {x,y}
    }
}
//等同于
let obj={
    method:function (x,y){
       return {x,y}
    }
}
```
- js属性名<br>js定义语言有两种方式,分别是
    - 直接使用标识符作为属性名(ES5仅仅此一种)
    ```js
    obj.foo=true;
    ```
    - 使用表达式作为属性名
    ```js
    obj['ab'+'c']=3;
    ```
- Object.is()<br>
`Object.is()`与严格运算符`===`基本一致,只是在`+0`,`-0`和`NaN`有区别
```js
Object.is(+0,-0);//false;
Object.is(NaN,NaN);//true;
```
- Object.assign()<br>`Object.assign(target,source,source...)`,该方法将源对象中所有可枚举的属性复制到目标对象中,若目标对象存在该对象,则覆盖,此处的赋值为浅拷贝
```js
let obj1={a:1,b{c:1}};
let obj2=object.assign({},obj1);
obj1.b.c=2;
//obj2.b.c==2
```
- 属性的可枚举性<br>对象的每一个属性都有一个描述对象`Descriptor`,用于控制属性的行为,可以使用`Object.getOwnPropertyDescriptor()`方法查看
```js
let obj={'a':123};
Object.getOwnPropertyDescriptor(obj,'a')
//{ value: 123, writable: true, enumerable: true, configurable: true }
```
其中`enumerable`表示该属性是否可枚举
- 属性的遍历,共有5种遍历对象属性的方法
  - `for in`
  - `Object.keys(obj)`返回一个数组,包含对象不含继承的,不含`Symbol`的所有可枚举属性
  - `Object.getOwnPropertyNames(obj)`返回一个数组,包含对象所有对象,包含不可枚举属性,不包含`Symbol`属性
  - `Object.getOwnPropertySymbols(obj)`返回一个数组，包含对象所有`Symbol`属性
  - `Reflect.ownKeys(obj)`返回一个数组,包含对象所有属性,不管是否是`Symbol`属性和是否可枚举

- `_proto_`属性、`Object.setPrototypeOf()`、`Object.getPrototypeOf()`
---

### 7.Symbol
**`Symbol`基本概述**<br>
由于ES5中对象的属性都是字符串,可能会造成属性名的冲突,为了解决冲突,ES6引入了类型`Symbol`用来保证属性名的独一无二.`Symbol`是js的第7种原始类型,表示独一无二的值,通过`Symbol()`函数生成<br>
`Symbol()`接受一个字符串换作为参数,如果参数是一个对象，就会掉用对象的`toString`方法，将其转化为字符串,然后再生成一个`Symbol`值<br>
`Symbol`函数的参数只表示对当前`Symbol`值的描述,因此相同参数的`Symbol`函数的返回值不相等.
```js
let s1=Symbol('foo');
let s2=Symbol('foo');
s1==s2;//false
```
**Symbol作为属性名**<br>
每一个`Symbol`变量都是不同的，用`Symbol`变量作为属性,可以保证不出现同名的属性,`Symbol`作为对象属性有三种写法
```js
let mySymbol=Symbol()
//第一种
let obj={};
obj[mySymbol]=123;
//第二种
let a={[mySymbol]:123};
//第三种
let a={};
Object.defineProperty(obj,mySymbol,{value:123})
```
**Symbol属性的应用**<br>
魔术字符串指在代码中多次出现、与代码出现强耦合的某一个字符串或者数字值,消除魔术字符串的方式是将其设置为一个`Symbol`属性

---

### 8. proxy
**概述**
`proxy`用于修改某些操作的默认行为,`Proxy`可理解为在目标对象前架设一个拦截层,外界对该对象的访问,必须先通过拦截层,提供了一种机制可以对外界的访问进行过滤和改写;<br>如下面代码重新定义了对象属性的读取`get`和设置`set`
```js
let obj = new Proxy({}, {
    get:function(target,key,receiver){
        console.log(`getting ${key}`);
        return Reflect.get(target,key,receiver)
    },
    set:function(target,key,value,receiver){
        console.log(`setting ${key}`)
        return Reflect.get(target,key,value,receiver)
    }
})
obj.val=123;
//setting val
++obj.val;
//getting val
//setting val
```
注意,起作用的是`Proxy`构造函数返回的Proxy对象,而不是作为`Proxy`构造函数的参数对象<br>
**This问题**<br>
在`Proxy`代理的情况下,目标对象内部的`this`关键字会指向`Proxy`代理

---
### 9.Reflect
`Reflect`主要将`Object`对象的一些明显属于语言内部的方法放到`Reflect`对象上以及修改某些`Reflect`方法的返回结果

---
### 10.Promise
**概述**<br>
`Promise`简单来说是一个容器,保存着某个未来才会结束的事件的结果,从语法上来讲`Promise`是一个对象,它提供一个统一的API,各种异步操作都可以,`Promise`有两个特点<br>
- 对象的状态不受外界影响,只有三种状态`Pending`、
`Fulfilled`、`Rejected`
- 一旦状态改变就不会再变,任何时候都可以得到这个结果,Promise状态的改变有两种可能从`Pending`状态到`Fulfilled`状态或从`Pending`状态到`Rejected`状态,发生改变后状态凝固,不会再发生变化,一直保持这个结果,这时称为`Resolved`<br>
**Promise基本用法**<br>
可使用`Promise`构造函数生成一个`promise`实例,再使用`then`、`catch`方法得到`promise`的结果
```js
const myPromise=new Promise((res,rej)=>{
    if(true){res(1)}
    else{rej(0)}
});
myPromise
    .then(res=>{})
    .catch(err=>{})
```
**Promise方法介绍**
- `Promise.prototype.then()`


### 参考
- 《ES6标准入门》 阮一峰
