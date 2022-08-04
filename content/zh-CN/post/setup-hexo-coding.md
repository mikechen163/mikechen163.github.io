+++
title = "使用hexo部署个人博客到coding.net"
date = "2016-06-17"
categories = [
"生活",
]
tags = [
"hexo",
]
+++

1 在coding.net创建一个新项目，名称和注册用户名一致。 类别为公开。

2 添加个人的public key到coding.net

3 修改hexo _config.yml文件，在 deploy 一节后添加下面的内容。 注意缩进对齐。

    - type: git 
      repository: git@git.coding.net:username/username.git  #把username 换成你的用户名
      branch: coding-pages
      
   执行 
       
       hexo g
       hexo d
       
   在coding.net项目页面，检查文件是否已经成功上传。    
      
4 修改项目的pages页面，启用pages，同时可绑定外部域名 

5 在你的域名服务器处，修改 外部域名，例如 blog.mike163.net 指向 mike163.coding.me

6 使用浏览器访问username.coding.me，如果成功，说明文件上传成功。再访问blog.mike163.net（换成你自己的外部域名），如果成功，说明外部域名也绑定成功。

参考：[coding.net个人页面设定帮助](https://coding.net/help/doc/pages/index.html)

