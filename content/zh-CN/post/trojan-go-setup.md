+++
title = "在centos 7配置trojan go"
date = "2022-09-12"
categories = [
"技术",
]
tags = [
"trojan-go",
"tls",
"certbot",
"nginx",
]
+++

### 1 前置条件

  需要一个域名
  需要一个 vps，安装好centos 7 或者 8

### 1 安装nginx
    
    
    sudo yum install -y epel-release
    sudo yum update -y 
    
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
    
    sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
    sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
    
    #保存防火墙配置文件
    sudo service iptables save
    
    
   申请证书 ,修改email地址 和 域名 参数
    
    #申请证书前，确保nginx工作正常
    sudo systemctl start nginx
    sudo systemctl status nginx
    
    # 修改下面命令中的， email地址 和 域名 
    sudo certbot certonly --webroot --email your_email@xxx -w /usr/share/nginx/html -d xxx.your.site
    
   如果看到
   
    Congratulations! Your certificate and chain have been saved at:
     /etc/letsencrypt/live/xxx.site/fullchain.pem
    Your key file has been saved at:
    /etc/letsencrypt/live/xxx.site/privkey.pem
    说明申请成功了
    
 <!-- more -->    
 
 设定certbot自动更新证书
    
    #执行下面的命令
    SLEEPTIME=$(awk 'BEGIN{srand(); print int(rand()*(3600+1))}'); echo "0 0,12 * * * root sleep $SLEEPTIME && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
    
    #检查/etc/crontab
    sudo cat /etc/crontab
    
    #如果有下面这句，说明自动更新证明任务已经注册
    0 0,12 * * * root sleep 2380 && certbot renew -q
 
    #证书更新完成后，重启nginx和trojan-go
    cd /etc/letsencrypt/renewal-hooks/post
    sudo touch renew_post.sh
    sudo chmod +x renew_post.sh
    
    #在renew_post.sh文件中，增加
    #!/bin/bash
    nginx -s reload

    #!/bin/bash
    systemctl restart trojan-go
    
    #测试一下证书更新流程。 如果后面trojian-go失败，不要担心，因为还没有安装，等trojan更新成功后，再执行一遍就行了。
    sudo certbot renew --dry-run
 
    
   修改/etc/nginx/conf.d/defaul.conf文件 ,注意把所有xxx.site修改为你自己的域名
   
   注意端口是4443，因为443留给了trojan-go，这里的4443是trojan-go回落后的端口
   
   
    server {
      # SSL configuration
      listen 4443 ssl http2 default_server;
      listen [::]:4443 ssl http2 default_server;
       ssl_certificate    /etc/letsencrypt/live/Yoursite.com/fullchain.pem;
       ssl_certificate_key  /etc/letsencrypt/live/Yoursite.com/privkey.pem;

     ssl_protocols TLSv1.2 TLSv1.3;
 
     root /usr/share/nginx/html;

     # Add index.php to the list if you are using PHP
     index index.html index.htm index.nginx-debian.html;

     server_name Yoursite.com;
}
    
   重启nginx
   
    sudo systemctl restart nginx
    
   如果成功,这时候可以访问https://xxx.site/ 
   
### 3 在centos 7上配置trojan-go

   下载trojan-go，并解压

    cd 
    mkdir trojan-go
    cd trojan-go
    wget https://github.com/p4gefau1t/trojan-go/releases/download/v0.10.6/trojan-go-linux-amd64.zip
    unzip trojan-go-linux-amd64.zip
    
    sudo mkdir /etc/trojan-go
    sudo cp trojan-go /etc/trojan-go
    sudo cp *.dat /etc/trojan-go
   
   服务器配置文件 /etc/trojan-go/server.json，注意修改yousite.com为你自己的站点域名

    {
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "修改为你的密码"
    ],
    "ssl": {
        "cert": "/etc/letsencrypt/live/Yousite.com/fullchain.pem",
        "key": "/etc/letsencrypt/live/Yousite.com/privkey.pem",
        "sni": "Yousite.com",
        "fallback":"4443"  #注意，这里的4443和前面nginx的端口对应。
        
    },
    "websocket": {
    "enabled": true,
    "path": "/ws_path",
    "hostname": "Yousite.com"
    },
    "router":{
        "enabled": true,
        "block": [
            "geoip:private"
        ]
    }
    }
    
  建立trojan-go服务
     
     sudo  vi /etc/systemd/system/trojan-go.service 
     
     #文件内容如下
     [Unit]
     Description=Trojan-Go - An unidentifiable    mechanism that helps you bypass GFW
     Documentation=https://p4gefau1t.github.io/trojan-go/
     After=network.target nss-lookup.target

     [Service]
     Type=simple
     StandardError=journal
     PIDFile=/usr/src/trojan/trojan/trojan.pid
     ExecStart=/etc/trojan-go/trojan-go -config /etc/trojan-go/server.json
     ExecReload=
     ExecStop=/etc/trojan-go/trojan-go
     Restart=on-failure
     RestartSec=10s
     LimitNOFILE=infinity

     [Install]
     WantedBy=multi-user.target
     
     
     sudo systemctl daemon-reload
     sudo systemctl start trojan-go
     
     #检查trojan-go正常运行
     sudo systemctl status trojan-go
     
     #设置为启动后自动执行
     sudo systemctl enable trojan-go
     sudo systemctl enable nginx
    
      #此时，再执行一下证书更新测试，应该一切正常了
      sudo certbot renew --dry-run
    
### 4 客户端配置trojan-go
    
    { 
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1082,
    "remote_addr": "yousite.com",
    "remote_port": 443,
    "password": [
        "你的密码"
    ],
    "ssl": {
        "sni": "yousite.com"
    },
    "mux" :{
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
    #如果想用cdn加速，把下面这行改成true
    "enabled": false, 
    "path": "/ws_path",
    "hostname": "yousite.com"
    },
    "router":{
        "enabled": true,
        "bypass": [
            "geoip:cn",
            "geoip:private",
            "geosite:cn",
            "geosite:geolocation-cn"
        ],
        "block": [
            "geosite:category-ads"
        ],
        "proxy": [
            "geosite:geolocation-!cn"
        ],
        "default_policy": "proxy"
    }
    }
       