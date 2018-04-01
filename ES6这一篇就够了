## 1. let、const
### 1.1 var存在的问题
1. var有作用域问题（会污染全局作用域）
2. var可已重复声明
3. var会变量提升预解释
4. var不能定义常量

### 1.2 let、const特性
1. let、const不可以重复声明
2. let、const不会声明到全局作用域上 
3. let、const不会预解释变量
4. const做常量声明（一般常量名用大写）

### 1.3 试一试
```
//1. 作用域
//var
var a = 5;
console.log(window.a); //5

//let、const
let a = 5;
console.log(window.a); //undefined
//有一个问题，let定义的a声明到哪里了呢？欢迎解释（：

//2. 重复声明 
//var 
var a = 5;
var a = 6;

//let、const
let a = 5;
let a = 6; //Identifier 'a' has already been declared 不能重复声明

//3. 预解释
//var
console.log(a); //undefined  相当于 var a;
var a = 5;

//let const
console.log(a)//Uncaught ReferenceError: a is not defined 语法错误，没有预解释
let a = 5;

//4. 常量
//var
var a = 5;
a = 6;

//const
const a = 5;
a = 6;//Uncaught TypeError: Assignment to constant variable. 语法错误
```

## 2.解构
解构是es6新特性，可以对数组对象内容直接解析。
```
//1.数组解构
let [a1,a2,a3] = [1,2,3]; //a1 = 1; a2 = 2; a3 = 3;

//2.对象解构
let {name,age} = {name:'meteor',age:8}; //name = 'meteor',age = 8;
//let {name:n} = {name:'meteor',age:8};//n = 'meteor' 可以用“：”的形式修改变量名

//3.复杂解构
let [{age}] = [{age:8,name:'xx'},'深圳',[1,2,3]]; //age = 8   注意对象解构

//4.默认赋值
let {age=5} = {age:8,name:'xx'}; //age=8 如果没有age字段 age=5
//常用函数参数给默认值
//以前
function(){var a = a || 5}
//现在
function(a=5){}

```

## 3.字符串
es6中加入了“`”反引号，反引号中${}处理模版字符串。
### 3.1 反引号
1. 更人性的字符串拼接
```
let name = 'xx';
let age = 9;
let str = `${name}${age}岁了`;
console.log(str); //xx今年岁了
```
2. 使用正则+eval的简单模拟
```
let name = 'xx';
let age = 9;
let str = `${name}${age}岁了`;
str = str.replace(/\$\{([^}]+)\}/g, function ($1) {
    return eval(arguments[1]);
})
console.log(str); //xx今年岁了
```
3. 带标签的模版字符
```
//反引号前使用方法，方法按模版变量把字符串分成两个数组
//第一个为固定值组成的数组 
//第二个值为替换变量组成的数组
let name = 'xx';
let age = 9;
function tag(strArr, ...args) {
    var str = '';
    for (let i = 0; i < args.length; i++) {
        str += strArr[i] + args[i]
    }
    console.log(strArr,args);
    //[ '', '今年', '岁了' ]   [ 'xx', 9 ]
    //
    str += strArr[strArr.length - 1];
    return str;
}
let str = tag`${name}今年${age}岁了`;
console.log(str);
```
### 3.2 incluedes 方法
```
//判断字符串中是否包含某个字符串
let str = 'woaini';
str.includes('ai'); //true
```
### 3.3 endsWith、startsWith 方法
```
//判断字符串是否以某一个字符串开始或结束
var a = '1AB2345CD';
console.log(a.startsWith('1A')); //true
a.endsWith('cD') //false
```

## 4. 函数
### 4.1 函数参数可赋值，可解构
```
function({a=5}){}
```
### 4.2 箭头函数
1. 如果参数只有一个，可以省略小括号
2. 如果不写return，可以不写大括号
3. 没有arguments变量
4. 不改变this指向
```
//求和
let sum = (a,b)=>a+b;
```
## 5. 数组
### 5.1 数组新方法
1. Array.from(); //将类数组转化成数组 
2. Array.of(); //创建数组
3. Array.fill();//填充数组
4. Array.reduce();//传入回调适合处理 临近数组元素
5. Array.filter();//数组过滤
6. Array.find();//查找返回值
7. Array.includes();//判断数组是否有某值

### 5.2 Array.from()
```
//Array.from();
let arr = Array.from({'0':1,'1':2,length:2});
console.log(arr);//[1,2]
```
### 5.3 Array.of()
```
Array.of(2,3); //[2,3]
```
### 5.4 Array.reduce()
```
[1, 2, 3, 4, 5].reduce((prev, next, index, current) => {
    //prev 如果reduce传入第二个参数，prev为第二个参数；否则prev为数组第一个元素  往后累加
    //next 如果reduce传入第二个参数，next为第数组第一个元素；否则next为数组第二个元素  依次累加
    //index 函数第几次执行1开始
    //current当前数组
    return prev + next;
})
//reduce方法简单实现
Array.prototype.myReduce = function (cb, pre) {
    let prev = pre || this[0];
    for (var i = pre ? 0 : 1; i < this.length; i++) {
        prev = cb(prev, this[i], i, this);
    }
    return prev;
}
```
### 5.3 Array.filter();
```
let result = [1,2,3,4,5].filter(function(item){
    return item>3;
})
console.log(result);//[4,5]
//filter简单实现
Array.prototype.myFilter = function(cb){
    var arr = [];
    for(var i=0; i<this.length; i++){
        var item = this[i];
        if( cb(item) ) arr.push(item);
    }
    return arr;
}
```
### 5.4 Array.find();
```
let result = [1,1,1,2,3].find(function(item){
    return item == 2;
})
console.log(result);//2
```
## 6 对象
### 6.1 如果key和value相等则可以简写
```
let name = 'xx';
let obj = {name};
```
### 6.2 解构

## 7 Class 类
1. Class 定义类
2. extends 实现继承
3. 支持静态方法
4. constructor构造方法
```
class Parent{
    constructor(name){
        this.name = name;
    }
    getName(){
        return this.name;
    }
    static fn(){
        return 9;
    }
}
class Child extends Parent{
    constructor(name,age){
        super(name);//子类有构造函数必须有super
        this.age = age;
    }
}
let child = new Child('xingxing',9);
console.log(child.getName());//xingxing
console.log(child.name);//xingxing
console.log( Child.fn() );//9
```
