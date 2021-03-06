---
layout:     post
title:      var & let & const
subtitle:   
date:       2019-06-21 16:02:00
author:     hjfrun
header-img: 
catalog: false
tags:
    - 
---





 **var声明与变量提升**

使用`var`关键字声明的变量，无论其实际声明位置在何处，都会被视为声明于所在函数的顶部（如果声明不在任意函数内，则视为在全局作用域的顶部）。这就是变量提升（`hoisting`）。

```javascript
function getValue(condition) {
	if (condition) {
		var value = "blue";
		
		return value;
	} else {
		// value can be accessed here, value is undefined
		return null;
	}
	// value can also be acessed here, value is undefined
}
```

实际上，`value`无论如何都会被创建。JS引擎会默默将`getValue`函数调整为如下形式：

```javascript
function getVaue(condition) {
	var value;
	if (condition) {
		value = "blue";
		
		return value;
	} else {
		return null;
	}
}
```

`value`变量的声明被提升到了函数顶部，而初始化操作则保留在了原处。这意味着`else`分支内`value`变量也是可以访问的，在此处它的值并未被初始化，因此是`undefined`。



**块级声明**

块级声明也就是让所有声明的变量在指定块的作用域外无法访问。块级作用域（又被称为词法作用域）在如下情况被创建：

1. 在一个函数内部
2. 在一个由一对花括号包裹的代码块内部

块级作用域是很多类C语言的工作机制，ES6引入块级声明，是为了给JS添加灵活性，并保证与其他语言的一致性。



**let声明**

`let`声明的语法与`var`的语法一致。大体上可以用`let`代替`var`进行变量声明，但会将变量的作用域限制在当前代码块中。由于`let`声明并不会被提升，因此若想让变量在整个代码块内部都可用，需要手动将`let`声明放置到代码块顶部。

```javascript
function getValue(condition) {
	if (condition) {
		let value = "blue";
		
		return value;
	} else {
		// value is not accessable here
		return null;
	}
	// value is not accessable here either
}
```

这种写法的`getValue`函数的行为更接近其他类C语言。



**禁止重复声明**

如果一个标识符已经在代码块内部被定义，那么在此代码块内使用同一个标识符进行`let`声明就会导致错误。例如：

```javascript
var count = 30;

// 语法错误
let count = 40;
```

因为`let`不能在同一作用域内重复声明一个已有的标识符，此处的`let`声明就会抛出错误。然而，在嵌套的作用域内使用`let`声明一个同名新变量，就不会有问题：

```javascript
var count = 30;

// 不会抛出异常
if (condition) {
  let count = 40;
}
```

在`if`内部创建的新变量，这个新变量会屏蔽全局的`count`变量，从而在局部阻止对于后者的访问。



**常量声明**

在ES6中也可以使用`const`语法进行声明。使用`const`声明的变量会被认为是常量。

```javascript
// 有效的常量
const maxItems = 30;
// 语法错误:未进行初始化
const name;
```

`maxItems`变量在声明时初始化，因此它的`const`声明是有效的。而`name`变量缺少了初始化操作。

对比常量声明和`let`声明

常量声明和`let`声明一样，都是块级声明。这意味着常量在声明他们的语句块之外都是无法被访问的，并且声明也不会被提升。

```javascript
if (condition) {
  const maxItems = 5;	
}

// maxItems 在此处无法访问
```



`const`和`let`的另外一个相似点是：在同一作用域内定义一个已有变量会抛出错误。



```javascript
const person = {
  name: "Nicholas"
};

// 工作正常
person.name = "Greg";

// 抛出错误
person = {
  name: "Greg"
};
```



**循环中的块级绑定**

```javascript
for (var i = 0; i < 10; i++) {
  process(items[i]);
}

// i在此处仍然可被访问
console.log(i);			// 10
```

```javascript
for (let i = 0; i < 10; i++) {
  process(items[i]);
}

// i在此处不可访问
console.log(i);
```



**循环内的常量声明**

```javascript
var funcs = [];

// 在一次迭代后抛出错误
for (const i = 0; i < 10; i++) {
  funcs.push(function() {
    console.log(i);
  });
}
```

`const`在`for-in`或`for-of`循环中使用时，与`let`变量效果相同。

```javascript
var funcs = [],
    object = {
      a: true,
      b: true,
      c: true
    };

// 不会导致错误
for (const key in object) {
  funcs.push(function(){
    console.log(key);
  });
}

funcs.forEach(function(func) {
  func();			// 依次输出'a', 'b', 'c'
});
```



**全局块级绑定**

`let`与`const`不同于`var`的另外一个方面是在全局作用于上的表现。当在全局作用域上使用`var`时，它会创建一个新的全局变量，并成为全局对象（在浏览器中是`window`）的一个属性。这意味着使用`var`可能会无意中覆盖一个已有的全局属性：

```javascript
// 在浏览器中
var RegExp = "Hello";
console.log(window.RegExp);			// "Hello"

var ncz = "Hi";
console.log(window.ncz);				// "Hi"
```

尽管全局的`RegExp`是定义在`window`上的，它仍然不能防止被`var`重写。在这个例子中`RegExp`覆盖了原有对象。`ncz`在定义为全局变量后就立即成为了`window`的一个属性。



然而如果是在全局作用域上使用`let`或`const`，虽然在全局作用域上会创建新的绑定，但不会有任何属性被添加到全局对象上。这意味着不能使用`let`或`const`来覆盖一个全局变量，只能将其遮蔽。

```javascript
// 在浏览器中
let RegExp = "Hello";
console.log(RegExp);			// "Hello"
console.log(window.RegExp === RegExp);		// false

const ncz = "Hi";
console.log(ncz);			// "Hi"
console.log("ncz" in window);		// false
```

在ES6开发阶段。默认情况下：应当使用`let`来代替`var`。对于多数JS开发者来说，`let`的行为方式正是`var`本应有的方式，因此直接用`let`地台`var`更符合逻辑。在这种情况下，需要收到保护的变量再使用`const`。

然而，一种替代方案变得更为流行：**在默认情况下使用`const`，仅当明确变量需要被更改时才只用`let`。依据是在大部分变量在初始化之后都保持不变。因为逾期外的改动是`bug`的源头之一。**



