+++
title = "配置v2ray+ws+tls1.3+nginx"
date = "2020-01-18"
categories = [
"技术",
]
tags = [
"v2ray",
"tls",
"ws",
"nginx",
]
+++

### 1 安装nginx
    
   
    sudo yum install nginx
    sudo systemctl start nginx
    sudo systemctl status nginx
    
检查nginx 版本,重点是检查 openssl是否支持1.1.1,只有1.1.1之后的版本,才能支持tls1.3
      
      nginx -V
      built by gcc 8.2.1 20180905 (Red Hat 8.2.1-3) (GCC)
      built with OpenSSL 1.1.1 FIPS  11 Sep 2018 (running with OpenSSL 1.1.1c FIPS  28 May 2019)
      TLS SNI support enabled
      
### 2 申请证书

    sudo yum install epel-release
    sudo yum install certbot

   打开 80 443 端口   
    
    sudo iptables -I INPUT 4  -p tcp -m tcp --dport 80 -j ACCEPT
    sudo iptables -I INPUT 4  -p tcp -m tcp --dport 443 -j ACCEPT
 
   申请证书 ,修改email地址 和 域名 参数
    
    sudo certbot certonly --webroot --email your_email@xxx -w /usr/share/nginx/html -d xxx.your.site
    
   如果看到
   
    Congratulations! Your certificate and chain have been saved at:
     /etc/letsencrypt/live/xxx.site/fullchain.pem
    Your key file has been saved at:
    /etc/letsencrypt/live/xxx.site/privkey.pem
    说明申请成功了
    
 <!-- more -->    
    
   修改/etc/nginx/conf.d/defaul.conf文件 ,注意把所有xxx.site修改为你自己的域名
   
   
    server {
    # SSL configuration
    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;
    ssl_certificate    /etc/letsencrypt/live/xxx.site/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/xxx.site/privkey.pem;

    #ssl_protocols TLSv1.2 TLSv1.3;
    ssl_protocols  TLSv1.3;

    root /usr/share/nginx/html;
    #root /var/www;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html;

    server_name xxx.site; 


    location /vray { #
        proxy_redirect off;
        proxy_pass http://127.0.0.1:9000; 
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
     }
    }
    
   重启nginx
   
    sudo systemctl restart nginx
    
   如果成功,这时候可以访问https://xxx.site/ 
   
### 3 配置v2ray

   服务器侧关键数据

    "inbound": {
    "port": 9000, //(此端口与nginx配置相关)
    "listen": "127.0.0.1",
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
        
            "id": "xxxxx-xxxx-xxxx",
           "level": 1,
          "alterId": 64 //此ID也需与客户端保持一致
        }
      ]
    },
    "streamSettings":{
      "network": "ws",
      "wsSettings": {
           "path": "/vray" //与nginx配置相关
      }
    }
    },



    "outbound": {
    "protocol": "freedom",
    "settings": {}
    },
    "outboundDetour": [
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
    ],
    "routing": {
    "strategy": "rules",
    "settings": {
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
          "outboundTag": "blocked"
        }
      ]
    }
    }
    
    
   客户段配置
   
     "outbounds": [
    {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
       
          "address": "xxx.xxx.xxx.xxx",
        "port": 443,  // 服务器端口

          "users": [
            {
                       "id": "xxx-xxxx-xxxx-xxxx",
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
            "serverName": "xxx.site"
        },
        "wsSettings": {
            "path": "/vray"
        }
    },
    "tag": "forgin"
    },

    {
    // Protocol name of the outbound proxy.
    "protocol": "freedom",

    // Settings of the protocol. Varies based on protocol.
    "settings": {},

    // Tag of the outbound. May be used for routing.
    "tag": "direct"
    },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
    }],

    // Transport is for global transport settings. If you have multiple transports with same settings
    // (say mKCP), you may put it here, instead of in each individual inbound/outbounds.
    //"transport": {},

    // Routing controls how traffic from inbounds are sent to outbounds.
    "routing": {
    "domainStrategy": "IPOnDemand",
    "rules":[
      {
        // Blocks access to private IPs. Remove this if you want to access your router.
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      },
      {
        // Blocks major ads.
        "type": "field",
        "domain": ["geosite:category-ads"],
        "outboundTag": "blocked"
      }
    ]
    },
