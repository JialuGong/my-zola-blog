+++
title = "Node的事件循环浅析"
date = 2021-01-02
insert_anchor_links = "right"
[taxonomies]
categories = ["编程"]
tags=["Node"]
+++
[TOC]
### 1. Node概述
<img src="node-structure.png" style="width:70%"></img>
- Node API,javascript实用程序
- Node Core，用于实现Node API的一组javascript模块，基于libuv、V8引擎等
- javacript　Engine,将js语言翻译成机器码,node使用V8引擎翻译所有的js代码，但是node不一定使用V8作为js引擎,也可使用其它js引擎.
- Event Loop,事件循环,Node使用libuv实现事件循环,无论是主线阶段还是事件循环阶段，都是在一个线程中完成的,实现事件循环需要使用到node异步API,将回调函数作为参数,在进行事件循环时会运行这些回调函数.<br>
  node不仅适合I/O密集型，同时CPU密集型工作也可以胜任.libuv 使用称为“工人池”的线程池，即用于卸载 I/O 密集型任务和 CPU 密集型任务的线程池。

---
### 2. Node 事件循环
1.1 **非阻塞I/O**
> node 使用三种技术来防止阻塞业务逻辑的执行,分别是事件、异步API和非阻塞I/O。

`非阻塞I/O`指程序在做其它事情的时候，该程序可以发起一个请求获取网络资源，等到网络资源获取成功后，就执行一个回调函数来处理该网络资源;

1.2 **Node的事件循环**

源自于IBM的一篇关于NODE的[教程](https://developer.ibm.com/zh/tutorials/learn-nodejs-the-event-loop/)。
首先，粗略的解释一下什么是`事件循环`(或者`事件轮询`),这里很容易联想到组成原理课上关于I/O访问的那一章,最原始的I/O访问方式即程序轮循，即由处理器每隔一定时间对各个I/O端口进行访问,检查各端口有无准备就绪。`事件轮询`浅显的理解为单向运行先入先出队列。
- Node生命周期
  
  ![node事件循环图](node-事件循环.png)</br>
在js主线完成以后,node线程将开始执行事件循环,即执行调用node非阻塞api中的回调函数.若代码中没有非阻塞的代码,运行完主线之后,node将结束;若存在非阻塞代码,则node将在所有回调函数执行完毕后结束.
- 一些概念
  - 任务队列(Task Queue)<br>
    上图中的每个阶段都会有一个FIFO的任务队列,一般来说,特定于某个阶段的逻辑会在开始时执行，直到队列为空或者达到系统的限制.需要值得注意的是`Task Queue`并不是一个真正的`Queue`,而是一个`set`,它是选取第一个可执行的`Task`执行,而非严格意义上的出队入队
  - js的异步任务分为hong任务和微任务 
    - 微任务(Microtask)<br> 
  在主线和事件每个阶段完成后，会立刻执行微任务回调,Node 开发者编写的代码仅以微任务形式在主线、计时器（Timers） 阶段、轮询（Poll） 阶段和 查询（Check） 阶段中运行。普遍的观点是认为promise回调属于微任务，但在某些浏览器中（Microsoft Edge, Firefox 40, iOS Safari and desktop Safari 8.0.8 ）中，会把promise的回调视作一个新的宏任务而不是微任务。<br>
  主要有:Promise.then、MutaionObserver、process.nextTick(Node.js环境）
    - 宏任务(Task)<br>
    主要有:script（全局任务）、setTimeout、setInterval、I/O、UI交互事件、postMessage、requestAnimationFrame、MessageChannel、setImmediate(Node.js环境） 
   
- 主要阶段说明<br>
大体上而言，事件循环包含多个调用回调函数阶段
    - 计时器阶段:将运行`setInterval()`和`setTimeout()`过期计时器回调函数
    - 轮询阶段:将轮询操作系统以查看是否完成所有I/O操作,如果完成，运行相应的回调函数
    - 检查阶段:将运行`setImmediate()`回调函数<br>
     
- 细节阶段说明<br>   
而按照上图,node时间循环中,更为详细的事件阶段为:
    - Timers阶段<br>
    任何 过期的计时器回调都会在事件循环的这个阶段中运行,计时器分为两类
        - `setImmediate()`Immediate计时器是一个node对象，它在下一个check阶段立即运行
        - `setTimeout()`,Timeout计时器也是一个node对象,它在计时器过期后尽快运行回调,在该计时器过期后,会在事件循环的下一个Timers阶段运行回调。Timeout计时器有两种,分别是:
          - `Interval`,该计时器由`setInterval()`创建,每次该计时器过期时，就运行一次回调，只要node进程仍在运行，就会重复此过程，除非你调用了`clearInterval()`
          - `Timeout`该计时器由`setTimeout()`创建，超过设置的delay(默认1ms)事件后，会运行一次回调，除非在回调运行之前调用了`clearTimeout()`<br><br> 
    当不再有过期的计时器回调运行时候，事件循环就会运行所有的微任务，运行微任务完成后,事件循环进入到Pending阶段 
    - Pending阶段<br>
    一些系统上的回调发生在本阶段执行
    - Idle和Prepare阶段
    - Poll阶段<br>
    此阶段执行I/O回调，如果轮询队列为空(无I/O之外的任何事件),则会阻塞并等待任何正在执行的I/O操作完成,然后立即执行这些操作的回调。如果调度了计时器,则Poll阶段将会结束，在必要时运行微任务,然后事件循环进入到Check阶段
    - Check阶段<br>
    只有`setImmediate()`回调会在该阶段中执行,使用户可以在Poll阶段变得空闲时立刻执行一些代码.Check阶段的回调队列为空后，会运行所有的微任务,然后事件循环进入到Close阶段 
    - Close阶段<br> 
    如果某个套接字（socket）或句柄突（handle）然关闭（例如，如果调用了一个套接字的 `socket.destroy()` 方法），则会执行此阶段，这种情况下会触发其close事件。 
- 通过代码加深理解
```javascript
const fs=require('fs');
const { setImmediate } = require('timers');
var cnt=0;
const event=()=>{
    if(cnt>=1) {
        clearInterval(myInterval)
    }
    console.log(`${cnt}: interval begin`)
    fs.readFile('images/node-structure.png',(err,file)=>{
        console.log(`${cnt}:poll begin`)
        if(err){console.log(err)}
        else{
            console.log(file.length)
        }
    })
    setImmediate(()=>{console.log(`${cnt}: immediate`)})
    cnt++;
}
const myInterval=setInterval(event,1);
```
运行结果
``` 
0: interval begin
1: immediate
1: interval begin
2: immediate
2:poll begin
190428
2:poll begin
190428
```
其中interval以１ms进行回调,在第一次循环中,首先执行Timers阶段,此时`cnt=1`,interval过期,执行回调函数,首先输出`interval begin`,接着进入pending、idle、prepare阶段,接着开始poll进行I/O访问,此时I/Ｏ未准备好,到达check阶段,执行immediat,而后开始第二次循环,此时`cnt=1`,首先执行`clearInterval`函数,再`cnt++`,Timers阶段后,`cnt=2`,第二次循环后I/O仍然没有准备好,于是执行immediate函数,第二次循环完成后,回调队列只有有I/O的回调未完成,于是在第三次循环的poll阶段阻塞,直到I/O事件完成

#### 参考
- [IBM node.js之旅](https://developer.ibm.com/zh/languages/node-js/)
- 《Node.js实战(第二版)》
- [JS中的Event Loop和Task queue](https://zhuanlan.zhihu.com/p/68606229)
- [html living standard](https://html.spec.whatwg.org/multipage/webappapis.html#task-queue)