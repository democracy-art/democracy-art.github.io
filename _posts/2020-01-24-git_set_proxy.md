---
layout:     post
title:      git设置代理
subtitle:   
date:       2020-01-24
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - git
---
# 设置代理
#### 使用http 代理
```
git config --global http.proxy http://127.0.0.1:10809
git config --global https.proxy https://127.0.0.1:10809
```
#### 使用socks5代理 (git已不推荐)
```
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

# 取消代理
```
git config --global --unset http.proxy
git config --global --unset https.proxy
```
