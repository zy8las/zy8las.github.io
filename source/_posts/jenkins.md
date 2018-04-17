---
title: jenkins安装详解windows 7
date: 2017-10-19 16:25:23
tags:
---
# 一、修改默认的jenkins安装路径

因为jenkins默认安装在c盘 C:\Users\Administrator\.jenkins下，那怎样将安装路径修改至e盘呢？

新建一个系统变量：JENKINS_HOME，值为E:\jenkins\jenkinspro，或者其他目录。再将此路径添加至Path里。
![huanjing](/images/jenkins/huanjing.jpg)
# 二、安装jenkins 
* 2.1、下载jenkins
首先到Jenkins官网下载windows安装包。在主页的右侧版本下载框里选择Windows，下载zip文件。
[官网](https://jenkins.io/)
* 2.2、解压运行，一路下一步(运行前先关闭8080端口，否则可能安装不成功，关闭方式查看温馨提示)
* 2.3、安装完成后，安装程序会自动打开默认浏览器，打开Jenkins主页。
* 2.4、jenkins安装完成
![jenkins](/images/jenkins/jenkins.jpg)
# 三、修改jenkins默认端口
可能有一些原因，8080端口被占用了，无法使用时需要修改jenkins的启动端口号。
如果首次安装，建议先停止原有系统的8080端口占用，等jenkins安装完成后，再进行修改，然后该回8080的原系统端口。
* 3.1、先停止jenkins服务
win+r-》cmd-》net stop jenkins
* 3.2、打开E:\jenkins\Jenkins\jenkins.xml
将<arguments>元素中的httpPort的值8080改为其他值即可
```
<arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -jar "%BASE%\jenkins.war" --httpPort=8099</arguments>
```
* 3.3、重启jenkins服务
win+r-》cmd-》net start jenkins

# 温馨提示
查看网络端口pid
win+r-》cmd-》netstat -ano|findstr 8080
停止响应pid端口
win+r-》cmd-》taskkill /pid pid号 /f