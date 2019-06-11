---
layout:     post
title:      Mongo notes
subtitle:   
date:       2019-06-11 11:37:35
author:     hjfrun
header-img: 
catalog: false
tags:
    - 
---



# Learning Mongo

进入MongoDB命令行： `mongo`，默认为`27017`端口，如果需要修改可以使用类似于`mongo --port 30017`

显示有哪些数据库：`show dbs`

切换到数据库：`use xxx`，`xxx`表示数据库名

显示当前数据库有哪些collections：`show collections`

查找数据库：

1）`db.getCollection('job').find({})`：找出`job`下所有的条目，也可以简写为`db.getCollection('job').find()`;更可以简写为`db.job.find()`;

```
{ "_id" : ObjectId("5cff6905aa70680dc1532801"), "jobId" : "EDGEAU1.ABC123", "status" : [ { "stateId" : "3000", "timestamp" : ISODate("2019-06-11T08:40:37.899Z") }, { "stateId" : "3001", "timestamp" : ISODate("2019-06-11T08:41:01.049Z") } ] }
{ "_id" : ObjectId("5cff692daa70680dc1532802"), "jobId" : "EDGEAU1.ABC124", "status" : [ { "stateId" : "3000", "timestamp" : ISODate("2019-06-11T08:41:17.752Z") }, { "stateId" : "3005", "timestamp" : ISODate("2019-06-11T08:41:26.153Z") }, { "stateId" : "3010", "timestamp" : ISODate("2019-06-11T08:41:32.182Z") } ] }
{ "_id" : ObjectId("5cff6950aa70680dc1532803"), "jobId" : "EDGEAU1.ABC125", "status" : [ { "stateId" : "3020", "timestamp" : ISODate("2019-06-11T08:41:52.455Z") } ] }
```



2）`db.getCollection('job').find({"jobId" : "EDGEAU1.ABC123"})`：键值对查找；

```
{ "_id" : ObjectId("5cff6905aa70680dc1532801"), "jobId" : "EDGEAU1.ABC123", "status" : [ { "stateId" : "3000", "timestamp" : ISODate("2019-06-11T08:40:37.899Z") }, { "stateId" : "3001", "timestamp" : ISODate("2019-06-11T08:41:01.049Z") } ] }
```

3) `db.getCollection('job').find({"jobId":"EDGEAU1.ABC123"},{"_id": false, "jobId": false})`：结果隐藏某些列；

```
{ "status" : [ { "stateId" : "3000", "timestamp" : ISODate("2019-06-11T08:40:37.899Z") }, { "stateId" : "3001", "timestamp" : ISODate("2019-06-11T08:41:01.049Z") } ] }
```







**restart mongo service**: `brew services restart mongodb-community`

 





