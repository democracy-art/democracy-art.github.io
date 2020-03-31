---
layout: post
title: The usage of command scp on linux
subtitle:
date: 2020-03-25
author: D
header-img:
catalog: true
tags: [linux_basic,scp]
---
### 1. Copy file from remote server to local
```
scp -P remote_port root@remote_ip:/remote_path/filename ./
```
copy directory 
```
scp -P remote_port -r root@remote_ip:/remote_path/dir_name ./
```
Enter remote server password.

### 2. Copy file from local to remote server 
```
scp -P remote_port local_filename root@remote_ip:/remote_path/
```
copy directory
```
scp -P remote_port -r local_dir_name root@remote_ip:/remote_path/
```
Enter remote server password.
