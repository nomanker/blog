---
title: jetbrains系中文乱码解决
categories:
  - 小tips
date: 2021-05-10 20:48:54
tags:
- C++
---

**之前用JetBrains系列的IDE，总是会出现莫名其妙的中文乱码（就很烦），今天才发现这么简单。**

### 1.在Settings中进行设置

IDE Encoding: UTF-8

Project Encoding: UTF-8

Default encoding for properties files: UTF-8

如下图：

![image-20210510205223869](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210510205223869.png)

***

### 2.设置VM Options

Help->Edit Custom VM Options

添加 -Dfile.encoding=UTF-8

如下图：

![image-20210510205400720](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210510205400720.png)

***

