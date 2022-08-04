+++
title = "ssh攻击辅助分析程序"
date = "2015-06-22"
categories = [
"技术",
]
tags = [
"ssh",
]
+++
每次登录到系统，使用 

	netstat -an | grep LIS
	cd /var/log
	tail -n 1000 secure | grep Failed
命令，都可以发现很多尝试登录系统的请求。 对于普通的扫描就算了，然而对于某个攻击的ip，会疯狂的尝试登录。这种情况下，必须禁止这类IP。


通过程序，找出这类IP，并生成iptables 命令，典型用法如下：
	

	#cd /var/log
	#tail -n 1000 secure | grep Failed > ssh_log.txt
	#ruby analysis_ssh_log.rb -f ssh_log.txt   #显示ssh攻击的ip地址
	{:ip=>"43.255.188.165", :counter=>220, :time_stamp=>"Jun 18 05:09:01"}
	{:ip=>"182.100.67.112", :counter=>168, :time_stamp=>"Jun 18 04:10:39"}
	{:ip=>"43.255.188.148", :counter=>16292, :time_stamp=>"Jun 18 04:01:23"}
	{:ip=>"43.229.52.78", :counter=>945, :time_stamp=>"Jun 18 04:27:06"}
	
	#ruby analysis_ssh_log.rb -m ssh_log.txt   #生成相应的iptables命令
	iptables -I INPUT -s 43.255.188.0/24 -j DROP
	iptables -I INPUT -s 43.229.52.78 -j DROP
	iptables -I INPUT -s 182.100.67.0/24 -j DROP
	
	如果不需要了，可以使用-v选项生成删除命令
	#ruby analysis_ssh_log.rb -v -m ssh_log.txt   #生成相应的iptables命令
	iptables -D INPUT -s 43.255.188.0/24 -j DROP
	iptables -D INPUT -s 43.229.52.78 -j DROP
	iptables -D INPUT -s 182.100.67.0/24 -j DROP
	
	
  
   附录；程序如下
       
  {% include_code ruby/analysis_ssh_log.rb %}
 

