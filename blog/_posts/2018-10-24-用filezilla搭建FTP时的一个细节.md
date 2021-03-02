---
title: 用filezilla搭建FTP时的一个细节
key: 20181024
tags: Linux
---
最近用自己的Linux主机搭建FTP，具体的配置不多说了基本是参照网上的教程，设置好ftp账号和密码之后使用其他设备登录就可以访问电脑上的事务了，而且我看移动端的FTP软件做得很不错，速度也很快。不过我遇到了一个问题：用手机平板通过设置的ftp账号登录后，默认的根目录就是你的FTP_user的目录(/home/FTP_user)，问题是你怎么能访问到其他的目录？比如说/home/working/Documents？或者/mnt/？比如我的1TB硬盘里面都是电影，他们都在/mnt/下。还有假如接了移动硬盘的话好像也访问不到了。

这个问题我想了好久也没有思路，google “ftp visiting other disks"，结果教程上都是说在设置vsftpd的配置文件的时候可以配置的。但是有很大的安全隐患...总之是具有劝退意味的方法。
 
其实，不这样做也没关系。只要直接把这些目录做成链接形式放到/home/FTP_user文件夹下就可以了。例如进入你的/home/FTP_user目录：
```angular2html
ln -s /mnt/ my_disk_link
```
这样在本目录下生成了/mnt的符号链接文件，这样FTP登陆下直接点my_disk_link就可以直接访问你的/mnt目录了。