---
title: linux下的cmake安装
categories:
  - 学习之路
date: 2021-05-08 18:55:35
tags:
- Linux
- C++
---

今天由于比赛需要，初次接触Ubuntu18.04LTS（以下简称u18），刚开始的时候发现清华源中u18支持的最新的cmake为3.10的，没怎么在意，把其他项目中cmakelists里面的最低版本改下就行了，结果自己要写点东西的时候，因为使用的是c++17所以较低版本的cmake并不支持，于是只好学习下Linux下怎们在不使用源的情况下升级cmake。

***

### 1.cmake的安装

首先卸载旧版本的cmake（非必须，因为使用soft link-软连接可以指定cmake安装的版本）。

~~~bash
sudo apt-get autoremove cmake
~~~

将cmake源码放到目录中（记得住就行，本人为opt目录），使用如下命令解压该文件

```bash
sudo tar -zxvf cmake-xxxxxx.tar.gz
```

安装cmake，在安装前确保已经装了libssl-dev和openssl

```bash
cd cmake-xxxxx
sudo chmod +x bootstrap
sudo ./bootstrap
sudo make
sudo make install
```

安装大概需要二十分钟的时间，安装完成后使用cmake --version查看cmake的版本来确认是否安装成功

***

### 2.建立软连接

使用如下命令建立软连接

```bash
sudo ln -sf /opt/cmake-*****/bin/*  /usr/bin
```

**软连接：**

软连接是Linux中的一个常用命令，它的功能是为某一个文件在另一个位置建立一个同步的链接。当我们需要在不同的目录用到相同的文件时，我们不需要在每一个目录下都放一个相同的文件，我们只要在其他的目录下用ln命令链接（link）即可，不必重复占用磁盘空间。

* 创建软连接

具体用法是：ln -s 源文件 目标文件

当前路径下创建test1.jar引向/opt/test2.jar

```bash
ln -s /opt/test2.jar test1.jar
```

* 删除软连接

```bash
rm -rf test1.jar
```

* 修改软连接

ln -snf 新的源文件或目录 目标文件或目录

```bash
ln -snf /opt/cmake-*****/bin/*  /usr/bin
```

* 常用参数
  1. -f : 链结时先将与 dist 同档名的档案删除
  2. -d : 允许系统管理者硬链结自己的目录
  3. -i : 在删除与 dist 同档名的档案时先进行询问
  4. -n : 在进行软连结时，将 dist 视为一般的档案
  5. -s : 进行软链结(symbolic link)
  6. -v : 在连结之前显示其档名
  7. -b : 将在链结时会被覆写或删除的档案进行备份
  8. -S SUFFIX : 将备份的档案都加上 SUFFIX 的字尾
  9. -V METHOD : 指定备份的方式
  10. --help : 显示辅助说明
  11. --version : 显示版本　

**硬连接：**

硬连接指通过索引节点来进行连接。在Linux的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号(Inode Index)。在Linux中，多个文件名指向同一索引节点是存在的。一般这种连接就是硬连接。硬连接的作用是允许一个文件拥有多个有效路径名，这样用户就可以建立硬连接到重要文件，以防止“误删”的功能。其原因如上所述，因为对应该目录的索引节点有一个以上的连接。只删除一个连接并不影响索引节点本身和其它的连接，只有当最后一个连接被删除后，文件的数据块及目录的连接才会被释放。也就是说，文件真正删除的条件是与之相关的所有硬连接文件均被删除。
