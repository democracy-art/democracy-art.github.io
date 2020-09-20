--- 
layout: post
title: 在FreeBSD12.0上用Let's Encrypt 申請SSL安全加固Nginx 
subtitle:
date: 2020-09-20
author: D
header-img:
catalog: true
tags: [freebsd, nginx, let's encrypt, ssl]
---
# 1.安裝Certbot
首先，获取端口树的压缩快照
```
# portsnap fetch
```
该命令可能需要几分钟的时间才能完成。 完成后，提取快照
```
# portsnap extract
```
此命令可能还需要一段时间才能完成。 完成后，导航至端口树内的py-certbot目录
```
# cd /usr/ports/security/py-certbot
```
然后使用make命令下载并编译Certbot源代码
```
# make install clean
```
接下来，导航到端口树中的py-certbot-nginx目录
```
# cd /usr/ports/security/py-certbot-nginx
```
从此目录再次运行make命令。 这将为Certbot安装nginx插件，我们将使用该插件获取SSL证书
```
# make install clean
```
在安装此插件的过程中，您将看到几个蓝色的对话框，如下所示
![py-certbot-nginx](/img/py-certbot-nginx.png)

这些使您可以选择安装插件及其依赖项的文档。 就本教程而言，您可以按ENTER键以接受将安装本文档的这些窗口中的默认选项。

# 2.设置防火墙并允许HTTP 和 HTTPS访问.
可以查看下面的鏈接如何設置 PF 防火牆.
[FreeBSD12.0 安裝防火牆PF(Packet Filter)](https://dm116.github.io/2020/09/19/freebsd-pf/)

# 3.獲取 SSL 證書
Certbot提供了多种通过各种插件获取SSL证书的方法。 Nginx插件将负责重新配置Nginx并重新加载配置文件：
```
# certbot --nginx -d example.com -d www.example.com
```
如果这是您第一次在此服务器上运行certbot，则客户端会提示您输入电子邮件地址并同意“让我们加密”服务条款。 
完成此操作后，certbot将与Let's Encrypt服务器通信，然后进行质询以验证您是否控制了要为其申请证书的域。

如果挑战成功，Certbot将询问您如何配置HTTPS设置：
```
. . .
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
```
选择您的选择，然后按<kbd>Enter</kbd> 这将更新配置并重新加载Nginx以获取新设置。
certbot将以一条消息结束，告诉您该过程已成功完成，并且证书的存储位置：
```
Output
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /usr/local/etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /usr/local/etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2018-09-24. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /usr/local/etc/letsencrypt. You should
   make a secure backup of this folder now. This configuration
   directory will also contain certificates and private keys obtained
   by Certbot so making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
完成後使用 `https://` 方式從新訪問你的網站.

# 4.验证Certbot自动续订
要测试续订过程，可以使用certbot进行试运行：
```
# certbot renew --dry-run
```
如果没有看到错误，则说明您都准备创建新的crontab：
```
# crontab -e
```
这将打开一个新的crontab文件。 将以下内容添加到新文件中，该文件将告诉
cron每天中午和午夜两次运行certbot renew命令. certbot renew检查系统上
是否有任何即将到期的证书，并在必要时尝试更新：
```
0 0,12 * * * /usr/local/bin/certbot renew
```
如果自动续订过程失败，Let’s Encrypt将向您指定的电子邮件发送一条消息，在证书即将过期时警告您。
