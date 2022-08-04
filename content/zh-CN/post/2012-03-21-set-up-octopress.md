+++
title = "使用octopress搭建博客记录"
date = "2012-03-20"
categories = [
"['技术']",
]
+++

打算建一个博客站很久了，由于种种原因，一直没有启动。作为技术人员，把博客搭建在新浪、百度上，总感觉不专业，现在有了octopress，又是用我喜欢的ruby语言，看了介绍后，感觉这是我想要的东西。下面说一下整个过程

 
##1.安装ruby环境##
我使用的是fedora 16 linux，装在virtual box 虚拟机里面。使用1.9.3版本的ruby代码，需要先安装yaml库，执行下面的代码

 wget http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz  
 tar xzvf yaml-0.1.4.tar.gz    
 cd yaml-0.1.4  
 ./configure --prefix=/usr/local  
 make  
 make install

然后再从ruby官网，下载ruby源代码  
 wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p125.tar.gz  
 tar xzvf ruby-1.9.3-p125.tar.gz   
 cd ruby-1.9.3-p125  
 ./configure --prefix=/usr/local --enable-shared --disable-install-doc --with-opt-dir=/usr/local/lib   
 make   
 make install

安装完后，执行ruby -v  
ruby 1.9.3p125 (2012-02-16) [i686-linux]


国内的用户，修改rubygems.org 为 ruby.taobao.org 进行加速。  
 gem sources --remove http://rubygems.org/  
 gem sources --add http://ruby.taobao.org/  
 gem sources list



##2.安装git ##
动作很简单 yum install git -y  
配置git  
git config --global user.name "your name"  
git config --global user.email yourname@email_server  
git config --global github.user username  
git config --global github.token tokenXXXXXXX  



##3.创建个人主页##

主要参考了这篇文章，[像黑客一样写博客](http://www.yangzhiping.com/tech/octopress.html)，基本上写的还是很明白。

需要注意的几点：
	
- rake deploy到 github后，要等一段时间，网站才能显示出来
- 如果使用了中文的页面，需要把页面保存为utf-8格式，才成正常显示中文
- 如果需要使用评论系统，需要[http://disqus.com](http://disqus.com)中创建一个账号，注意使用网站的短名，在_config.yml中，配置  
`disqus_short_name:  yourwebsite_shortname  
disqus_show_comment_count: true
`



----------

##参考文档##

1.像黑客一样写博客 [http://www.yangzhiping.com/tech/octopress.html](http://www.yangzhiping.com/tech/octopress.html)  
2.配置git [http://help.github.com/set-your-user-name-email-and-github-token/](http://help.github.com/set-your-user-name-email-and-github-token/)
