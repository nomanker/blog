---
title: WSL的SSH配置以及迁移出系统盘
categories:
  - 学习之路
date: 2021-05-07 21:42:46
tags:
- Linux
- WSL
---

### 1.安装WSL

从Microsoft Store中下载相应的Linux发行版（Ubuntu，Kali等），安装即可，此时会默认装到C盘中，刚刚装上时系统占用空间可能还比较小，只有几百兆，用的时间久的话就会超出几个G了，原本C盘吃紧的话就会很难受了，后面会有迁移的方法。

![image-20210507214929847](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210507214929847.png)

***

### 2.配置SSH

首先让我们搜索关键词Clion WSL（毕竟我们的目的就是要用WSL进行Linux开发的），Jetbrains官方是有给教程的

[Jetbrains官方教程]: https://www.jetbrains.com/help/clion/how-to-use-wsl-development-environment-in-product.html

只不过由于一些网络原因，中间wget的那一步会出现问题，没办法，我们只好手动写一个script。

![image-20210507215407720](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210507215407720.png)

点击这个script链接，会跳转到GitHub的一个仓库clion-wsl中，然后我们进入WSL子系统中，在一个好记的目录下（个人偏向于在bin目录下）创建名为**ubuntu_setup_env.sh**的文件，并把仓库中对应的文件中的内容粘贴到子系统的**ubuntu_setup_env.sh**中，保存退出后，使用

```bash
sudo chmod +x ubuntu_setup_env.sh
```

更改权限，然后使用

```bash
./ubuntu_setup_env.sh
或
bash ubuntu_setup_env.sh
```

运行该脚本，脚本运行成功后在Windows中terminal中使用ssh username@localhost -p自定义的端口,看看是否能够成功连接。

ssh端口默认是开在2222端口的，因此如果一台主机上有多个WSL的话，需要在ubuntu_setup_env.sh中手动更改端口。

***

### 3.将WSL迁移至系统盘

感谢GitHub上的一个dalao写的软件LxRunOffline，

[LxRunOffline]: https://github.com/DDoSolitary/LxRunOffline

直接去releases里面下载最新的就可以了

**具体用法：**

* 显示电脑上安装了哪些子系统

  ```bash
  LxRunOffline.exe list
  ```

* 将子系统迁移至指定目录

  ~~~bash
  LxRunOffline.exe move -n ubuntu18.04 -d D:\wsl\ubuntu18.04
  ~~~

迁移的过程中可能会出现一些warning，但是也只是警告，无大碍。

教程参考：https://p3terx.com/archives/manage-wsl-with-lxrunoffline.html