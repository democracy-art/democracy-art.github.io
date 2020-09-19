--- 
layout: post
title: HTTP Headers解析 
subtitle:
date: 2020-09-19
author: D
header-img:
catalog: true
tags: [http]
---

- `X-Request-Id` : 客戶端創建的隨機的ID，且把它上傳給服務器.服務器會把X-Request-Id包含在它自己創建的日記里面.這樣如果一個客戶端接收到錯誤，那麼它可以在錯誤報告中包含X-Request-Id，這樣就允許服務器管理員根據X-Request-Id去精準定位相關日記(這樣就不用依賴時間戳，IPs,等等這些東西了).
- X-Request-Id 解決的問題: 
    - 1.客户端访问的Web服务时，如何将客户端请求与服务端日志关联 
    - 2.微服务架构下，访问日志如何查询
    - 3.不同项目交互出现异常，如何做日志关联
