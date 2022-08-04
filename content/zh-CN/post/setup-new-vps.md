+++
title = "UBNT ERX v2ray 配置透明代理"
date = "2020-01-18"
categories = [
"技术",
]
tags = [
"ubnt",
"er-x",
"ss",
]
+++

### 1 安装v2ray
  自动安装脚本应该是不行的。因此需要手工操作。还好都已经编译好了。路径
  [Releases · v2ray/v2ray-core](https://github.com/v2ray/v2ray-core/releases)
 
  选择v2ry-linux-mipsle.zip 解压后，手工拷贝到路由器的相关目录。假设erx是路由器的hostname
  
  
    scp v2ray root@erx:/usr/bin/v2ray/v2ray
    scp v2ctl root@erx:/usr/bin/v2ray/v2ctl
    scp geoip.dat root@erx:/usr/bin/v2ray/geoip.dat
    scp geosite.dat root@erx:/usr/bin/v2ray/geosite.dat
    
    scp config.json root@erx:/etc/v2ray/config.json
   修改配置文件，参考本帖。本文的配置是 v2ray+ws+tls+nginx的客户端配置
    
   在路由器/etc/init.d目录下，建立v2ray文件,修改内容
  
    ...
    DESC=v2ray
    NAME=v2ray
    DAEMON=/usr/bin/v2ray/v2ray
    PIDFILE=/var/run/$NAME.pid
    SCRIPTNAME=/etc/init.d/$NAME

    DAEMON_OPTS="-config /etc/v2ray/config.json"
    ...
    
   启动v2ray服务
     
     /etc/init.d/v2ray start
   记得把启动命令写入 /config/scripts/post-config.d/update_iptables

   用下面的命令测试一下是否成功
   
    curl -vx socks5://127.0.0.1:1080 https://www.google.com
    
   用netstat -tupn 命令检查是否成功建立本地1080端口的连接 
   
   成功的话,可以看到ssl tls的握手过程和google返回的内容
   
   <!-- more --> 
   
### 2 客户端修改

  config.json 修改为
  
    "inbounds": [
    {
    "port": 1080,
    "listen": "0.0.0.0",
    "protocol": "socks",
    "settings": {
      "auth": "noauth",
      "udp": false, // dns使用了doh tls 因此不转发 udp 53端口
      "ip": "127.0.0.1"
     }
    },
    {
        "port": 12345, //开放的端口号
        "protocol": "dokodemo-door",
        "settings": {
           "network": "tcp,udp",
           "followRedirect": true // 这里要为 true 才能接受来自 iptables 的流量
         },
        "sniffing": { //从ip包里面得到url
           "enabled": true,
           "destOverride": ["http", "tls"]
        },
       "streamSettings": {  
            "sockopt": {
               "tproxy": "redirect" 
            }
        }
    }
    ],
    
     "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
           {
          // 你的服务器ip
          "address": "xxx.xxx.xxx.xxx",
          "port": 443,  // 服务器端口
          "users": [
            {
              //你得id
              "id": "xxxx",
              "alterId": 64,
              "security": "aes-128-gcm"
            }
          ]
        }
      ]
    },
    
    "streamSettings":{
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
            "serverName": "www.example.com"  //修改你的tls 域名"
        },
        "wsSettings": {
            "path": "/v2ray“  //修改你的路径
        },
        
        
        "sockopt": {
           "mark": 255  //这里是 SO_MARK，用于 iptables 识别，每个 outbound 都要配置；255可以改成其他数值，但要与下面的 iptables 规则对应；如果有多个 outbound，最好奖所有 outbound 的 SO_MARK 都设置成一样的数值
        }
    },
    "tag": "forgin"
    },
  
    "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {},
      
       //这里一定要修改，否则国内流量不能出去
       "streamSettings": {
          "sockopt": {
              "mark": 255
           }
       },
      "tag": "direct"
    }
    ],
    "dns": {
    "servers": [
      "localhost" //使用系统的dns
    ]
    },
    "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "IPIfNonMatch",
      "rules": [
        {
          "type": "field",
          "ip": [
            "0.0.0.0/8",
            "10.0.0.0/8",
            "100.64.0.0/10",
            "127.0.0.0/8",
            "169.254.0.0/16",
            "172.16.0.0/12",
            "192.0.0.0/24",
            "192.0.2.0/24",
            "192.168.0.0/16",
            "198.18.0.0/15",
            "198.51.100.0/24",
            "203.0.113.0/24",
            "::1/128",
            "fc00::/7",
            "fe80::/10"
          ],
          "outboundTag": "direct"
        },
        {
          "type": "chinasites",
          "outboundTag": "direct"
        },
        {
          "type": "chinaip",
          "outboundTag": "direct"
        }
      ]
    }
    }
    
### 3 修改iptables

 命令：
  
    iptables -t nat -N V2RAY #  V2RAY config.json
    iptables -t nat -A V2RAY -d 192.168.0.0/16 -j RETURN # config.json 192.168.0.0/16
    iptables -t nat -A V2RAY -p tcp -j RETURN -m mark --mark 0xff

  
    #你自己的服务器地址，修改xxx.xxx.xxx.xxx为你得服务器地址
    iptables -t nat -A V2RAY -d xxx.xxx.xxx.xxx -j RETURN 
        
    #由于dns使用了doh tls，因此允许tls 853端口通过
    iptables -t nat -A V2RAY -p tcp -m tcp --dport 853  -j RETURN

    iptables -t nat -A V2RAY -d 0.0.0.0/8 -j RETURN
    iptables -t nat -A V2RAY -d 10.0.0.0/8 -j RETURN
    iptables -t nat -A V2RAY -d 127.0.0.0/8 -j RETURN
    iptables -t nat -A V2RAY -d 169.254.0.0/16 -j RETURN
    iptables -t nat -A V2RAY -d 172.16.0.0/12 -j RETURN
    iptables -t nat -A V2RAY -d 192.168.0.0/16 -j RETURN
    iptables -t nat -A V2RAY -d 224.0.0.0/4 -j RETURN
    iptables -t nat -A V2RAY -d 240.0.0.0/4 -j RETURN

    #把所有tcp流量都发给12345端口
    iptables -t nat -A V2RAY -p tcp -j REDIRECT --to-ports 12345
    
    #激活命令
    iptables -t nat -A PREROUTING  -p tcp -j V2RAY
    iptables -t nat -A OUTPUT -p tcp -j V2RAY
    
    
 
 
### 4 保存配置   
 把上面的命令写到 /config/scripts/post-config.d/update_iptables，这样重启路由器之后，可以自动更新。
 
 需要说明的是，如果以前配置了gfwlist，需要注释掉原来的配置 /config/scripts/post-config.d/update_iptables这几句
 
    注释掉这几句。
    #ipset -N gfwlist iphash
    #iptables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1080
    #iptables -t nat -A OUTPUT -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1080
    
   如果原来有gfwlist配置，用下面命令删除
   
    iptables -t nat -D PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1080
    iptables -t nat -D OUTPUT -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1080
    
   记得随时用下面的命令查看数据
      
    iptables -t nat -L -nvx 
    
    
### 参考文章
 
  [V2RAY透明代理 | xdays](https://xdays.me/V2RAY%E9%80%8F%E6%98%8E%E4%BB%A3%E7%90%86/)
  
  [Dokodemo · Project V Official](https://www.v2ray.com/en/configuration/protocols/dokodemo.html)
  
  [透明代理(REDIRECT) · V2Ray 配置指南|V2Ray 白话文教程](https://toutyrater.github.io/app/transparent_proxy.html)
    

   
 
   
    
   
