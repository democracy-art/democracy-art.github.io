--- 
layout: post
title: FreeBSD SSH 安全配置 
subtitle:
date: 2020-09-19
author: D
header-img:
catalog: true
tags: [freebsd, ssh]
---

# 1.配置`/etc/ssh/sshd_config`
```
# vi /etc/ssh/sshd_config
```

1.1 禁止使用root登錄ssh
```
#PermitRootLogin yes
PermitRootLogin no
```
>默認允許root登錄,所以值默認爲`yes`,把它改爲`no`.

1.2 修改ssh默認端口,默認爲`22`,修改1024-65535 之間的值(<1024值爲系統預留端口).比如:
```
#Port 22
Port 21892
```

1.3 禁止密碼登錄(做這一步之**前**, 先要設置ssh密鑰登錄)
```
PasswordAuthentication no
```

1.4 啓用pubkey認證登錄
```
PubkeyAuthentication yes
```
1.5 重啓ssh
```
# service sshd restart
```

# 2.設置ssh密鑰登錄

2.1 在ssh客戶端生成密鑰對
2.1.1 創建存儲密鑰對的目錄
```
$ mkdir -p  .ssh/freebsd_ip1
```
2.1.2 生成密鑰對
```
$ ssh-keygen -t rsa
```
- `-t` 參數可以指定四種算法類型:[-t dsa | ecdsa | ed25519 | rsa] 這裏選擇 rsa;
```
Generating public/private rsa key pair.
Enter file in which to save the key (/path/to/.ssh/id_rsa): /path/to/.ssh/freebsd_ip1/id_rsa (輸入要存私鑰的絕對路徑)
Enter passphrase (empty for no passphrase): 回車(或者輸入密碼)
Enter same passphrase again:  回車(或者再次輸入密碼)
Your identification has been saved in /path/to/.ssh/freebsd_ip1/id_rsa
Your public key has been saved in /path/to/.ssh/freebsd_ip1/id_rsa.pub
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxyyyyyyyyyyyyyyyyzzzzzzzzzzzz user@linux
The key's randomart image is:
+---[RSA 3072]----+
|        .... .. o|
|         ..o.+o..|
|          o =o*o |
|E    . . . * oo..|
|..    + S = o. +.|
|+..    = + . o= .|
|o+.   . +     ...|
|.o=. .         . |
|**+=.            |
+----[SHA256]-----+
```
2.1.3 查看密鑰對
```
$ ls  .ssh/freebsd_ip1/
```
> 會看到密鑰對: id_rsa(私鑰) 和 id_rsa.pub(公鑰)

2.3 複製公鑰到ssh服務器，使用 `ssh-copy-id` 命令
**不**建議使用root用戶登錄ssh,所以這裏使用普通用戶登錄ssh爲例子:
```
$ ssh-copy-id -i .ssh/freebsd_ip1/id_rsa.pub user1@server_ip -p port (如果沒有改ssh端口, 不用加 "-p port")
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/freebsd_ip1/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password for user1@: 輸入user1 用戶的密碼

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh -p 'port' 'user1@server_ip'"
and check to make sure that only the key(s) you wanted were added.
```
2.4 使用ssh公鑰登錄遠程服務器 
```
$ ssh -p port -i .ssh/freebsd_ip1/id_rsa user1@server_ip
```
