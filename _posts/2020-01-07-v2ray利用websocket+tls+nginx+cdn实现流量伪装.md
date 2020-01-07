
[TOC]

# 一.购买vps,申请域名,使用Cloudfalre的CDN

1.购买vps可以去 [搬瓦工](https://bwh88.net/)
1.1 设置vps时间(系统 Centos 7 x86_64 bbr)

1.1.1 创建clock文件
```
vi /etc/sysconfig/clock
```

1.1.2 写入clock文件
```
ZONE="Asia/Shanghai"
UTC=false
ARC=false
```

1.1.3 安装ntp
```
yum -y install ntp
```

1.1.4 建立软链接
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai    /etc/localtime
```

1.1.5 使用 ntpdate 测试 NTP
```
ntpdate asia.pool.ntp.org
```

2.申请域名可以去freenom网站

2.1 注册`freenom`账号

2.2 注册域名

>登录freenom主页-->Services标签-->Register a New Domain-->
在输入框(显示Find your new Domain的输入框)中输入域名(自己喜欢的随意几个字符)
-->点击Check Availability(查看域名是否可用)-->如果域名可用它会出现 .tk, .ml, .ga, .cf, .gq
结尾的5个免费的域名在最上面5行,选一个喜欢的点击 Get it now 
-->它会跳转到另一个页面,在 Period 下面,点击下拉框 选择 12 Months@FREE(12个月免费)
-->点击 Continue(继续)-->Review & Checkout页面 输入一个有效的邮箱用来接收信息
-->Your Details 页面,填写信息随便填就好,只要能通过,只要上一步的邮箱是有效的就行,填完同意协议,点击 Complete Order
-->完成后注意查收邮件,它会通知你的域名是否生效,或者登录freenom来到主页面,点击 Services标签-->My Domains
-->看到刚注册的域名 Status是 ACTIVE 说明注册成功

3.将解析域名的DNS服务器设置为Cloudflare的DNS,(需要注册Cloudflare账号)

3.1 将你申请的vps ip地址的DNS服务器设置为Cloudflare的DNS

3.1.1 注册Cloudflare的账号

3.1.2 获取cloudflare的DNS服务器
登录 Cloudflare主页 点击 `+Add site` 标签-->输入你在freenom注册的域名-->来到另一个页面
显示`We're querying your DNS records` 点击 `Next`
-->来到另一个页面 Select a Plan 选择 FREE $0/month 点击 Confire Plan
--> `Add more DNS records for xxx.kt` (这里假设你申请的域名为 `xxx.tk` ) 假设你的域名是 `xxx.tk` 下面都以它为例子
添加记录如下:
`A `  `www`   `xxx.tk ` `AutomaticTTL` 点击 `Add Record`
`A `  `@`   `xxx.tk ` `AutomaticTTL` 点击 `Add Record`

--> 点击刚添加的两条记录的橘黄小云朵的,点击之后它会变为灰色
--> 点击"Continue"继续
--> 然后会来的一个页面显示你当前域名使用的DNS服务器和Cloudflare要你使用的它的DNS服务器
复制且保存 Replace with Cloudflare's  nameservers:下面的两条记录
Nameserver 1 和 Nameserver 2
一般形式是:
```
xxx.ns.cloudflare.com
yyy.ns.cloudflare.com
```
复制保存好后 点击 `Done,check nameservers`

3.1.3 登录freenom,将原来freenom提供的DNS服务器替换为Cloudflare的DNS服务器
登录freenom主页-->Services标签-->My Domains-->Manage Domain-->Management Tools-->Nameservers
-->Use custom nameservers (enter below)粘贴你保存的两个cloudflare的DNS地址即,
Nameserver 1:
`xxx.ns.cloudflare.com`
Nameserver 2:
`yyy.ns.cloudflare.com`
然后点击 `Change Nameservers`
-->这样之后它会在24小时之内帮你跟新好DNS服务器,一般用不到24小时,大概`10分钟左右`可以了
之后你登录 Cloudflare 就可以看到 DNS 更新了为 `xxx.ns.cloudflare.com` 和 `yyy.ns.cloudflare.com`
同时它也会发送邮件跟你说.

3.1.4 登录cloudflare查看DNS是否更新成功
登录cloudflare主页 你会看到你的域名 `xxx.tk` 下面的 `ACTIVE 打勾` 说明成功了

3.1.5 帮域名设置 SSL/TLS
点击域名 `xxx.tk` --> `SSL/TLS` --> `Flexible` (当下面步骤中 v2ray和nginx设置好并且申请了证书才能回来这里,把Flexible设置为 `Full(strict)` )

3.1.6 设置缓存加速
Cloudflare主页,标签 Caching --> Browser Cache TTL --> 1 year(选择缓存最长时间1年)


# 二. nginx 和 v2ray 安装设置
1.安装nginx源
```
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

2.安装 nginx
```
yum install -y nginx
```

2.1 停止nginx
systemctl stop nginx

2.2 安装伪站点
2.2.1 创建伪站点目录
```
rm -rf /home/wwwroot && mkdir -p /home/wwwroot && cd /home/wwwroot
```

2.2.2.1 安装git
```
yum install git
```
出现提问输入 y 
```
Is this ok [y/d/N]: y
```

2.2.2.2 获取伪站点程序
```
git clone https://github.com/dunizb/sCalc.git
```

3.安装官方v2ray
3.1 回到 /root 目录
```
cd /root
```

3.2 安装 v2ray
```
bash <(curl -L -s https://install.direct/go.sh)
```

4.记录端口和uuid

```
PORT:10086
UUID:0e658508-bbf1-4655-995f-2c00543ce3d4
```
**注意:** 端口和uuid要根据自己的来,我这里乱写的端口和UUID

4.获取和安装证书( `acme.sh` 实现了 acme 协议, 可以从 `letsencrypt` 生成免费的证书.)
4.1 获取
```
curl  https://get.acme.sh | sh
```

4.2 使生效
```
source ~/.bashrc
```

4.3 找Key和邮箱

4.3.1 找Key
如何找Key,登录Cloudflare主页,右上角有几个标签如 Add site, Support, English ,头像,
头像-->选择MyProfile-->API Tokens-->会显示有API Keys 点击Global API Key 这一行右边的 View 按钮.
-->然后它会让你输入你账号的Cloudflare的密码和验证码-->之后就会看到 Key复制出来就好
```
export CF_Key="vyvkcs6k9u0g7qw8fpksruxnttassdx4dsli8"
```

4.3.2 邮箱
邮箱就是你注册Cloudflare时候用的邮箱
```
export CF_Email="xxxxxxxxxxx@qq.com"
```

4.4 
```
~/.acme.sh/acme.sh --issue --dns dns_cf -d xxx.tk -d *.xxx.tk -k ec-256
```

4.5 
```
~/.acme.sh/acme.sh --installcert -d xxx.tk -d *.xxx.tk --fullchainpath /etc/v2ray/xxx.tk.crt --keypath /etc/v2ray/xxx.tk.key --ecc
```

4.6 定期自动更新
```
~/.acme.sh/acme.sh --upgrade --auto-upgrade
```

5. v2ray 配置文件设置
配置文件绝对路径:
```
/etc/v2ray/config.json
```
设置如下:
```
{
  "inbounds": [
    {
      "port": 10086,   //端口保持跟nginx的default.conf里面端口一致
      "listen": "127.0.0.1",
      "tag": "vmess-in",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "0e658508-bbf1-4655-995f-2c00543ce3d4",  
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws", 
        "wsSettings": {
          "path": "/yf321"   //保持跟nginx的default.conf里面location一致
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "inboundTag": [
          "vmess-in"
        ],
        "outboundTag": "direct"
      }
    ]
  }
}
```
6. nginx 配置文件设置
配置文件绝对路径:
```
/etc/nginx/conf.d/default.conf
```
设置如下:
```
server {
	listen 443 ssl;
	ssl on;
	ssl_certificate       /etc/v2ray/xxx.tk.crt;
	ssl_certificate_key   /etc/v2ray/xxx.tk.key;
	ssl_protocols         TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; 
	ssl_ciphers           HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers on;
	ssl_session_cache shared:SSL:10m;
	ssl_session_timeout 10m;
	server_name www.xxx.tk;
	index index.html index.htm;
	root  /home/wwwroot/sCalc;
	error_page 400 = /400.html ;
	location /yf321  #保持跟v2ray的config.json里面path一致
	{ 
		proxy_redirect off;
		proxy_pass http://127.0.0.1:10086; #保持跟v2ray的config.json里面port一致
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
		proxy_set_header Host $http_host;
	}
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
# 配置 80 重定向 443 强制 SSL
server {
	listen 80;
	server_name xxx.tk;
	return 301 https://xxx.tk$request_uri;
}
```


7.TLS开启OSCP
```
openssl s_client -connect xxx.tk:443 -status -tlsextdebug < /dev/null 2>&1 | grep -i "OCSP response"
```

8.开启TCP fastopen
```
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
```

9. 重启nginx
```
systemctl restart nginx
```

10. 重启v2ray
```
systemctl restart v2ray
```

11.设置 SSL/TLS和CDN <br/>
11.1 登录Cloudflare主页-->点击域名 xxx.tk --> DNS --> 点击两条记录的灰色小云朵,从灰色变为橘黄色<br/>
11.2 登录Cloudflare主页-->点击域名 xxx.tk --> SSL/TLS --> Full(strict)<br/>

# 三.下载安装 v2ray客户端
### 安装
下载 v2ray windows的客户端可以下载 [v2ray-core](https://github.com/v2ray/v2ray-core/releases) ,
但是,更建议下载 [v2rayN](https://github.com/2dust/v2rayN/releases) 的 v2rayN-Core.zip,
因为`v2rayN-Core.zip` 里面即包含了 v2rayN 界面 还包含了v2ray-core,
即v2rayN只是一个UI界面,需要搭配v2ray-core, 它们搭配起来就组成了v2rayN-Core.zip. 
安装v2rayN-Core需要安装依赖环境 [.NET Framework 4.6](https://docs.microsoft.com/zh-cn/dotnet/framework/install/guide-for-developers) 或更高版本.
	
### 配置 v2rayN-Core windows客户端
解压v2rayN-Core.zip-->进入v2rayN-Core文件夹-->双击打开v2rayN.exe-->服务器-->添加[VMess]服务器:
- 地址(address):www.xxx.tk                       //这里写你申请的域名
- 端口(port):443
- 用户(ID):0e658508-bbf1-4655-995f-2c00543ce3d4  //要与服务器保持一致
- 额外ID(alterId):64                             //不能大于服务器上面的alterId的值
- 加密方式(security):auto                        
- 传输协议(network):ws
- 别名(remarks): 随便填都行
- 伪装类型(type):none                            //保持默认
- 伪装域名(host):不需要填
- 路径(path):/yf321                              //与服务器保持一致
- 底层传输安全:tls
- allowInsecure:false

# 四 其他配置
### 1. Cloudflare 加速配置
登录Cloudflare主页 --> 点击要加速的域名假设为 xxx.tk -->Firewall-->Firewall Rules
-->Create a Firewall rule --> Rule name(填写相应名称比如 allow_URI)
- Field:`URI Path`
- Operator:`equals`
- Value:`/yf321`      (要和客户端的path一致)<br/>
-->Then...(Choose an action):`Allow`
