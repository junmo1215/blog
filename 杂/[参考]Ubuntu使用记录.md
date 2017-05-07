---
date: 2017-5-7 15:10
status: public
title: '[参考]Ubuntu使用记录'
---

记录一些自己用到的Ubuntu零碎知识点

使用的系统是ubuntu 16.04 LTS

![ubuntu_version](http://7xrop1.com1.z0.glb.clouddn.com/others/ubuntu_version.png)

# 修改环境变量

修改 /etc/environment 文件，直接添加`变量名='变量值'`的方式修改，不需要export，这个文件修改的是整个系统的环境。

> 修改这个文件需要root权限，可以在terminal中使用vim修改，也可以复制到其他地方然后粘贴回etc文件夹。
> 粘贴命令可以使用：`sudo cp -f '/home/junmo/桌面/environment' '/etc/environment' `

参考：
- [设置Linux环境变量的方法和区别_Ubuntu_给力星](http://www.powerxing.com/linux-environment-variable/)

# 命令行安装软件

执行指令：

``` sh
sudo dpkg -i [名称].deb
```

> **注：dpkg命令无法自动解决依赖关系。如果安装的deb包存在依赖包，则应避免使用此命令，或者按照依赖关系顺序安装依赖包。**

参考：
- [ubuntu下如何用命令行运行deb安装包 - windtail - 博客园](http://www.cnblogs.com/windtail/archive/2012/06/02/2623175.html)

