## 背景
node的招牌技能是“事件驱动，非阻塞IO”，可以看到事件在node中的重要性。  
在浏览器端存在各种事件绑定(click、mousedown...)，node中也存在各种事件绑定(fs.read、http.creaeServer...)。  
在node中还有一个自定义事件EventEmitter，官网写到很多node的api都是基于EventEmitter。

## 使用
一般我们会通过继承来使用EventsEmitter，单独new一个EventEmitter似乎除了验证方法没有意义。
```
//引入events模块
const EventEmitter = require('events');
//定义一个MyEmitter类继承EventEmiteer
class MyEmitter extends EventEmitter {}

//new一个MyEmitter对象
const myEmitter = new MyEmitter();
//给myEmitter对象绑定一个event事件
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
//触发event事件
myEmitter.emit('event');    //输出 "an event occurred!"
```
这里我们实现对EventEmitter继承，绑定事件监听函数，触发事件。

## EventEmitter简单实现
简单原理：在EventEmitter下创建一个events对象。以on监听的事件名为键名，键值是监听的函数的组合为一个数组。

1.创建EventEmitter对象
```
function EventEmitter() {
    this.events = {};  //所有事件监听函数放在这个对象里保存
    this._maxListeners = 10; //监听函数最多10个
}
```
2.给EventEmitter增加on、addEventListener绑定事件
```
//给制定函数绑定事件处理函数
EventEmitter.prototype.on = EventEmitter.prototype.addEventListener = function (type,listener) {
    if(this.events[type]){
        this.events[type].push(listener);
        if(this._maxListeners != 0 && this.events[type].length > this._maxListeners){
            console.error('MaxListenersExceededWarning: Possible EventEmitter memory leak detected.\n');
        }
    }else {
        this.events[type] = [listener];
    }
}

```
3.触发事件
```
EventEmitter.prototype.emit = function (type, ...rest) {
    if(this.events[type]){
        //遍历触发函数数组   apply把this指向当前对象  解构rest
        this.events[type].forEach((listener)=>listener.apply(this,rest));
    }
}
```
4.移除监听函数
```
EventEmitter.prototype.removeListener =  function(type,listener) {
   if (this.events[type]){
       this.events[type] = this.events[type].filter(l=>l!=listener);
       //使用filter过滤数组
   }
}
```

简单的EventEmitter模型实现完成，欢迎点评交流。
