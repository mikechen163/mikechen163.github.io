+++
title = "Centos 7 VPS服务器配置说明"
date = "2015-08-01"
categories = [
"技术",
]
tags = [
"vps",
]
+++
vps 配置主要包括 安全、科学上网、应用相关。相关命令以centos 7为基准
 
###  1 安全配置

  参考文档
  
  [Additional Recommended Steps for New CentOS 7 Servers](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-centos-7-servers)
 
 [Securing Your Server](https://www.linode.com/docs/security/securing-your-server#centosfedora)
  
 [Linux Security Basics](https://www.linode.com/docs/security/linux-security-basics)


####   1.1 添加常用用户
   新增一个testuser,添加到wheel组，平时就使用这个用户操作。
   
   	useradd -g wheel -d /home/testuser testuser
   	passwd testuser
   
   在.bashrc文件后，添加ls的别名
   					
   	alias l='ls -l'
   	
####    	1.2 配置ssh
   
   配置ssh的访问方式为 key访问，关闭密码方式。
   
#####   1 生成public key，并添加到到远程主机~/.ssh/authorized_keys文件
   
   	ssh-keygen -t rsa
   	
拷贝到远程主机

    ssh-copy-id -i .ssh/id_rsa.pub mike@ubox
   
	
   退出ssh,重新登录ssh，检查是否已经不需要密码了。验证成功后，关闭ssh密码访问方式
   
#####    3 修改服务器/etc/ssh/ssh_config文件
   
	PubkeyAuthentication yes #启用public key访问
    PasswordAuthentication no #关闭密码
	ChallengeResponseAuthentication no#关闭密码
	PermitRootLogin no#禁止root用户登录
	
   修改完毕后，重新启动ssh服务
   
	sudo systemctl sshd restart	
	
<!-- more --> 
#### 1.2 更新系统，安装常用软件
   
#####   1 更新系统
   
    sudo yum update
    sudo yum install git gcc ruby
    
#####    2 配置时区 ntp
   
    sudo timedatectl set-timezone Asia/Shanghai
    sudo timedatectl
    sudo yum install -y ntp
    sudo systemctl start ntpd
    sudo systemctl enable ntpd
    
#####    3 配置swap文件
   
    sudo fallocate -l 2G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'
    
#####    4 检查监听端口，关闭不需要的服务
   
    netstat -an | grep LIS
    sudo service postfix stop
    sudo systemctl disable postfix
    
#####    5 安装rvm，使用rvm安装最新版本的ruby
   
    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
    \curl -sSL https://get.rvm.io | bash -s stable
    
    rvm list known
    rvm install 2.2-head
    rvm docs generate-ri #生成ri文档，可选
    
#####    6 安装vnstat，统计网络用量
   
    sudo yum install vnstat
    
    sudo vi /etc/cron.d/vnstat
    内容如下：每5分钟，更新一次数据库，注意修改用户为 root
    */5 * * * *  root /usr/sbin/vnstat.cron 
   
    vnstat --dumpdb #检查数据库
    vnstat --help #打印帮助
    vnstat -q #显示查询结果
    
   参考 [Linux VPS流量查看/监测工具 -- vnStat](http://www.vpser.net/manage/vnstat.html)
	
#### 1.3 防火墙设置

  centos 7默认是用firewall防火墙，关闭，修改为iptables
	
	systemctl stop firewalld#停止firewall
	systemctl disable firewalld.service#禁止firewall开机启动
	
	sudo yum install iptables-services
	sudo systemctl enable iptables
	sudo systemctl start iptables #启用iptables
	sudo systemctl  | grep firewall #检查防火墙
	
 启用fail2ban，修改/etc/fail2ban/jail.conf进行定制
 
    sudo yum install epel-release
    sudo yum install fail2ban
    sudo service fail2ban restart
    
#### 1.4 关闭selinux

  centos 8默认启用selinux,使用下面命令检查
  
	#sestatus 
	
	SELinux status:                 enabled  
	SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   enforcing
    Mode from config file:          enforcing 
	...
	
  用下面的命令关闭
  
	sudo setenforce 0
	
编辑

    sudo vi /etc/selinux/config
    
    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #    enforcing - SELinux security policy is enforced.
    #    permissive - SELinux prints warnings instead of enforcing.
    #    disabled - No SELinux policy is loaded.
    SELINUX=disabled
  
	
### 2 科学上网
  使用vps，翻墙技能是必须的，简单配置如下：
  
####   2.1 配置pptp隧道

参考文档

[CENTOS7下搭建 PPTP VPN](http://www.lkycn.com/2015/03/23/425.html)

[How To Setup Your Own VPN With PPTP](https://www.digitalocean.com/community/tutorials/how-to-setup-your-own-vpn-with-pptp)


具体步骤：

##### 1 安装 ppp pptpd
  
    sudo yum -y install ppp pptpd
  
##### 2 配置/etc/pptpd.conf

    debug #打开调试信息
    localip 10.0.0.1
    remoteip 10.0.0.100-200
    
##### 3 配置/etc/ppp/options.pptpd

    ms-dns 8.8.8.8 #配置dns
    ms-dns 8.8.4.4
    debug #打开调试信息
    logfile /var/log/pptpd.log #设置log文件
 
#####  4 配置 /etc/ppp/chap-secrets    #格式很通俗易懂。
 
	#client为帐号，server是pptpd服务，secret是密码，*表示是分配任意的ip
	 user          pptpd            pwd         *
    
#####  5 配置 /etc/sysctl.conf
 
    net.ipv4.ip_forward = 1
    sysctl -p #使内核参数生效
    
#####  6 配置iptables，启用nat地址转换，并且开放lcp gre 协议
 
    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE 
    sudo iptables -I INPUT -p gre -j ACCEPT
    sudo iptables -I INPUT -p tcp --dport 1723 -j ACCEPT
    sudo service iptables save
    
##### 8 配置开机启动

    sudo systemctl restart pptpd
    sudo systemctl enable pptpd
    
##### 9 如果有问题，检查日志文件的记录

    tail -f /var/log/pptpd.log
 

####   2.2 shadowsocks配置

 参考这篇：
 [Centos系统下shadowsocks-libev服务端的一键安装脚本的搭建及常见问题的解决方法](http://piaoyun.cc/centos-shadowsocks-libev.html)

 [CentOS下shadowsocks-libev一键安装脚本](http://teddysun.com/357.html)
 
 使用root用户登录，运行以下命令：
 
    wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-libev.sh
    chmod +x shadowsocks-libev.sh
    ./shadowsocks-libev.sh 2>&1 | tee shadowsocks-libev.log
    
    sudo vi /etc/shadowsocks-libev/config.json #修改配置
    sudo service shadowsocks start #启动服务
    sudo iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport 8989 -j ACCEPT #增加规则
    sudo service iptables save
    
####   2.3 安装anyconnect服务

记录 [Centos 7 ccserv anyConnect服务器配置说明](http://blog.mike163.net/2015/08/04/centos_setup_anyconnect/) 
  	
####   2.4 自定义proxy

  推荐 https://github.com/jiangmiao/proxy.git
  建议使用go语言版本. 编译命令
  
  	go build proxy.go 
   
  服务器端启动脚本：
    
    ./proxy -verbose  back :8781 &
    
  客户端启动脚本
  
  	./proxy -poolsize=10  front linode:8781
  	

   
   

 
 
   ``````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
   .  /;''''\¬ ≤≥    ƒ∫bfgv c
   rtzsdetgf[pv];'\\ m.bvbvv  √ 
