---
layout:     post
title:      配置阿里云服务器要做的事
subtitle:   
date:       2019-11-22 21:45:00
author:     hjfrun
header-img: 
catalog: true
tags:
	- Docker
	- NodeJS
	- pm2
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



## 安装NodeJS

0. 先要安装`wget`

1. 再安装`NodeJS`，执行以下命令：

   `wget https://nodejs.org/dist/v12.16.1/node-v12.16.1-linux-x64.tar.xz`

   要注意以上的版本在`NodeJS`的官方网站查看：`https://nodejs.org/dist/v12.16.1/`

2. 解压文件：

   ```bash
   tar xvf node-v12.16.1-linux-x64.tar.xz
   ```

3. 创建软链接，这样可以在任意目录直接使用`node` 和`npm`

   ```bash
   ln -s /root/node-v12.16.1-linux-x64/bin/node /usr/local/bin/node
   ln -s /root/node-v12.16.1-linux-x64/bin/npm /usr/local/bin/npm
   ```

4. 查看`node`、`npm`版本

   ```bash
   node -v
   npm -v
   ```

   

5. 如果要将`node`安装到其他目录，例如`/opt/node/`下请进行如下操作：

   ```
   mkdir -p /opt/node
   mv /root/node-v12.16.1-linux-x64/* /opt/node
   rm -f /usr/local/bin/node
   rm -f /usr/local/bin/npm
   ln -s /opt/node/bin/node /usr/local/bin/node
   ln -s /opt/node/bin/npm /usr/local/bin/npm
   ```



## pm2

说明：安装`pm2`需要借助`node`，要保证已经把`node`安装成功。

### 安装

```bash
npm i pm2 -g
```

需要注意的是，`pm2`安装之后，可能会发现找不到命令的情况，这个时候，要么把`pm`的目录添加到系统的`PATH`，要么向上面一样新建一个软链接。

### 使用：

启动一个服务：

```bash
pm2 start xxx.js --name [appname]
```

查看有多少服务启动：

```bash
pm2 ls
```

重启服务：

```bash
pm2 restart appname/id
```

停止服务：

```bash
pm2 stop appname/id
```

删除服务：

```bash
pm2 delete appname/id
```



 





