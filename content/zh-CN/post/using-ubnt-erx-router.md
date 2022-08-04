+++
title = "使用UBNT ER-X路由器的一些总结"
date = "2017-02-13"
categories = [
"技术",
]
tags = [
"ubnt",
"er-x",
"ss",
]
+++

以前是用wndr4300，加载openwrt，完成科学上网的任务。然而，长期使用下来，存在几个问题：

    1 访问国外网站太慢，由于丢包太多，导致看视频的速率只有几百kbps，根本无法看。
    2 流量大的时候，路由器CPU使用率会升的很高，导致无线网络故障。
    
经过一段时间的搜索，终于发现了路由神器ubnt的ER-X，价格也很便宜，更关键的是，完全运行linux debian，需要什么功能，apt-get出手立即搞定，而且二层交换纯硬件转发，ip转发速率达到130Kbps，对于家用来说完全满足。京东下单后，第二天就到货，实际大小相当于一个巴掌。这么小的硬件，这么强大的功能和性能，真当得起神器二字。

#### 1 ER-X基本设置。

步骤如下：

1 有物理网口的计算机一部，网线一根

2 计算机物理端口配置静态ip地址，192.168.1.X，X可以2到254任意配置。

3 路由器上电，通过eth0端口连接网线到计算机的网络接口


4 在计算机上ping 192.168.1.1，收到应到后，表示网络已经OK。

5 通过浏览器访问192.168.1.1，系统会提示证书不可用。在mac 上，safari点击高级，选择信任证书，出现登录窗口。用户名和密码都是ubnt，进入web管理界面

6 在最右侧的wizards标签上，左边的列表选择WAN+2LAN2,然后在右边的内容处，选择你的上网方式，我是DHCP，最关键是设定lan地址，设定为192.168.8.1（为了避免和上行的电信接入路由器冲突），掩码为255.255.255.0，选择Enable the DHCP server.见下图。
![](https://coding.net/u/mike163/p/mike163/git/raw/coding-pages/images/erx-basic-config.png) 
然后点击最下面的apply按钮，保存配置。 同时重启路由器（关掉电源再插上）

7 修改计算机的物理网口的ip地址方式为dhcp，同时把网线从路由器的eth0端口拔出插入到剩下eth1-4的任何一个端口。

8 在计算机上ping 192.168.8.1地址，等ping命令有了正常应答，就可以通过web浏览器再次访问192.168.8.1，重新进入管理界面。也可以通过ssh ubnt@192.168.8.1 命令，通过命令行访问路由器。

9 把路由器的eth0网口和接入路由器端口通过网线链接。在计算机上ping www.baidu.com，等ping命令有响应后，此时基本配置已经OK。

10 把旧的无线路由器的上行网口，通过网线连接erx路由器的eth1-4任意一个端口。 之后可以通过旧路由器的无线网络，访问互联网了。

<!-- more --> 

#### 2 固件版本升级
  web界面失败，通过cli处理。核心要点在于先把固件文件拷贝到/tmp目录下。
  
  参考：
  
  [EdgeRouter - Upgrading EdgeOS firmware](https://help.ubnt.com/hc/en-us/articles/205146110-EdgeRouter-Upgrading-EdgeOS-firmware)
  
  [Solved: EdgeMax Firmware Upgrade - Not enough disk space for root file system](https://community.ubnt.com/t5/EdgeMAX/EdgeMax-Firmware-Upgrade-Not-enough-disk-space-for-root-file/td-p/1533512)
  
#### 3 常用命令
   show dhcp leases 检查IP地址分配情况
   
   设置debian源
   
    configure
    set system package repository wheezy components 'main contrib non-free'
    set system package repository wheezy distribution wheezy
    set system package repository wheezy url http://http.us.debian.org/debian
    
    commit
    save
    exit
   
#### 4 SSR 配置参考
   主要参考链接：[ER-X $$集成包](http://bbs.ubnt.com.cn/forum.php?mod=viewthread&tid=16879&page=1) 。但集成包的一个问题就是把所有未识别的ip流量都走了ss，建议修改为只把墙掉的流量改成走ss即可。
 
 1  /config/scripts/post-config.d/update_iptables 脚本修改如下：
 
    #！/bin/bash
    cp -f /home/ubnt/dnsmasq/dnsmasq.conf /etc/dnsmasq.conf
    /etc/init.d/dnsmasq restart
    
    ipset -N gfwlist iphash
    iptables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1080
    iptables -t nat -A OUTPUT -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1080
    
    alias l='ls -l'
    
  2 /home/ubnt/dnsmasq/dnsmasq.conf 增加下面的内容：
  
    server=127.0.0.1#5353	    # 使用chinadns作为dns服务
    server=/trailers.apple.com/180.153.225.136 # 苹果atv3使用
    conf-dir=/etc/dnsmasq.d  #增加包含目录
  
  3 修改 /etc/dnsmasq.conf  ，在最后加入  conf-dir=/etc/dnsmasq.d  ，新建并进入目录  /etc/dnsmasq.d  ，下载[dnsmasq_list.conf](http://pan.baidu.com/s/1qWDVbfY)后放入该目录
   
 附：[自动生成dnsmasq_list.conf 的脚本](https://github.com/cokebar/gfwlist2dnsmasq)
   


