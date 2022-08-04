+++
title = "启用tcp转发"
date = "2020-07-10"
categories = [
"技术",
]
tags = [
"iptable",
]
+++

最近的一些技术总结

### 1 启用TCP 端口转发

服务器需要新增一个ip，记录一下：
    
   
    打开包转发开关
    #sudo sysctl -w net.ipv4.ip_forward=1
    net.ipv4.ip_forward = 1
    
    
    #iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 8080 -j DNAT --to 2.56.184.31:8080
    #iptables -t nat -A POSTROUTING -d 2.56.184.31 -p tcp --dport 8080 -j SNAT --to 106.52.195.213
    
    # iptables -A FORWARD -p tcp -d 2.56.184.31 --dport 8080 -j ACCEPT    
   
   
    
    
