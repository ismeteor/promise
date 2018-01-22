## 1背景
从敲下第一行“Hello World!”起，我们都无时无刻的使用并接触着事件循环。

去年写react的时候总会遇到设置setState，后面调用值不会改变的问题。会想到是异步的原因，试着延时定时问题得到解决。
```
    this.setState({status: 1})
    console.log(this.state.status);   //0
    setTimeout(()=>{
        console.log(this.state.status);    //1
    },100)
```
问题可以解决，所以延时定时解决了之后的很多问题。

后面看到javascript的事件循环，才真正理解了这之中的原理。

## 2单线程
从学习JavaScript开始，就一直看书和资料说JavaScript是一门单线程的语言。JavaScript到处存在着异步（刚开始认为异步是主线程下开的子线程）是依靠事件循环来实现的。

## 3同步异步
单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等着。

如果计算量大，读写IO请求、Ajax开销大。如果通过同步来获取数据会造成程序阻塞。所以像这种开销大的任务，我们使用异步处理。

所有任务可以分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。
## 4事件循环

![](https://user-gold-cdn.xitu.io/2018/1/21/1611656121ace250?w=601&h=527&f=png&s=22933)
根据事件循环图来理解这一段代码
```
//首先整段代码进入主线程执行
let a= 5;
//同步代码立即执行，输出a=5
console.log(a);
//按钮绑定点击事件，当用户点击按钮事件回调函数进入任务队列，待主线程执行完毕调用该函数
oBtn.onclick(function () {
    a = 10;
    console.log(a, 'click');
})
//当延时定时时间到的时候，回调函数进入任务队列，待主线程执行完毕调用任务队列中的函数。
setTimeout(function () {
    console.log(a,'timeout');
},5000)
```
## 5node中的事件循环
Node.js也是单线程的Event Loop，但是它的运行机制不同于浏览器环境。
![](https://user-gold-cdn.xitu.io/2018/1/22/1611b9836878c5d7?w=800&h=316&f=png&s=21842)
* V8引擎解析JavaScript脚本。
* 解析后的代码，调用Node API。
* libuv库负责Node API的执行。它将不同的任务分配给不同的线程，形成一个Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎。
* V8引擎再将结果返回给用户。

除了setTimeout和setInterval这两个方法，Node.js还提供了另外两个与"任务队列"有关的方法：process.nextTick和setImmediate。它们可以帮助我们加深对"任务队列"的理解。

process.nextTick方法可以在当前"执行栈"的尾部----下一次Event Loop（主线程读取"任务队列"）之前----触发回调函数。也就是说，它指定的任务总是发生在所有异步任务之前。setImmediate方法则是在当前"任务队列"的尾部添加事件，也就是说，它指定的任务总是在下一次Event Loop时执行，这与setTimeout(fn, 0)很像。
```
process.nextTick(function A() {
  console.log(1);
  process.nextTick(function B(){console.log(2);});
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
}, 0)
// 1
// 2
// TIMEOUT FIRED
```

上面代码中，由于process.nextTick方法指定的回调函数，总是在当前"执行栈"的尾部触发，所以不仅函数A比setTimeout指定的回调函数timeout先执行，而且函数B也比timeout先执行。这说明，如果有多个process.nextTick语句（不管它们是否嵌套），将全部在当前"执行栈"执行。

现在，再看setImmediate。

```
setImmediate(function A() {
  console.log(1);
  setImmediate(function B(){console.log(2);});
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
}, 0);
```
哪个回调函数先执行呢？答案是不确定。运行结果可能是1--TIMEOUT FIRED--2，也可能是TIMEOUT FIRED--1--2。setImmediate与setTimeout不分伯仲。

但在递归中，setImmediate永远优于setTimeout
```
setImmediate(function (){
  setImmediate(function A() {
    console.log(1);
    setImmediate(function B(){console.log(2);});
  });

  setTimeout(function timeout() {
    console.log('TIMEOUT FIRED');
  }, 0);
});
// 1
// TIMEOUT FIRED
// 2
```

最后感谢[阮一峰老师的文章](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)和张仁阳老师的思路。

