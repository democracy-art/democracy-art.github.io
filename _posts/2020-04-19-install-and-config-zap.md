--- 
layout: post
title: Install and Configure ZAP
subtitle:
date: 2020-04-19
author: D
header-img:
catalog: true
tags: [ZAP,SecTools]
---

# Introducing ZAP

ZAP: Zed Attack Proxy is a free, open-source penetration testing tool being maintained under the umbrella of the Open Web Application Security Project(OWASP).

At its core, ZAP is what is know as a "man-in-the-middle proxy". It stands between the tester's browser and the web application so that it can intercept and inspect messages sent between browser and web application, modify the contents if needed, and then forward those packets on to the destination. It can be used as standalone application, and as a daemon process.

![zap as man-in-the-middle proxy](/img/zap/zap-as-man-in-the-middle-proxy.png)

If there is another network proxy already in use, as in many corporate environments, ZAP can be configured to connect to that proxy.

![zap as proxy in another network proxy already in use](/img/zap/zap-as-proxy-in-another-network-proxy-already-in-use.png)

# Install and configure ZAP
### Install ZAP
Download ZAP(zaproxy) from [here](https://www.zaproxy.org/download/).

Note that ZAP requires Java 8+ in order to run.
### Persisting a session
Whe you first start ZAP, you will be asked if you want to persist the ZAP session. By default, ZAP sessions are always recorded to disk in a HSQLDB database with a default name and location. If you do not persist the session, those files are deleted when you exit ZAP.

If you choose to persist a session, the session information will be saved int the local database so you can access it later, and you will be able to  provide custom names and locations for saving the files.

![persist session](/img/zap/persist-session.png)

### Generate Certificates
<kbd>Tools</kbd>-->`Options`-->`Dynamic SSL Certificates`--><kbd>Generate</kbd>--><kbd>Save</kbd>. And Use Firefox import ZAP'S SSL Certificates.

### Local Proxies
<kbd>Tools</kbd>-->`Options`-->`Local Proxies`. You can change the local Proxy's information such as `Address`,`Port` and so on.

### Set ZAP proxy server
If there is another network proxy already in use, as in many corporate environments, ZAP can be configured to connect to that proxy. As following:

![zap as proxy in another network proxy already in use](/img/zap/zap-as-proxy-in-another-network-proxy-already-in-use.png)

ZAP's proxy server must be setted. As following:<br>
<kbd>Tools</kbd>-->`Options`-->`Connection`-->`Use Proxy Chain`
- [x] Use an outgoing proxy server
- Address/Domain Name: `localhost`
- Port(e.g. 8080): `1080`
--> <kbd>OK</kbd>

Reference:[ZAP Quick Start Guide](https://www.zaproxy.org/getting-started/)
