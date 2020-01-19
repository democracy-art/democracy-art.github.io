---
layout:     post
title:      Centos7安装shadowsocks-libev服务端(最新版)支持多用户
subtitle:   
date:       2020-01-19
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - shadowsocks
---

转载：[centos7下shadowsocks服务端的安装配置与自启动](https://wonderlust91.github.io/shadowsocks%E7%9A%84%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE%E4%B8%8E%E8%87%AA%E5%90%AF%E5%8A%A8/)

**注意：本文所有操作都是默认使用 `root` 用户，默认服务器系统为centos7**

# 安装shadowsocks
### 添加epel源
首先添加epel源，以保证能正确安装shadowsocks的各项依赖
```
yum install epel-release -y
```
安装完成后清理配置并重建缓存
```
yum clean all
yum makecache
```
更新
```
yum update
```
### 安装shadowsocks
首先安装git并用git下载源码
```
yum install git -y
```
git安装完成后下载shadowsocks源码
```
git clone https://github.com/shadowsocks/shadowsocks-libev.git
```
改变工作目录到刚下载的源码目录
```
cd shadowsocks-libev
```
更新子模块
```
git submodule update --init --recursive
```

接下来安装shadowsocks的各种依赖
要注意输出的安装信息确保所有依赖正确安装，如各别依赖无法自动安装则需手动安装，一般正确更新了epel源都可以自动安装所有依赖
```
yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto c-ares-devel libev-devel libsodium-devel mbedtls-devel -y
```

依赖全部成功安装后，用源码编译安装shadowsocks<br>
**注意当前工作目录应为刚下载的shadowsocks-libev源码目录**
```
./autogen.sh && ./configure && make
```
make完成后进行安装
```
make install
```

# 用户配置
### 单用户配置
在任意目录下建立一个配置文件，例如在root目录下建立ss_serverConfig
```
vi /root/ss_serverConfig
```
```
{
    "server": "111.111.111.111", # 你vps的ip地址
    "server_port": 8388, # shadowsocks server要使用的端口，自行设定
    "local_port": 1080, # 本地端口，自行设定
    "password": "barfoo!", # 密码，自行设定
    "timeout": 600, # 超时毫秒数
    "method": "chacha20-ietf-poly1305" # 加密方式，建议优先选chacha20-ietf-poly1305，若客户端不支持，可选aes-256-cfb
}
```
写好配置文件后运行ss-server（非libev版使用ssserver命令）
```
ss-server -c /root/ss_serverConfig 
```
`-c` 参数是使用配置文件

### 多用户配置
多用户配置文件与单用户不同，我们还是在root下创建一个ss_managerConfig
```
vi /root/ss_managerConfig
```
```
{
    "server": "111.111.111.111", # 你vps的ip地址
    "local_port": 1080, # 本地端口，自行设定
    "timeout": 600, # 超时毫秒数
    "method": "chacha20-ietf-poly1305", # 加密方式，建议优先选chacha20-ietf-poly1305，若客户端不支持，可选aes-256-cfb
    "port_password": {
        "8388": "barfoo1", # 端口号与密码
        "8389": "barfoo2" 
    }
}
```
多用户配置时，应用ss-manager而不是ss-server
```
ss-manager -c /root/ss_managerConfig
```

这时我们发现不论是ss-server还是ss-manager都是前台程序，挂在前台执行，监听端口，使用ctrl+c停止后，服务也就停止了。有两个方法使程序运行到后台。

1.程序运行后，我们可以使用ctrl+z键使程序挂起到后台，使用jobs命令查看程序编号，
若后台无其他程序一般编号为1，再用bg 1命令，使程序在后台运行。<br>
2.也可以在程序运行代码的后面加一个&符号，使程序一开始就在后台运行。<br>

这时看起来一切都没问题了，可是当我们退出登陆我们的vps后，发现服务又出问题了。原因是后台运行的服务程序与登陆账户是相关的，当登陆账户退出时，
通过该账户运行的后台程序也会接到退出信号。这时我们可以使用nohup使程序与登陆账户无关，不再接收账户退出时的退出信号。

单用户:
```
nohup ss-server -c /root/ss_serverConfig &
```
多用户时则用ss-manager:
```
nohup ss-manager -c /root/ss_managerConfig &
```

# shadowsocks开机自启动
通过上面的文章，我们已经可以使shadowsocks服务良好地运行了，但你一定不想每次重启服务器时都要运行一遍上面的代码。那么我们就可以使用centos7的systemctl将程序注册为一个随着开机自动启动的服务。

**注意：该方法基于centos7，不适用centos6。**

centos7 设置自启动时，应在/usr/lib/systemd/system下新建一个.service文件，例如ss.service。
```
vi /usr/lib/systemd/system/ss.service
```
```
[Unit]
Description=shadowsocks manager # 服务的描述
After=network.target # 该服务跟在哪个服务后启动

[Service]
Type=forking # 启动时的进程行为，forking是以fork形式从父进程创建子进程，子进程创建后父进程退出
ExecStart=/root/startSS # 启动服务时执行的命令
PrivateTmp=true # 使用私有临时文件目录

[Install]
WantedBy=multi-user.target # 附挂在multi-user.target下
```
本来直接在ExecStart处填nohup /usr/local/bin/ss-manager -c /path/to/config/file &
但是ExecStart指令串仅接受“指令 参数 参数…”的格式，不能接受<，>，>>，|，&等特殊字符，很多的 bash 语法也不支持。
所以，要使用这些特殊的字符时，应直接写入到指令脚本里面。故用将以上命令写在一个bash可执行文件中，例如startSS。
```
vim /root/startSS
```
```
#!/bin/bash
nohup /usr/local/bin/ss-manager -c /path/to/config/file &
```
保存文件后增加执行权限
```
chmod 754 startSS
```
所有配置文件与脚本都设置完毕，使用systemctl开启服务。
更改.service文件后，都要重载守护进程更新systemctl
```
systemctl daemon-reload
```
启动服务
```
systemctl start ss.service
```
查看服务状态，若为绿色active(running)则说明服务成功启动
```
systemctl status ss.service
```
设置服务开机自启动
```
systemctl enable ss.service
```
还可使用 `systemctl restart ss.service` 重启服务，使用 `systemctl stop ss.service` 停止服务

