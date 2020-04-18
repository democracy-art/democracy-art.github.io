--- 
layout: post
title: Git restore deleted files
subtitle:
date: 2020-04-18
author: D
header-img:
catalog: true
tags: [git]
---

1.Get all the commits which have deleted files and the files deleted.
```
git log --diff-filter=D --summary
```
For example, I got as following message:
```
commit dee07eecb2705991d8cb29f805b4ae7afa5ec956
Author: dm116 <nonofrist010@gmail.com>
Date:   Sat Apr 18 09:09:40 2020 +0800

    add guiguzi pdf

 delete mode 100644 _posts/2020-04-17-guiguzi_1.md
 delete mode 100644 _posts/2020-04-17-guiguzi_10.md
 delete mode 100644 _posts/2020-04-17-guiguzi_11.md
 delete mode 100644 _posts/2020-04-17-guiguzi_12.md
 delete mode 100644 _posts/2020-04-17-guiguzi_2.md
 delete mode 100644 _posts/2020-04-17-guiguzi_4.md
 delete mode 100644 _posts/2020-04-17-guiguzi_5.md
 delete mode 100644 _posts/2020-04-17-guiguzi_6.md
 delete mode 100644 _posts/2020-04-17-guiguzi_7.md
 delete mode 100644 _posts/2020-04-17-guiguzi_8.md
 delete mode 100644 _posts/2020-04-17-guiguzi_9.md

commit e6c3e7fc20d0fe1e722312f1dcf4a6bbba23f401
Author: dm116 <59077166+dm116@users.noreply.github.com>
Date:   Sat Mar 28 13:17:08 2020 +0800

    Delete shuangpin.html

 delete mode 100644 shuangpin.html
```
2.To restore the deleted files
```
git checkout commits^ path/to/filename
```
For example, I want to restore `_posts/2020-04-17-guiguzi_2.md` I should do as following:
```
git checkout commits^ _posts/2020-04-17-guiguzi_2.md
```

Reference:<br>
[Find and restore a deleted file in a Git repository](https://stackoverflow.com/questions/953481/find-and-restore-a-deleted-file-in-a-git-repository?rq=1)
