--- 
layout: post
title: Set Timezone on Kali Linux
subtitle:
date: 2020-04-10
author: D
header-img:
catalog: true
tags: [timezone,linux_basic]
---

Kali version 2020

```
sudo timedatectl set-timezone "Asia/Shanghai"
systemctl restart ntp.service
```

