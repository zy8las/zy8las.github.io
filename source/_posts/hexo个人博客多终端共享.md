---
title: hexo个人博客多终端共享
date: 2018-04-18 14:34:59
tags:
---
主要思路就是利用git分支实现
hexo生成的静态博客文件默认放在master分支上。
hexo的源文件（部署环境文件）可以都放在hexo分支上（可以新创建一个hexo分支），换新电脑时，直接git clone hexo分支（git@github.com:Michaeljian/Michaeljian.github.io.git）即可。

# 一、hexo搭建博客原理
hexo博客的部署环境目录：
![hexomulu](/images/hexoshare/hexomulu.jpg)
hexo博客的目录结构解析：

![jiegou](/images/hexoshare/jiegou.png)

source tree 创建一个hexo仓库将文件上传到github的hexo仓库

1.hexo帮助把博客发送到github，同时把md文件转换成网页文件。
2.hexo目录下的文件和github上的文件是不同的，public文件夹的文件通过hexo d 上传到github去了，其他的文件则留在本地目录下。

# 二、其他电脑
将新电脑的生成的ssh key添加到GitHub账户上（生成方式：ssh-keygen -t rsa -C "464999578@qq.com"）
配置git账户信息：
              git config --global user.name "zy8las"
              git config --global user.email "464999578@qq.com"）
在新电脑上克隆username.github.io仓库的hexo分支到本地，此时本地git仓库处于hexo分支
切换到username.github.io目录，执行npm install(由于仓库有一个.gitignore文件，里面默认是忽略掉 node_modules文件夹的，也就是说仓库的hexo分支并没有存储该目录[也不需要]，所以需要install下)
