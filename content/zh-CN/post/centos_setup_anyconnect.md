+++
title = "Centos 7 ocserv AnyConnect服务器配置说明"
date = "2015-08-04"
categories = [
"技术",
]
tags = [
"ocserv",
"anyconnect",
]
+++
    

参考文件

 [在 CentOS 7 上搭建 Cisco AnyConnect VPN](http://www.xuanlove.net/jishujiaoliu/1210.html)
 
 [Gentoo编译安装Ocserv上Cisco AnyConnect VPN](http://blog.ihipop.info/2014/07/4782.html) 
 
  [折腾笔记：架设OpenConnect Server给iPhone提供更顺畅的网络生活](http://bitinn.net/11084/) 
 
 安装没有任何问题，直接安装即可
 
    sudo yum install epel-release
    sudo yum install ocserv
 

 遇到的主要问题：
 
#####  1 本地lan禁止vpn。需要修改/etc/ocserv/ocserv.conf
 
    #注意 这里推送路由条数限制据说只有64条 而且不能指定是走还是不  走路由 这点没有openVPN灵活 我这里全部把它注释了 走全局了
    #设定全部ip都走 VPN，由于本地路由有优先表，所以本地路由不走vpn
    route = 0.0.0.0/128.0.0.0
    route = 128.0.0.0/128.0.0.0
    
   另外一个办法是配置常用的路由，客户端最大支持200条，服务器缺省64条，如果要修改，需要重新编译。[这里](http://blog.ltns.info/wp-content/uploads/2014/09/ocserv.zip) 可以下载一个route配置样本
 
#####  2 启动用户证书登录。
 
   用户证书制作脚本，需要一个参数 用户名
      
    sh make_user_cert.sh username
    
  脚本如下： 
    
    if [ -z "$1" ];then
      echo "please input user_name for cert "
    exit

    else
     mkdir $1
     cd $1
     cp ../ca-cert.pem .
     cp ../ca-key.pem  .
     certtool --generate-privkey --outfile user-key.pem

     cat >user.tmpl<<EOF
      cn = "$1"
      unit = "my company"
      expiration_days = 3650
      signing_key
     tls_www_client
    EOF  #注意，文件中这个EOF必须放在行首，否则shell会报错误
  
      certtool --generate-certificate --load-privkey user-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template user.tmpl --outfile user-cert.pem
      certtool --to-p12 --load-privkey user-key.pem --pkcs-cipher 3des-pkcs12 --load-certificate user-cert.pem --outfile user.p12 --outder
      mv user.p12 $1.p12

    fi
 
<!-- more --> 
 
##### 3  修改/etc/ocserv/ocserv.conf
  
    # 确保服务器正确读取用户证书（后面会用到用户证书）
    cert-user-oid = 2.5.4.3
    
    # 注释掉所有的route，让服务器成为gateway，如果是手机用，则全部流量都走vpn
    #route = 192.168.1.0/255.255.255.0
    
    # 改为证书登陆，注释掉原来的登陆模式
    auth = "certificate"
    #auth = "plain[/etc/ocserv/ocpasswd]"
    
    # 证书认证不支持这个选项，注释掉这行
    #listen-clear-file = /var/run/ocserv-conn.socket
    
    # 启用证书验证证书，实际路径和你的CA的证书路径一致即可。
    ca-cert = /etc/ssl/private/my-ca-cert.pem
   
     #！！！把这句注释掉，否则客户端总是说下载不了配置文件，导致连接失败
    #user-profile = profile.xml
    
 4 启动服务
   
    sudo service ocserv start
    sudo systemctl enable ocserv #自动启动

5 分发到用户。
通过邮件或者其他方式，把用户证书发给手机。然后在客户端菜单-诊断-证书管理-导入，导入生成的证书。
  	
