---
layout:     post
title:      Python Review
subtitle:   
date:       2019-07-18 10:10:00
author:     hjfrun
header-img: 
catalog: false
tags:
    - 
---



#### 1. 除法问题

```python
>>> 8 / 5			# division always returns a floating point number
1.6
>>> 17 / 3		# classic division returns a float
5.66666667
>>> 17 // 3		# floor division discards the fractional part
```

####  

#### 2. 交互模式下，上一次打印出来的表达式被赋值给变量`_`。

```python
>>> tax = 12.5 / 100
>>> price = 100.5
>>> price * tax
12.5625
>>> price + _
113.0625
>>> round(_, 2)
113.06
```



#### 3. 索引

对于使用非负索引的切片，如果索引不越界，那么得到的切片长度就是起止索引之差。例如， `word[1:3]` 的长度为2.

使用过大的索引会产生一个错误: 

```python
>>> word[42]  # the word only has 6 characters
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: string index out of range
```

但是，切片中的越界索引会被自动处理:

```python
>>> word[4:42]
'on'
>>> word[42:]
''
```

#### 4. 解包参数列表

```python
>>> list(range(3, 6))            # normal call with separate arguments
[3, 4, 5]
>>> args = [3, 6]
>>> list(range(*args))            # call with arguments unpacked from a list
[3, 4, 5]
```



#### 5. 循环的技巧

当在字典中循环时，用`items()`方法可将关键字和对应的值同时取出

```python
knights = {'gallahad': 'the pure', 'robin': 'the brave'}
for k, v in knights.items():
    print(k, v)
```

当在序列中循环时，用 `enumerate()` 函数可以将索引位置和其对应的值同时取出

```python
for i, v in enumerate(['tic', 'tac', 'toe']):
    print(i, v)
```

当同时在两个或更多序列中循环时，可以用 [`zip()`](https://docs.python.org/zh-cn/3/library/functions.html#zip) 函数将其内元素一一匹配。

```python
questions = ['name', 'quest', 'favorite color']
answers = ['lancelot', 'the holy grail', 'blue']
for q, a in zip(questions, answers):
    print('What is your {0}? It is {1}.'.format(q, a))
    
    
What is your name? It is lancelot.
What is your quest? It is the holy grail.
What is your favorite color? It is blue.
```



