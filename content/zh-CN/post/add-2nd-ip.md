+++
title = "Centos 7 增加第二个IP"
date = "2020-07-01"
categories = [
"技术",
]
tags = [
"iptable",
"cloudflare",
"v2ray",
"dns",
]
+++

最近的一些技术总结

### 1 Centos 7 增加第二个ip

服务器需要新增一个ip，记录一下：

检查服务器是否启用了NetworkManager服务
    
   
    #ls /etc/sysconfig/network-scripts/ifcfg*
    #grep 'NM_CONTROLLED' /etc/sysconfig/network-scripts/ifcfg-eth0
    没找到
    
    #sudo systemctl -a | grep NetworkManager
    没启用
    not-found inactive dead      NetworkManager.service
    
    #sudo systemctl -a | grep network
    还是旧的方式
    network.service                                                                                                   loaded    active   exited    LSB: Bring up/down networking
    
    
    
直接增加一个配置文件：
      
     
      #sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0:0
      
      BOOTPROTO=none
      DEVICE="eth0:0"
      IPADDR=xx.xx.xx.xx #第二个ip地址 
      NETMASK=255.255.255.0
      ONBOOT=yes
      
 启用命令：
  
      #sudo ifdown eth0; ifup eth0
      
      #ifconfig 命令检查新的ip
      eth0:0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet xx.xx.xx.xx  netmask 255.255.255.0  broadcast xx.xx.xx.255
     
 另外一套体系是ip 
 
       
      #ip addr show 检查ip地址
      #sudo ip addr add xx.xx.xx.xx/24 broadcast xx.xx.xx.255 dev eth0  添加一个新的ip地址，关机就没有了。
      
      #ip route show 检查路由
      # sudo ip route add 8.8.4.4/32 via xx.xx.xx.xx 添加一个新的路由， 
      
      #ip route get 8.8.8.8 检查到8.8.8.8 的路由
      
      
### 2 在路由器上，转发udp端口

    #转发 8.8.8.8的 dns 查询请求到192.168.1.1
    iptables -t nat -A PREROUTING -p udp -d 8.8.8.8 --dport 53 -j DNAT --to 192.168.1.1:53
    

      
### 3 v2ray增加直达的域名

      
    
   客户段配置
   
    "routing": {
    "domainStrategy": "IPOnDemand",
    "rules":[
      {
        // Blocks access to private IPs. Remove this if you want to access your router.
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "ip": ["geoip:cn"],
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "domain": ["geosite:cn",
                    "github.com",
                    "apple.com"
                  ],
        "outboundTag": "direct"
      },
     
### 4 启用cloudflare

  注册用户，属于你的域名，cloudflare会自动识别现有的地址绑定。
  
  然后在域名登记的网站，修改域名的解析服务器为 cloudflare的域名服务器。
  
  设定 ssl的加密方式为 full(strict)
  
  增加一个防火墙规则，识别你的流量，allow。 没有这个，cloudflare可能会限制流量。

