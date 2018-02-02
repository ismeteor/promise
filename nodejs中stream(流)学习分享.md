## 为什么需要stream
做为一个前端猿，一直对后端每天可以任性的操作服务器文件感到羡慕。最近学习node的fs模块玩的也是意犹未尽好不满足。fs.readFile方法在读取大文件时无法胜任，所以流出现了。

## stream(流)
我的理解
我们在网上下载资源时，稍微大一点就要下载个把小时。我们的带宽远远不能满足从服务端把资源整个端走，所以要把资源拆分成小块，一块一块的运输。资源就像水一样流到了我们本地。

While it is important for all Node.js users to understand how streams work.(官网说流很重要！)

## 流的类型
nodejs有四种基本的流类型：
* 1 Readable 可读流
* 2 Writable 可写流
* 3 Duplex 可读可写流
* 4 Transform 在读写过程中可以修改和变换数据的Duplex流

## 1.Readable
可读流的功能是作为上游，提供数据给下游。
### 创建与使用
```
let {Readable} = require('stream');
let readable = Readable();
/*
Readable({read(){})
*/
let source = ['a','b','c'];
readable.setEncoding('utf8');
readable._read = function () {
    let data = source.shift()||null;
    console.log('read:',data);
    this.push(data);
}
readable.on('end', function () {
  console.log('end')
})
readable.on('data', function (data) {
    console.log(data)
})
/*
输出：
read: a
read: b
a
read: c
b
read: null
c
end
*/
```
Readable通过实例_read或read方法,在需要数据时，_read()方法会自动调用。
* _read方法结束的标志是this.push(null)。
* 当readable绑定data时_read()方法自动调用

data事件：当读入数据时触发data事件传入回调读到内容。   
end事件：“消耗完”，需要满足两个条件：
* 已经调用push(null)，声明不会再有任何新的数据产生  
* 缓存中的数据也被读取完

### 两种模式(flowing 和 paused)

#### 流动(flowing)模式
 flowing 模式下， 可读流自动从系统底层读取数据，并通过 EventEmitter 接口的事件尽快将数据提供给应用。  
 以下条件均可以使readable进入flowing模式：  
* 调用resume方法
* 如果之前未调用pause方法进入paused模式，则监听data事件也会调用resume方法。
* readable.pipe(writable)。pipe中会监听data事件。  

#### 暂停(paused)模式
 paused 模式下，必须显式调用 stream.read() 方法来从流中读取数据片段。   
 可读流可以通过下面途径切换到 paused 模式：
 * 如果不存在管道目标（pipe destination），可以通过调用 stream.pause() 方法实现。
 * 如果存在管道目标，可以通过取消 'data' 事件监听，并调用 stream.unpipe() 方法移除所有管道目标来实现。

---

### Readable流程
1. 通过read或_read方法读入数据
2. 将数据push到当前对象
3. 触发emit data方法

深入理解readable  
众所周知node i/o操作通常采用异步操作。在读取数据时会把数据写入缓存区。
![stream-how-data-comes-out.png](http://upload-images.jianshu.io/upload_images/3269069-f31112262038fec2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### read方法
在创建流时，会设置一个highWaterMark参数创建最高水位线。

![stream-read.png](http://upload-images.jianshu.io/upload_images/3269069-d59a10d66024b9f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###自定义可读流
```
var stream = require('stream');
var util = require('util');
util.inherits(Counter, stream.Readable);
function Counter(options) {
    stream.Readable.call(this, options);
    this._index = 0;
}
Counter.prototype._read = function() {
    if(this._index++<3){
        this.push(this._index+'');
    }else{
        this.push(null);
    }
};
var counter = new Counter();

counter.on('data', function(data){
    console.log("读到数据: " + data.toString());//no maybe
});
counter.on('end', function(data){
    console.log("读完了");
});
```

## 2.Writable
可写流是对数据写入'目的地'的一种抽象。可写流的功能是作为下游，消耗上游提供的数据。
```
let {Writable} = require('stream');
var writable = Writable({
    write: function (data,_,next) {
        console.log(data);
        next&&next();
    }
})
/*
Writable._write(data,_,next)
*/
writable.write('a');
writable.write('b');
writable.write('c');
writable.end();
/*
<Buffer 61>
<Buffer 62>
<Buffer 63>
*/
```
### write方法
* write()或_write()的第三个参数next为回调函数，调用next()表示写入完成，开始写下一个数据。
* 必须调用end()方法来告诉writable，所有数据均已写入。

### 自定义可写流
```
var stream = require('stream');
var util = require('util');
util.inherits(Writer, stream.Writable);
let stock = [];
function Writer(opt) {
    stream.Writable.call(this, opt);
}
Writer.prototype._write = function(chunk, encoding, callback) {
    setTimeout(()=>{
        stock.push(chunk.toString('utf8'));
        console.log("增加: " + chunk);
        callback();
    },500)
};
var w = new Writer();
for (var i=1; i<=5; i++){
    w.write("项目:" + i, 'utf8');
}
w.end("结束写入",function(){
    console.log(stock);
});
```

## 3.管道流
从readable读出数据，writeable写入数据
### 创建与使用
```
const stream = require('stream')

var index = 0;
const readable = stream.Readable({
    highWaterMark: 2,
    read: function () {
        process.nextTick(() => {
            console.log('push', ++index)
            this.push(index+'');
        })
    }
})

const writable = stream.Writable({
    highWaterMark: 2,
    write: function (chunk, encoding, next) {
        console.log('写入:', chunk.toString())
    }
})

readable.pipe(writable);
```
## 4.Duplex
```
const {Duplex} = require('stream');
const inoutStream = new Duplex({
    write(chunk, encoding, callback) {
        console.log(chunk.toString());
        callback();
    },
    read(size) {
        this.push((++this.index)+'');
        if (this.index > 3) {
            this.push(null);
        }
    }
});

inoutStream.index = 0;
process.stdin.pipe(inoutStream).pipe(process.stdout);
```
## 5.Transform
```
const {Transform} = require('stream');

const upperCase = new Transform({
    transform(chunk, encoding, callback) {
        this.push(chunk.toString().toUpperCase());
        callback();
    }
});

process.stdin.pipe(upperCase).pipe(process.stdout);
/*
实现大小写转换
inpt: a
A
*/
```
 
---
_最近在学习nodejs，经过整理写下学习笔记。欢迎同学沟通学习心得，如有错误的地方也欢迎指正。_
