---
layout:     post
title:      Centos7编译安装shadowsocks-libev最新版
subtitle:   
date:       2020-01-16
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - shadowsocks
---

转载:[CentOS7下手动编译安装Shadowsocks-libev最新版](https://b.awei.pub/2019/03/shadowsocks-libev/)

### 1. 编译环境

```
yum install epel-release -y

yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto c-ares-devel libev-devel libsodium-devel mbedtls-devel -y
```

### 2. 下载源码

若没安装 git，则先安装 git.
```
yum install git
```

```
cd /usr/local/src

git clone https://github.com/shadowsocks/shadowsocks-libev.git
```

### 3. 编译

```
cd /usr/local/src/shadowsocks-libev

git submodule update --init --recursive

sh autogen.sh

./configure --disable-documentation

make

make install
```

### 4. 配置

配置/etc/shadowsocks-libev/config.json:

```
cp /usr/local/src/shadowsocks-libev/debian/config.json /etc/shadowsocks-libev/config.json
```

如果出现如下提示
>cp: cannot create regular file ‘/etc/shadowsocks-libev/config.json’: No such file or directory

则先创建shadowsocks-libev目录：
```
 mkdir -p /etc/shadowsocks-libev
```
然后再执行上一步`配置/etc/shadowsocks-libev/config.json:`.

然后根据自己服务器的具体情况，填上相关信息，如：
```
{
    "server":"192.168.1.86",
    "server_port":10086,
    "local_port":1080,
    "password":"ab10086K",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
```

配置shadowsocks.service:
>因为是make安装的，程序默认安装到 `/usr/local` 下面，所以要改下 `/usr/local/src/shadowsocks-libev/rpm/SOURCES/systemd/shadowsocks-libev.service` 文件：

把
```
ExecStart=/usr/bin/ss-server -c "$CONFFILE" $DAEMON_ARGS
```
改为
```
ExecStart=/usr/local/bin/ss-server -c "$CONFFILE" $DAEMON_ARGS
```

然后将相应文件cp到对应目录
```
cp /usr/local/src/shadowsocks-libev/rpm/SOURCES/systemd/shadowsocks-libev.service /usr/lib/systemd/system/

cp /usr/local/src/shadowsocks-libev/rpm/SOURCES/systemd/shadowsocks-libev.default /etc/sysconfig/shadowsocks-libev
```

Linux客户端设置与服务器端相差无几，只不过，要改的文件是shadowsocks-libev-local.service而已，其他一样。

### 5. 启动

```
systemctl enable shadowsocks-libev
systemctl start shadowsocks-libev
systemctl status shadowsocks-libev
```

安装过程中，如果对`配置`有任何更改，只要重启下服务就行了，不用重启服务器，命令如下：
```
systemctl restart shadowsocks-libev
```

### 6. 防火墙设置

```
firewall-cmd --permanent --add-port={PORT/tcp,PORT/udp}
```
PORT/tcp,PORT/udp 中 `PORT` 要替换成自己想要开设的端口号，比如我要开设 tcp 的 80 端口
和 udp 的 443 端口，那么就设置为:
```
firewall-cmd --permanent --add-port={80/tcp, 443/udp}
```

```
firewall-cmd --reload
firewall-cmd --list-all
```

删除已开通端口
```
firewall-cmd --permanent --remove-port=80/tcp 
```


### 7. 客户端连接

在客户端上输入之前设置的IP、密码、等数据，连接服务器

另外：make安装是相当蛋痛的。可以直接用yum源安装
Centos7，只要把[yum源](https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo)添加进去，
yum源:
```
[copr:copr.fedorainfracloud.org:librehat:shadowsocks]
name=Copr repo for shadowsocks owned by librehat
baseurl=https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/epel-7-$basearch/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/pubkey.gpg
repo_gpgcheck=0
enabled=1
enabled_metadata=1
```
然后
```
yum -y install shadowsocks-libev
```
再配置一下/etc/shadowsocks-libev/config.json文件，就万事大吉了。





