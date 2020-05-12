--- 
layout: post
title: Upload files to FastDFS with Bash 
subtitle:
date: 2020-05-13
author: D
header-img:
catalog: true
tags: [FastDFS]
---

`uri.sh` be call by `json.sh`.

# uri.sh
```
#!/bin/bash

/usr/bin/fdfs_test /etc/fdfs/client.conf upload $1 | grep 'remote_filename'|head -n 1 | awk '{print $2}' | awk -F '[=]' '{print $2}'

```
# json.sh
```
#!/bin/bash

prefix=${1}
i=0
img=${prefix}"_"${i}".jpg"
thumbnail=${prefix}"_"${i}"_thumbnail.jpg"
img_uri=" "
thumbnail_uri=" "

echo [
while [ -f "${img}" ]; do
                img_uri=`./uri.sh ${img}`
                thumbnail_uri=`./uri.sh ${thumbnail}`
                echo {id:${i}, thumbnail:\"${thumbnail_uri}\", img:\"${img_uri}\"},
                let i++
                img="${prefix}_${i}.jpg"
                thumbnail="${prefix}_${i}_thumbnail.jpg"
done
echo ]


```
