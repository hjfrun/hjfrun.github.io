---
layout:     post
title:      React组件生命周期
subtitle:   
date:       2018-05-18 14:24:35
author:     hjfrun
header-img: 
catalog: false
tags:
    - React
---



最近在学习**React**，还是有些笔记要做。



### 组件的生命周期

* 装载过程（Mount），第一次在DOM树中渲染的过程；
* 更新过程（Update），当组件被重新渲染；
* 卸载过程（UNmount），组件从DOM中删除；



#### 装载过程

- constructor
- ~~getInitialState~~
- ~~getDefaultProps~~
- componentWillMount
- render
- componentDidMount

*componentWillMount*都是紧贴着自己组件的`render`函数之前被调用，单*componentDidMount*可不是紧跟着`render`函数之后被调用，当几个组件的`render`函数都调用之后，三个组件的*componentDidMount*才连在一起被调用。



#### 更新过程

* componentWillReceiveProps
* shouldComponentUpdate
* componentWillUpdate
* render
* componentDidUpdate

1. *componentWillReceiveProps(nextProps)*

只要父控件的render函数被调用，在render函数里面被渲染的子组件就会经历更新过程，不管父控件传递给子控件的props有没有改变，都会出发子组件的componentWillReceiveProps函数；

注意，通过this.setState方法出发的更新不会调用这个函数，因为这个函数适合更具新的props值来计算出是不是要更新内部状态state。更新组件内部状态的方法就是this.setState，如果this.setState的调用导致componentWillReceiveProps再被调用，那就是一个死循环了。



#### 卸载过程

* componentWillUnmount



*componentWillUnmount*中的工作往往和*componentDidMount*有关；





