--- 
layout: post
title: bash shell編程產數組0000-9999並導入文件code 
subtitle:
date: 2020-09-23
author: D
header-img:
catalog: true
tags: [bash, shell編程, 0000-9999]
---
bash文件內容如下
```
#!/usr/bin/bash  #每個系統的bash路徑可能不一樣,可以用vim底行命令模式 :.!which bash 找到
for i in {0..9999}
do
    if [ $i -lt 10 ]; then
            echo "000$i"
    elif [ $i -lt 100 ]; then
            echo "00$i"
    elif [ $i -lt 1000 ]; then
            echo "0$i"
    else
            echo "$i"
    fi
done
```
假設文件名稱爲generate_code.sh .還需要给文件執行權限:
```
chmod +x generate_code.sh
```
運行
```
./generate_code.sh
```
如果需要將結果導入到文件，假設要導入的文件爲code,那麼命令如下:
```
./generate_code.sh > code
```
