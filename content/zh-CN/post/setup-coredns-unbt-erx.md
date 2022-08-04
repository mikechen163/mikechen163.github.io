+++
title = "UBNT ERX dns配置"
date = "2020-01-18"
categories = [
"技术",
]
tags = [
"erx",
"dns",
]
+++

   主要使用了这个项目
   https://github.com/mikechen163/dohproxy

### 1 下载和编译
    

   备注:go版本大于1.13

    git clone --depth 1 https://github.com/mikechen163/dohproxy.git
    cd mikechen163/dohproxy
    GOOS=linux GOARCH=mipsle go build
    
###  2 copy到erx路由器上
    
    scp cn.txt root@erx:/usr/local/bin/
    scp block.txt root@erx:/usr/local/bin/
    scp dohproxy root@erx:/usr/local/bin/mydns
 
###  3 启用mydns
 
   创建一个/etc/init.d/mydns 文件,前面的内容修改为:
   
     ....
     DESC=mydns
     NAME=mydns
     DAEMON=/usr/local/bin/mydns
     
     DAEMON_OPTS="-port 5353 -block /usr/local/bin/block.txt -chn /usr/local/bin/cn.txt   
     ... 
     
   然后/etc/init.d/mydns restart启用服务.
   
     dig @localhost -p 5353 www.google.com 
   测试是否成功.
   
   修改 /etc/dnsmasq.conf ,也可以通过配置实现
     
     server=127.0.0.1#5353	# statically configured
    
   测试是否成功
   
    /etc/init.d/dnsmasq restart  
    dig @localhost www.google.com 
    
###   4 设置开机脚本
   
   /config/scripts/post-config.d/update_iptables
   增加一行,保证mydns重启之后服务启动
    
    /etc/init.d/mydns restart
   
   
