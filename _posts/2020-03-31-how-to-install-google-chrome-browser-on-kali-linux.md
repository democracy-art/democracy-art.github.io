--- 
layout: post
title: How to install Google Chrome Browser on Kali Linux
subtitle:
date: 2020-03-31
author: D
header-img:
catalog: true
tags: [Chrome,Kali]
---

# Download Google Chrome
```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```
# Install Google Chrome
The easiest way to install google chrome on your Kali Linux is to by use of <br>
`gdegi` which will automatically download all depended packages.First install `gdegi`:
```
apt install gdebi-core
```
Once ready, install the actual google chrome package:
```
gdebi google-chrome-stable_current_amd64.deb
```
# Start Google Chrome
To start Google Chrome, open up a terminal and run google-chrome command:
```
google-chrome --no-sandbox
```
But it is not safe to run google-chrome without sandbox.

# Run Google Chrome as standard user
```
useradd -m chromeuser
```
To run google chrome use command:
```
apt install gksu
```
```
gksu -u chromeuser google-chrome
```
# Configure Google Chrome to Use a Proxy Server
```
vim ~/.bashrc
```
Add following Content to `~/.bashrc`
```
alias chrome='google-chrome --proxy-server="socks://127.0.0.1:1080" --no-sandbox --user-data-dir'
```
Enable the configuration
```
source ~/.bashrc
```
Run Google Chrome
```
chrome
```
**NOTE:** run Google Chrome in this way that without `--no-sandbox` is **NOT** safe.


