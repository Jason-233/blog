---
title: "清理Debian(openstick棒子)空间"
date: 2024-03-29T14:31:08+08:00
lastmod: ":fileModTime"
draft: false
---

我刷的openstick容量不足，上网搜了下怎么清理。

# 查看空间

当然，首先得看看到底哪儿占地方。

**查看某个目录的文件大小并排序**（单位为MB）

```shell
du -hm --max-depth=1 /var/ | sort -n
```

用 `du -t 100M /var `或 `journalctl --disk-usage `命令查看日志占用。

# 日志清理

首先就是这个，我一看还有随着系统一起的21年的日志，自然是不用的。

默认日志存放在`/var/log/journal/d2bad6894db34e26d7cedceb6288ea82`后面一串是机器码，可以用`hostnamectl`查看。

**journalctl命令的用法**

```shell
journalctl
-n 3                             ##日志的最新3条
--since "2020-05-01 11:00:00"    ##显示11：00后的日志
--until "2020-05-01 11:05:00"    ##显示日志到11：05
-o                               ##设定日志的显示方式
                                 # short 经典模式显示日志
                                 # verbose 显示日志的全部字节
                                 # export 适合传出和备份的二进制格式
                                 # json js格式显示输出
-p                               ##显示制定级别的日志
                                 #0 emerg 系统的严重问题日志
                                 #1 alert 系统中立即要更改的信息
                                 #2 crit 严重级别会导致系统软件不能正常工作
                                 #3 err 程序报错
                                 #4 warning 程序警告
                                 #5 notice 重要信息的普通日志
                                 #6 info 普通信息
                                 #7 debug 程序排错信息
-F                               ##查看某一列信息
                                 # PRIORITY 查看可控日志级别
-u sshd                          ##指定查看服务
#实现日志清理：
--disk-usage                     ##查看日志大小
--vacuum-size=1G                 ##设定日志存放大小
--vacuum-time=1W                 ##日志在系统中最长存放时间
-f                               ##监控日志
```

参考：[Linux journalctl命令详解（journalctl指令、journal命令）（systemd服务默认日志管理工具）-CSDN博客](https://blog.csdn.net/Dontla/article/details/132415985)

# apt 缓存

### apt autoclean

如果你的硬盘空间不大的话，可以定期运行这个程序，将已经删除了的软件包的.deb安装文件从硬盘中删除掉。如果你仍然需要硬盘空间的话，可以试试apt clean，这会把你已安装的软件包的安装包也删除掉，当然多数情况下这些包没什么用了，因此这是个为硬盘腾地方的好办法。


### apt clean

类似上面的命令，但它删除包缓存中的**所有**包。这是个很好的做法，因为多数情况下这些包没有用了。但如果你是拨号上网的话，就得重新考虑了。



### apt autoremove

删除为了满足其他软件包的依赖而安装的，但现在不再需要的软件包。



### apt remove

删除已安装的软件包（保留配置文件）。

### apt remove --purge

删除已安装包（不保留配置文件`)。`

# 总结

目前大概就这样，我还删去了不用的docker镜像。小空间没办法，就折腾呗。
