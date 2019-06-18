---
layout:     post
title:      Promise
subtitle:   
date:       2019-xx-xx 00:00:00
author:     hjfrun
header-img: 
catalog: false
tags:
    - Promise
	- Javascript
---



```
var p = new Promise(function(resolve, reject) {
  // do some async stuff
  setTimeout(function() {
    console.log('mission completed');
    resolve('whatever data');
  }, 2000);
});
```

`Promise`的构造函数接收一个参数，是函数，并且传入两个参数：`resolve`和`reject`，分别表示异步操作执行成功后的回调函数和异步操作执行失败后的回调函数。按照标准来讲，`resolve`是将`Promise`的状态置为`fullfiled`，`reject`是将`Promise`的状态置为`rejected`。

以上该代码中，执行一个异步操作，也就是setTimeout，2秒之后输出`mission completed`，并且调用resolve方法。



```
function runAsync() {
  var p = new Promise(function(resolve, reject) {
    // do some async stuff
    setTimeout(() => {
      console.log('mission completed');
      resolve('whatever data');
    }, 2000);
  });

  return p;
}

runAsync().then(function(data) {
  console.log(data);
  // do additional work with the data received
  // ...
})
```

输出为

```
mission completed
whatever data
```

在runAsync()的返回上直接调用then方法，then接收一个参数，是函数。并且会拿到我们在runAsync中调用resolve时传递的参数。运行这段代码，2秒后执行`mission completed`，紧接着输出`whatever data`。

then里面的函数跟我们平时的回调函数一个意思，能够在runAsync这个异步任务执行完成之后被执行。这个就是Promise的作用了。简单来说，就是把原来的回调方法分离出来。在异步操作执行完后，用链式调用的方式执行回调函数。



```
function runAsync(callback) {
  setTimeout(() => {
    console.log('mission completed');
    callback('whatever data');
  }, 2000);
}

runAsync(function(data) {
  console.log(data);
});
```

以上代码效果也是一样的。既然如此，还要Promise干嘛？问题是在处理多层回调的情况！

如果callback也是一个异步操作，而且执行完成后也需要有相应的回调函数，该怎么办？总不能再定义一个callback2，然后给callback传进去吧。而Promise的优势在于，可以在then方法中继续写Promise对象并返回，然后继续调用then来进行回调操作。



```
function runAsync1() {
  var p = new Promise(function(resolve, reject) {
    // do some async stuff
    setTimeout(() => {
      console.log('1 finished');
      resolve('data 1');
    }, 1000);
  });

  return p;
}

function runAsync2() {
  var p = new Promise(function(resolve, reject) {
    // do some async stuff
    setTimeout(() => {
      console.log('2 finished');
      resolve('data 2');
    }, 2000);
  });
  
  return p;
}

function runAsync3() {
  var p = new Promise(function(resolve, reject) {
    // do some async stuff
    setTimeout(() => {
      console.log('3 finished');
      resolve('data 3');
    }, 3000);
  });

  return p;
}

runAsync1().then(function(data) {
  console.log(data);
  return runAsync2();
}).then(function(data){
  console.log(data);
  // return runAsync3();
  return 'direct data';
}).then(function(data){
  console.log(data);
});
```



```
function getNumber() {
  var p = new Promise(function(resolve, reject) {
    // do some async stuff
    setTimeout(() => {
      var num = Math.ceil(Math.random() * 10);  // generate number in 1 ~ 10
      if (num < 5) {
        resolve(num);
      } else {
        reject('too large number');
      }
    }, 1000);
  });

  return p;
}

getNumber()
.then(function(data) {
  console.log('resolved');
  console.log(data);
}, function(reason, data) {
  console.log('rejected');
  console.log(reason);
});
```

getNumber函数用来异步获取一个数字，1秒后执行完成，如果数字小于等于5，我们认为是'成功'了，调用resolve修改Promise的状态。否则，我们认为'失败'了，调用reject并传递一个参数，作为失败的原因。

运行getNumber并且在then中传递两个参数，then方法可以接受两个参数，第一个是对应resolve的回调，第二个是对应reject的回调。所以我们能够分别拿到他们传过来的数据。多次运行，会随机得到下面的结果。

```
rejected
too large number
```

```
resolved
4
```



**catch**

catch其实和then的第二个参数一样，用来指定reject的回调。不过它还有另外一个作用：在执行resolve的回调时，如果跑出了异常，那么不会报错卡死js，而是进到这个catch方法中。

 **all**

```
Promise.all([runAsync1(), runAsync2(), runAsync3()])
.then(function(results) {
  console.log(results);
})
```

执行结果：

```
1 finished
2 finished
3 finished
[ 'data 1', 'data 2', 'data 3' ]
```

接收一组参数，里面的值最终都算返回Promise对象。这样，三个异步操作的并发执行的，等到他们都执行完后才会进到then里面。三个异步操作返回的数据都在then里面。all会把所有异步操作的结果放进一个数组中传递给then。就是上面的results。



**race**

```
Promise.race([runAsync1(), runAsync2(), runAsync3()])
.then(function(results) {
  console.log(results);
});
```

谁跑得快，以谁的为准执行回调。结果如下：

```
1 finished
data 1
2 finished
3 finished
```







