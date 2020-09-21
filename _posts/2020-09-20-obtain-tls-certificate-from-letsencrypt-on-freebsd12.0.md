--- 
layout: post
title: 在FreeBSD12.0上安裝acme.sh客戶端獲取TLS證書 
subtitle:
date: 2020-09-20
author: D
header-img:
catalog: true
tags: [freebsd, acme.sh, let's encrypt, tls]
---
安裝前環境準備
```
# freebsd-update fetch install
# pkg update && pkg upgrade -y
```
安裝依賴包
```
pkg install -y unzip wget bash socat git
```

1.安裝 acme.sh
```
# pkg install -y acme.sh
```

2.爲你的域名獲取RSA或者ECDSA證書

注意:證書你只需要選取其中**一種**即可.這裏是爲了完整性所以把兩種都寫下來.

RSA證書
```
# acme.sh --issue --standalone -d example.com --ocsp-must-staple --keylength 2048
```
ECDSA證書
```
# acme.sh --issue --standalone -d example.com --ocsp-must-staple --keylength ec-256
```
注意: 將 `example.com` 換成自己的域名.

3.創建目錄用於存儲證書和密鑰. 這裏創建 `/etc/letsencrypt` 目錄來存儲

存儲RSA證書的目錄
```
# mkdir -p /etc/letsencrypt/example.com
```
存儲ECDSA證書的目錄
```
# mkdir -p /etc/letsencrypt/example.com_ecc
```

4.安裝且複製證書到目錄`/etc/letsencrypt`

RSA
```
# acme.sh --install-cert -d example.com --cert-file /etc/letsencrypt/example.com/cert.pem --key-file /etc/letsencrypt/example.com/private.key --fullchain-file /etc/letsencrypt/example.com/fullchain.pem
```
ECC/ECDSA
```
# acme.sh --install-cert -d example.com --ecc --cert-file /etc/letsencrypt/example.com_ecc/cert.pem --key-file /etc/letsencrypt/example.com_ecc/private.key --fullchain-file /etc/letsencrypt/example.com_ecc/fullchain.pem
```

參考:[How to Enable TLS 1.3 in Nginx on FreeBSD 12](https://www.vultr.com/docs/how-to-enable-tls-13-in-nginx-on-freebsd-12)<br>
參考:[Using acme.sh on FreeBSD 12](https://devopscraft.com/using-acme-sh-on-freebsd-12/)
