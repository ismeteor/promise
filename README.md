## promise理解
promise的意思是承诺。承诺理解为某个时候一些条件满足后，会兑现一件事情。
```javascript
//为了方便理解我编一个小故事
//先假装我有一个女朋友
//她承诺如果她爸妈不回来就给我就可以去帮她修电脑 否则...
let promise = new Promise(function (success,fail) {
    //她说三秒后会告诉我今晚爸妈回不回来
    let herParentBack = Math.random() > 0.5 ? true : false;
    setTimeout(function(){
        if(herParentBack){
            success('我马上过来来帮你修你电脑');
        }else{
            fail('你要早点休息');
        }
    },3000);

});
 
 //第一个函数接收sucess函数传来的成功信息
 //第二个函数接收fail函数传来的失败信息

promise.then(function (sucessMsg) {
    console.log(sucessMsg);
},function (fail) {
    console.log(failMsg);
})
//我马上过来帮你修电脑
```

new Promise会传入一个回调函数，会伴着对象创建被立即调用。
这个function是承诺的主体内容。一般我们会做一些判断或者异步请求。（在这里要等女友回消息，我想好怎么回答她。）

promise.then方法在promise执行了success或fail后会执行对应的成功失败方法。 （这里理解为吧想好告诉她）


# promise实现
根据promis规范promise  [Promise/A](https://segmentfault.com/a/1190000002452115) 

Promise设置了三个状态，'pending'、'resolved'、'rejected'。


```
//Promise对象设置初始状态
function Promise(callback) {
    var self = this;
    //默认为等待状态
    self.status = 'pending';
    self.value = undefined;
    self.reason = undefined;
    //用数组来保存成功函数
    self.onResolvedCallBacks = [];
    self.onRejectedCallbacks = [];

    function resolve(value){
        if(self.status === 'pending'){
            //设置为成功状态
            self.status = 'resolved';
            self.value = value;
            self.onResolvedCallBacks.forEach(item=>item(self.value));
        }
    }
    function reject(reason) {
        if(self.status === 'pending'){
            //设置为失败状态
            self.status = 'rejected';
            self.reason = reason;
            self.onRejectedCallbacks.forEach(item=>item(self.reason));
        }
    }

    callback(resolve,reject);
    //调用Promise回调函数
}

module.exports = Promise;
```

增加Promise.then方法
```
//根据Promise状态执行成功失败方法
Promise.prototype.then = function (onFulfilled,onRejected) {
    let self = this;
    onFulfilled = typeof onFulfilled == 'function'?onFulfilled:function(value){return value};
  onReject = typeof onReject=='function'?onReject:function(reason){throw reason;}
    if(self.status === 'resolved'){
        return new Promise(function (resolve,reject) {
            try {
                let x = onFullFilled(self.value);
                if(x instanceof Promise){
                    x.then(resolve,reject);
                }else{
                    resolve(x);
                }
            }
            catch(e) {
                reject(e);
            }
        })
        //执行成功方法
    }else if(self.status == 'rejected'){
        return new Promise(function (resolve,reject) {
            try {
                let x = onRejected(self.reason);
                if(x instanceof Promise){
                    x.then(resolve,reject);
                }else{
                    resolve(x);
                }
            }
            catch(e) {
                reject(e)
            }
        })
        //执行失败方法
    }
    if(self.status === 'pending'){
        return new Promise(function (reslove,reject) {
            self.onResolvedCallBacks.push(function () {
                let x = onFullFilled(self.value);
                if(x instanceof Promise){
                    x.then(resolve,reject);
                }else{
                    resolve(x);
                }
            })
            self.onRejectedCallbacks.push(function () {
                let x = onRejected(self.reason);
                if(x instanceof Promise){
                    x.then(resolve,reject);
                }else{
                    resolve(x);
                }
            })
        })
        //将成功失败方法保存在数组里
    }
}
```

Promise.all
```
Promise.all = all;
function all(iterable) {
  var self = this;
  if (!isArray(iterable)) {
    return this.reject(new TypeError('must be an array'));
  }

  var len = iterable.length;
  var called = false;
  if (!len) {
    return this.resolve([]);
  }

  var values = new Array(len);
  var resolved = 0;
  var i = -1;
  var promise = new this(INTERNAL);

  while (++i < len) {
    allResolver(iterable[i], i);
  }
  return promise;
  function allResolver(value, i) {
    self.resolve(value).then(resolveFromAll, function (error) {
      if (!called) {
        called = true;
        doReject(promise, error);
      }
    });
    function resolveFromAll(outValue) {
      values[i] = outValue;
      if (++resolved === len && !called) {
        called = true;
        doResolve(promise, values);
      }
    }
  }
}
```

Promise.race
```
Promise.race = function(iterable) {
  var self = this;
  if (!isArray(iterable)) {
    return this.reject(new TypeError('must be an array'));
  }

  var len = iterable.length;
  var called = false;
  if (!len) {
    return this.resolve([]);
  }

  var i = -1;
  var promise = new this(INTERNAL);

  while (++i < len) {
    resolver(iterable[i]);
  }
  return promise;
  function resolver(value) {
    self.resolve(value).then(function (response) {
      if (!called) {
        called = true;
        doResolve(promise, response);
      }
    }, function (error) {
      if (!called) {
        called = true;
        doReject(promise, error);
      }
    });
  }
}
```

