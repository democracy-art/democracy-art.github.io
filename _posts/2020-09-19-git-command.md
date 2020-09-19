--- 
layout: post
title: git常用命令 
subtitle:
date: 2020-09-19
author: D
header-img:
catalog: true
tags: [git]
---
# 1.本地與遠程互動操作
- `git clone` <遠程庫地址> : 克隆遠程庫
> 功能: 1. 完整的克隆遠程庫爲本地庫 2.爲本地庫新建 origin 別名 3.初始化本地庫

- `git push` <別名> <分支名> : 本地庫某個分支推送到遠程庫，分支必須指定

- `git pull` <別名> <分支名> : 把遠程庫的修改拉取到本地
> tip: 該命令包括 git fetch, git merge

- `git fetch` <遠程庫別名> <遠程庫分支名> : 抓取遠程庫的指定分支到本地，但沒有合并

- `git merge` <遠程庫別名/遠程庫分支名> : 將抓取下來的遠程的分支，跟當前所在的分支進行合并

- `git remote -v` : 查看遠程庫地址別名

- `git remote add` <別名> <遠程庫地址> : 新建遠程庫地址別名

- `git remote rm` <別名> : 刪除本地中遠程庫別名

- `git fork` : 複製遠程庫
> tip: 一般是外面团队的开发人员fork本团队项目，然后进行开发，之后外面团队发起pull request，然后本团队进行审核，如无问题本团队进行merge（合并）到团队自己的远程库，整个流程就是本团队跟外面团队的协同开发流程，Linux的团队开发成员即为这种工作方式。

# 2.本地操作
### 2.1 其它 
- `git init` : 初始化本地庫
- `git status` : 查看工作區，暫存區的狀態
- `git add <filename>` : 將工作區的**新建/修改**添加到暫存區
- `git commit <filename>` : 將暫存區的內容提交到本地庫
> tip: 需要再編輯提交日記，比較麻煩建議用下面帶參數的提交方法
- `git commit -m "提交日記的內容" <filename>` : 文件從暫存區到本地庫

### 2.2 日誌
- `git log` : 查看歷史提交
> tip: 空格向下翻頁， b向上翻頁，q退出

### 2.3 版本控制
### 2.4 比較差異
### 2.5 分支操作
