---
layout:		post
title:		Chapter 7 Attacking Session Management(2)-Weakness in Session Token Handling(2)
subtitle:	
date:		2020-03-12
author:		D
header-img:
catalog:	true
mermaid:	true
tags: [web hacking]:
---

参考: *The Web Application Hacker's Handbook* Chapter 7

无论应用程序如何有效地确保它生成的会话令牌不包含任何有意义的信息,而且不受分析或预测的影响,如果这些令牌在以后没有仔细处理,它的会话机制将会受到广泛的攻击。例如,如果以某些方式向攻击者披露令牌,攻击者可以劫持用户会话,即使预测令牌是不可能的。

应用程序不安全处理令牌可以使其容易受到多种方式攻击。

# 1.在网络上披露标记(Disclosure of Tokens on the Network)
# 2.在日志中披露令牌(Disclosure of Tokens in Logs)
# 3.将令牌映射到会话的脆弱映射(Vulnerable Mapping of Tokens to Sessions)

[Chapter 7 Attacking Session Management(2)-Weaknesses in Token Generation](https://dm116.github.io/2020/03/13/attacking-session_management_2/)<br>
[Chapter 7 Attacking Session Management(3)-Weakness in Session Token Handling(2)](https://dm116.github.io/2020/03/13/attacking-session_management_3_2/)<br>
