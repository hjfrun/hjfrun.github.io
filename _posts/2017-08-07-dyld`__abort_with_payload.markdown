---
layout:     post
title:      dyld`__abort_with_payload
subtitle:   
date:       2017-08-07 16:17:35
author:     hjfrun
header-img: 
catalog: false
tags:
    - bugfix
---

**问题发生情形**：

workspace下集成了多个project。各project之间存在依赖情况。

模拟器可以调试，但是真机一跑起来就挂。

![crash](https://i.stack.imgur.com/7gCPD.png)

`dyld``__abort_with_payload`



**解决方法**：

把嵌入的自定义framework在Target/General界面添加。