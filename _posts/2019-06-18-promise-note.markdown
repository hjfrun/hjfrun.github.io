---
layout:     post
title:      Promise
subtitle:   
date:       2019-06-18 16:52:00
author:     hjfrun
header-img: 
catalog: false
tags:
    -
---



**基本用法**

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

以上该代码中，执行一个异步操作，也就是`setTimeout`，2秒之后输出`mission completed`，并且调用`resolve`方法。



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

在`runAsync()`的返回上直接调用`then`方法，`then`接收一个参数，是函数。并且会拿到我们在`runAsync`中调用`resolve`时传递的参数。运行这段代码，2秒后执行`mission completed`，紧接着输出`whatever data`。

`then`里面的函数跟我们平时的回调函数一个意思，能够在`runAsync`这个异步任务执行完成之后被执行。这个就是`Promise`的作用了。简单来说，就是把原来的回调方法分离出来。在异步操作执行完后，用链式调用的方式执行回调函数。



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

以上代码效果也是一样的。既然如此，还要`Promise`干嘛？问题是在处理多层回调的情况！

如果`callback`也是一个异步操作，而且执行完成后也需要有相应的回调函数，该怎么办？总不能再定义一个`callback2`，然后给`callback`传进去吧。而`Promise`的优势在于，可以在`then`方法中继续写`Promise`对象并返回，然后继续调用`then`来进行回调操作。



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

`getNumber`函数用来异步获取一个数字，1秒后执行完成，如果数字小于等于5，我们认为是'成功'了，调用`resolve`修改`Promise`的状态。否则，我们认为'失败'了，调用`reject`并传递一个参数，作为失败的原因。

运行`getNumber`并且在`then`中传递两个参数，`then`方法可以接受两个参数，第一个是对应`resolve`的回调，第二个是对应`reject`的回调。所以我们能够分别拿到他们传过来的数据。多次运行，会随机得到下面的结果。

```
rejected
too large number
```

```
resolved
4
```



**catch**

`catch`其实和`then`的第二个参数一样，用来指定`reject`的回调。不过它还有另外一个作用：在执行`resolve`的回调时，如果跑出了异常，那么不会报错卡死`js`，而是进到这个`catch`方法中。

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

接收一组参数，里面的值最终都算返回`Promise`对象。这样，三个异步操作的并发执行的，等到他们都执行完后才会进到`then`里面。三个异步操作返回的数据都在`then`里面。`all`会把所有异步操作的结果放进一个数组中传递给`then`。就是上面的`results`。



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







