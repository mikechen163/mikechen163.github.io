+++
title = "使用octopress搭建个人博客"
date = "2015-06-22"
categories = [
"技术",
]
+++


#### 1 在本地系统安装octopress系统。具体步骤参考
[Mac环境下部署Octopress个人博客到Heroku](http://sawyerzhu.herokuapp.com/blog/2014/05/06/octopress-deloyment-on-mac/)

#### 2 创建新文章

	rake new_post['文章标题']

#### 3 部署到heroku
  
	rake generate
	git add .
	git commit -m "add new post"
	git push heroku master


#### 4 部署到vps

   3.1 服务器端：安装httpd server
   
   	sudo yum -y install httpd
   	
   		
   3.2 服务器端：增加虚拟服务器配置
   
   在 /etc/httpd/conf.d目录下，添加一个vhost-name.conf文件，内容如下：

	   <VirtualHost *:80>
	 	 ServerName   你的域名
	  	 DocumentRoot /home/user/octopress  #注意这里user换成你的用户名
		</VirtualHost>
		<Directory /home/user/octopress>
    		Require all granted
		</Directory> 
   
 
   3.3  本地端：修改octopess目录下的Rakefile,填写ssh server的的相关信息 
   
   		ssh_user       = "username@MyVPS.com"
   		ssh_port       = "22"
   		document_root  = "~/octopress"
   		rsync_delete   = false
   		rsync_args     = ""
   		deploy_default = "rsync" 
   
   3.3 重新生成静态html页面,并部署到服务器
   
   		rake generate
   		rake deploy
 
#### 5 注意事项

   5.1 git使用http proxy，执行下面的命令：
   
    export all_proxy=socks5://127.0.0.1:1080
    
   5.2 ssh 登录免密码
   centos 7，ssh使用public key登录加了很多限制。
   生成ssh key,把 key拷贝到 ~/.ssh 下的authorized_keys里面后，还是不行，研究了半天，需要继续修改：
   
   在/etc/ssh/sshd_config文件中，
   
   	PubkeyAuthentication yes #缺省是no
   	
   在本地端和服务器端，都要修改id_rsa的权限为600，修改 .ssh的权限为700
   
    chmod 700 .ssh
    cd .ssh
    chmod 600 id_rsa*
    

#### 6 参考文档

[Mac环境下部署Octopress个人博客到Heroku](http://sawyerzhu.herokuapp.com/blog/2014/05/06/octopress-deloyment-on-mac/)

[像黑客一样写博客](http://www.yangzhiping.com/tech/octopress.html)

[部署Octopress到你的VPS](http://www.xiaozhou.net/deploy-octopress-to-your-vps-2013-08-13.html)

[Octopress第三方主题](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)


[heroku link](https://mike163.herokuapp.com/)
