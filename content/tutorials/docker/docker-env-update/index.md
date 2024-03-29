---
title: "已启动的Docker容器，如何修改其中env使之长期有效"
date: 2024-03-29T14:33:08+08:00
lastmod: ":fileModTime"
draft: false
---





转载自[Docker已经创建运行启动的容器，如何修改容器中的环境变量env使长期有效_修改容器的环境变量-CSDN博客](https://blog.csdn.net/lishuoboy/article/details/130174200?ops_request_misc=%7B%22request%5Fid%22%3A%22171169373916800182760412%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fblog.%22%7D&request_id=171169373916800182760412&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-130174200-null-null.nonecase&utm_term=docker&spm=1018.2226.3001.4450)

## 1.查看Docker Root目录

```shell
docker info | grep 'Docker Root'
```

> [root@[jenkins](https://so.csdn.net/so/search?q=jenkins&spm=1001.2101.3001.7020) ~]# docker info | grep ‘Docker Root’
> Docker Root Dir: /data/docker

## 2.查到容器的长id（container id）

方式一：

```shell
docker inspect pdmaas | grep "Id"
```

方式二：

```shell
docker ps -a --no-trunc | grep  pdmaas
```

> docker ps -a --no-trunc |grep pdmaas
> 2bd5ad1314bfff05099142aae2f896fc4c3ee6b640160d27fb7c4d8ce1d5aead pdmaas:1.3.2 “bash start.sh” 4 weeks ago Exited (137) 28 minutes ago pdmaas

## 3.停止容器

```shell
docker stop [容器名|容器id]
```

## 4.编辑修改环境变量env

***建议：修改前先备份***
***建议：修改前先备份***

```shell
vim ${Docker Root}/containers/${container-id}/config.v2.json
```

或

```shell
vim ${Docker Root}/containers/${container-id}/config.json
```

json文件代码是压缩的，可以使用`sz path`命令下载下来格式化后再编辑，再用`rz -y`命令上传覆盖
![在这里插入图片描述](https://img-blog.csdnimg.cn/0a627f4971ac4f8688058fe54d215074.png)

## 5.重载服务的配置文件

```shell
systemctl daemon-reload
```

## 6.重启docker

```shell
systemctl restart docker
```

# 阅读心得

还有一种方法是commit当前的容器为一个新的镜像，再在这之上来run。如果不是一启动就是用的env，还可以`docker exec -it <container-id> /bin/sh export xx=xxx`。

总之最好是在run之前就处理好这些。
