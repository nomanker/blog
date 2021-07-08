---
title: WSL2下的Git搭配VScode使用
categories:
  - 学习之路
date: 2021-07-08 17:42:34
tags:
- Git
- Linux
- WSL
---

我们的目标是能够使得WSL2可以使用Win10中的V2ray快速访问诸如Github等网站

#### WSL1和WSL2网络的区别

WSL1的时代，Linux子系统和Windows系统共享网络端口，因此Linux子系统只需要访问本地127.0.0.1对应端口即可请求通过代理上网。

```bash
export ALL_PROXY="http://127.0.0.1:10809"
```

但是到了WSL2，则Linux子系统和Windows在网络上是两台各自独立的机器。因此，从Linux子系统访问Windows首先要找到Windows的IP。

#### 配置访问Windows代理

通过/etc/resolv.conf 文件可以查看WSL2中Win10的IP

```bash
cat /etc/resolv.conf
```

使用如下命令方便获取IP

~~~bash
cat /etc/resolv.conf|grep nameserver|awk '{print $2}'
~~~

接着在~/.bashrc文件最下方添加如下内容

~~~bash
export windows_host=`cat /etc/resolv.conf|grep nameserver|awk '{print $2}'`
export ALL_PROXY=http://$windows_host:10809
export HTTP_PROXY=$ALL_PROXY
export http_proxy=$ALL_PROXY
export HTTPS_PROXY=$ALL_PROXY
export https_proxy=$ALL_PROXY

if [ "`git config --global --get proxy.https`" != "http://$windows_host:10808" ]; then
            git config --global proxy.https http://$windows_host:10809
fi
~~~

具体协议视情况更改。

#### 在V2Ray中设置允许局域网连接

![image-20210708180042820](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210708180042820.png)

#### 设置Win10防火墙

在Win10防火墙中设置允许V2Ray进行公用和专用网络的访问。

输入curl www.google.com 验证是否成功

***

#### 配置Git

使用git config --list 查看当前配置

配置个人信息

~~~bash
git config --global user.name "xxx"
git config --global user.email "xxxxxxxxx@qq.com"
~~~

创建SSH key

```bash
ssh-keygen -t rsa -C "xxxxxxxx@qq.com" //GitHub上的邮箱
```

无脑三次回车

复制 /home/.ssh/id_rsa.pub 中的内容，在Github的setting中添加一个新的SSH key

配置完成后即可正常使用Github进行项目版本控制

***

#### VScode中相关操作

首先在Linux子系统中切换至项目目录，使用 code . 命令即可在Windows中打开该文件夹（初次使用会自动安装VScode Server）。

在VScode中操作和编写与本地项目无差别，侧边栏同样实时展示文件的更改。

若要将本地项目推送至Github，要将本地仓库和远程仓库建立联系，即添加remote

![image-20210708181526739](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210708181526739.png)

点击后，填写仓库的URL后便和remote仓库建立联系，之后便可正常执行push,pull等操作了。😁