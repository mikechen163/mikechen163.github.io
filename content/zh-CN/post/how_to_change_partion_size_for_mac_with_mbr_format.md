+++
title = "MAC OS X下的一些小技巧"
date = "2015-08-04"
categories = [
"技术",
]
tags = [
"Mac OS X",
"NTFS",
]
+++

### 1 MAC OS X下修改NTFS分区大小

一个移动硬盘，买来后格式为NTFS格式，就一个分区。使用mac磁盘工具，提示分区格式为MBR主引导记录格式，无法改变大小。

解决方法如下：

1、首先在windows系统下，通过diskpart压缩分区大小

2、把空出来的空间，创建一个格式为exFAT的分区

3、在MAC 磁盘工具中，抹去这个exFAT格式的分区，重新格式化为HFS+

### 2 MAC 支持NTFS写操作

mac os x是原生支持ntfs读操作的，但由于某些原因，写操作被屏蔽了。有的时候我们需要打开ntfs的写操作。可以通过修改 /etc/fstab实现。

    #假设移动硬盘的卷标是My Passport,那么修改如下
     LABEL=My\040Passport none ntfs rw,auto,nobrowse
   
### 3 iTunes Photo支持多个资料库

检查了一下磁盘空间，photo资料库居然达到80个G了，考虑把photo资料库转移到移动硬盘上去，留出空间给其他磁盘用。

1、考虑资料库到移动硬盘。移动硬盘的分区最好为HFS+格式

    cd /Volume/backup/"照片 Library.photoslibrary"
    ruby /Users/mike/work/tools/sync.rb -c /Users/mike/Pictures/"照片 Library.photoslibrary" .
 
2 按住option键，打开照片应用，此时会出现一个对话框让你选择资料库。此时可以检查备份的照片是否都OK了。

3 iTunes的资料库也可以同样处理。 

    cd /Volume/backup/iTunes
    ruby /Users/mike/work/tools/sync.rb -c /Users/mike/Music/iTunes .

  
4 目录同步的小工具
这个工具会递归的坚持目标路径、文件是否存在，如果不存在，则创建目录、复制文件；如果存在，再检查判断文件大小是否和源文件一致，如果不一致，就覆盖目标文件。 

<!-- more --> 
 {% include_code ruby/sync.rb %}
 
 
 参考文档
 [在Mac中Option键 的 使用技巧](http://www.uc.hk/post/87.shtml)
 
### 4 locate命令使用spotlight数据库

在 .bashrc 或者 .zshrc 文件中加入：

    function locate { mdfind "kMDItemDisplayName == '$@'wc"; }
    
   就可以让locate命令使用spotlight的数据库了
   
   
