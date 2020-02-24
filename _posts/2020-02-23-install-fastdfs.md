---
layout:     post
title:      FastDFS安装
subtitle:
date:       2020-02-23
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [FastDFS]
---

github:[FastDFS](https://github.com/happyfish100/fastdfs)<br>
参考：[happyfish100/fastdfs/INSTALL](https://github.com/happyfish100/fastdfs/blob/master/INSTALL#L113)

软件环境:
- FastDFS V6.06
- Ubuntu18.04

# 1.安装依赖环境
```
sudo apt install make
sudo apt install gcc
```

# 2.下载安装FastDFS依赖库libfastcommon
```
sudo git clone https://github.com/happyfish100/libfastcommon.git
cd libfastcommon
./make.sh clean && ./make.sh && ./make.sh install
```

# 3.安装FastDFS

## 3.1 下载安装FastDFS
```
sudo apt install git
sudo git clone https://github.com/happyfish100/fastdfs.git
cd fastdfs
./make.sh clean && ./make.sh && ./make.sh install
```

## 3.2 设置配置文件
```
./setup.sh /etc/fdfs
```

## 3.3 编辑或修改 tracker, storage, clien 配置文件

**请确保防火墙允许 tracker 和 storage 的端口.**
tracker:
- port:22122
- http_server_port:8080
storage:
- port:23000:
- http_server_port:8888
### 3.3.1 修改 tracker.conf
```
sudo vi /etc/fdfs/tracker.conf
```
- `bind_addr =` 绑定此主机的一个地址,如果留空表示绑定此主机所有地址.
- `base_path`=`/home/yuqing/fastdfs`修改为自己的文件路径，例如:`base_path`=`/home/xxx/fastdfs`(这里的文件目录如果没有，要自己新建，不然会报错)
- 其他要求请查看文档根据实际情况配置

### 3.3.2 修改 storage.conf
```
sudo vi /etc/fdfs/storage.conf
```

- `base_path`=`/home/yuqing/fastdfs`改为自己的文件路径如:`base_path`=`/home/xxx/fastdfs`(这里的文件目录如果没有,要自己新建,不然会报错)
- `store_path0`=`/home/yuqing/fastdfs`改为自己的文件路径`store_path0=/home/xxx/fastdfs`
- `tracker_server`:`192.168.1.100:22122` 改为自己的IP
- `group_name=group1` (因为现在先只是配一个组,所以这里就为group1,多个组存储时才需要改动)
- 其他要求请查看文档根据实际情况配置
### 3.3.3 修改 client.conf
```
sudo vi /etc/fdfs/client.conf
```
- `base_path`=`/home/yuqing/fastdfs` 这里需要改为自己的文件路径，如：`base_path`=`/home/xxx/fastdfs` (这里的文件目录如果没有，要自己新建，不然会报错)
- `tracker_server`=`192.168.1.100:22122`改为自己的IP
- 其他要求请查看文档根据实际情况配置


## 3.4 运行服务器程序
### 3.4.1 运行 tracker 服务器
```
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
```
### 3.4.2 运行 storage 服务器
```
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```
### 3.4.3 在Linux中，可以启动fdfs_trackerd和fdfs_storaged作为服务:(可选)
这一步就不用做了，因为Ubuntu18.04上不存在`/sbin/service`
```
/sbin/service dfs_trackerd restart
/sbin/service fdfs_storaged restart
```


## 3.5 运行监控程序(可选)
```
/usr/bin/fdfs_monitor /etc/fdfs/client.conf
```

## 3.6 (可选)运行测试程序
比如:
```
/usr/bin/fdfs_test <client_conf_filename> <operation>
/usr/bin/fdfs_test1 <client_conf_filename> <operation>
```
例子:上传一个文件做测试
```
/usr/bin/fdfs_test /etc/fdfs/client.conf upload /usr/include/stdlib.h
```
成功会返回URL如下:
```
This is FastDFS client test program v6.06

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.
Please visit the FastDFS Home Page http://www.fastken.com/ 
for more detail.

[2020-02-24 03:22:16] DEBUG - base_path=/home/yuqing/fastdfs, connect_timeout=5, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
	server 1. group_name=, ip_addr=192.168.1.100, port=23000

group_name=group1, ip_addr=192.168.1.100, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/aIBdvl5Th7iAM1PwAACLyBo1AoQ37510.h
source ip address: 192.168.1.100
file timestamp=2020-02-24 03:22:16
file size=35784
file crc32=439681668
example file url: http://192.168.1.100/group1/M00/00/00/aIBdvl5Th7iAM1PwAACLyBo1AoQ37510.h
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/aIBdvl5Th7iAM1PwAACLyBo1AoQ37510_big.h
source ip address: 192.168.1.100
file timestamp=2020-02-24 03:22:16
file size=35784
file crc32=439681668
example file url: http://192.168.1.100/group1/M00/00/00/aIBdvl5Th7iAM1PwAACLyBo1AoQ37510_big.h
```

tracker 服务器配置文件例子请查看 `conf/tracker.conf` <br>
storage 服务器配置文件例子请查看 `conf/storage.conf` <br>
client 配置文件例子请查看 `conf/client.conf` 

## 3.7 其他
### 3.7.1 服务器共同选项

|item name|type|default| Must|
|-|-|-|-|
|base_path |string||Y|
| disabled              | boolean| false   |  N   |
| bind_addr             | string |         |  N   |
| network_timeout       | int    | 30(s)   |  N   |
| max_connections       | int    | 256     |  N   |
| log_level             | string | info    |  N   |
| run_by_group          | string |         |  N   |
| run_by_user           | string |         |  N   |
| allow_hosts           | string |   *     |  N   |
| sync_log_buff_interval| int    |  10(s)  |  N   |
| thread_stack_size     | string |  1M     |  N   |

**备注:**<br>
`base_path`是子目录的基路径:
`data` 和 `logs` `base_path`必须存在,如果不存在,其子目录将自动创建.
```
$base_path/data: store data files
$base_path/logs: store log files
```

`log_level`是syslog的标准日志级别,不区分大小写.

- emerg: 紧急
- alert
- crit: 危险
- error
- warn: 警告
- notice
- info
- debug


`allow_hosts`可以多个，主机可以是主机名或ip地址:<br>
`*`表示匹配所有的ip地址，可以使用这样的范围:`10.0.1.[1-15,20]`<br>
或者 host[01-08,20-25].domain.com, 例如:
```
allow_hosts=10.0.1.[1-15,20]
allow_hosts=host[01-08,20-25].domain.com
```

### 3.7.2 tracker服务器选项

|item name|type|default| Must|
|-|-|-|-|
| port                  | int    | 22000   |  N   |
| store_lookup          | int    |  0      |  N   |
| store_group           | string |         |  N   |
| store_server          | int    |  0      |  N   |
| store_path            | int    |  0      |  N   |
| download_server       | int    |  0      |  N   |
| reserved_storage_space| string |  1GB    |  N   |

**备注:**
- store_lookup 的值:
	- 0:循环(默认)
	- 1:指定group
	- 2:负载均衡(从V1.1开始支持)
- store_group是存储文件的组的名称
当`store_lookup`设置为`1`时(指定组),必须将`store_group`设置为指定的组名.
- `reserved_storage_space`是为系统或其他应用程序保留的存储空间.
如果stoarge服务器group可用的空间`<=`reserved_storage_space`,
则不再有文件能够上传到该group(版本V1.1及以上).
字节单位如下:
	- G or g:gigabyte(GB)
	- M or m:megabyte(MB)
	- K or k for kilobyte(KB)
	- 无字节单位(B)
	
	
### 3.7.3 storage服务器选项

|item name|type|default| Must|
|-|-|-|-|
| group_name          | string |         |  Y   |
| tracker_server      | string |         |  Y   |
| port                | int    | 23000   |  N   |
| heart_beat_interval | int    |  30(s)  |  N   |
| stat_report_interval| int    | 300(s)  |  N   |
| sync_wait_msec      | int    | 100(ms) |  N   |
| sync_interval       | int    |   0(ms) |  N   |
| sync_start_time     | string |  00:00  |  N   |
| sync_end_time       | string |  23:59  |  N   |
| store_path_count    | int    |   1     |  N   |
| store_path0         | string |base_path|  N   |
| store_path#         | string |         |  N   |
|subdir_count_per_path| int    |   256   |  N   |
|check_file_duplicate | boolean|    0    |  N   |
| key_namespace       | string |         |  N   |
| keep_alive          | boolean|    0    |  N   |
| sync_binlog_buff_interval| int |   60s |  N   |

**备注:**
- `tracker_server`可以多个,且`tracker_server`格式是`host:port`,主机可以是主机名或ip地址.
- `store_path#`, `#`是数字,从`0`开始
- `check_file_duplicate`:设置为`true`时,必须和`FastDHT`服务器一起工作,
更多的细节请查看INSTALL of FastDHT. FastDHT下载地址:
```
http://code.google.com/p/fastdht/downloads/list
```
- key_namespace:当`check_file_duplicate`为真时,FastDHT `key_namespace`不能为空,键命名空间应该尽可能`短`.





