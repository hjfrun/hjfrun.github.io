---
layout:     post
title:      配置阿里云服务器要做的事
subtitle:   
date:       2019-11-22 21:45:00
author:     hjfrun
header-img: 
catalog: false
tags:
    - 
---



## 源更新

```bash
yum update
```

## 安装Docker

```bash
1. yum install docker
2. systemctl enable docker.service
3. groupadd docker (可选)
4. usermod -aG docker dil （可选）
5. systemctl start docker.service
```

## 安装Docker Compose

1. `curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

2. `chmod +x /usr/local/bin/docker-compose`

3. `ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`



## 安装Git

```bash
yum install git
```



 





