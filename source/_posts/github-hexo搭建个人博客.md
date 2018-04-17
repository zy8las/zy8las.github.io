---
title: 使用github+hexo搭建个人免费博客
date: 2017-10-19 11:56:22
tags:
---
# 一、前言
使用github pages服务器搭建博客的好处有：
1、全是静态文件，访问速度快；
2、免费方便，不用花一分钱就可以搭建一个自由的个人博客，不需要服务器不需要后台；
3、可以随意绑定自己的域名，不仔细看的话根本看不出来你的网站是基于github的；
4、数据绝对安全，基于github的版本管理，想恢复到哪个历史版本都行；
5、博客内容可以轻松打包、转移、发布到其它平台；
# 二、准备工作
1、有一个github账号，没有的话去注册一个，[官网](https://github.com/)；
2、安装了node.js、npm，并了解相关基础知识；
3、安装了git for windows（或者其它git客户端）
4、本文所使用的环境：
* Windows7 x64
* node-v6.11.4-x64
* git@1.9.2
* hexo@3.3.9

# 三、搭建环境准备

大概可以分为以下三步 
- Node.js 的安装和准备 
- git的安装和准备 
- gitHub账户的配置
## 3.1、配置Node.js环境
下载Node.js安装文件：
* [node-v6.11.4-x64](https://nodejs.org/en/download/)根据自己windows版本选择相应的安装文件。如图所示：
![nodejs](/images/githubHexo/nodejs.png)
保持默认设置即可，一路Next，安装很快就结束了。 然后我们检查一下是不是要求的组件都安装好了，同时按下Win和R，打开运行窗口：Windows的运行界面
![cmd](/images/githubHexo/cmd.png)
在新打开的窗口中输入cmd，敲击回车，打开命令行界面。（下文将直接用打开命令行来表示以上操作，记住哦~） 在打开的命令行界面中，输入
node -v
npm -v
如果结果如下图所示，则说明安装正确，可以进行下一步了，如果不正确，则需要回头检查自己的安装过程。
![cmdnode](/images/githubHexo/cmdnode.png)
配置Git环境

下载Git安装文件：

[git官网下载地址](https://git-scm.com/download/win)

然后就进入了Git的安装界面，如图：
![git](/images/githubHexo/git.png)
保持默认设置，一路next，直到安装完成。
右键->git bash here输入以下指令查看安装是否成功
`git --version`
如图：
![gitversion](/images/githubHexo/gitversion.png)

## 3.1、github账户的注册和配置
如果已经拥有账号，请跳过此步~

3.11、Github注册
打开https://github.com/，在下图的框中，分别输入自己的用户名，邮箱，密码。
然后前往自己刚才填写的邮箱，点开Github发送给你的注册确认信，确认注册，结束注册流程。
一定要确认注册，否则无法使用gh-pages！
3.12、创建代码库
登陆之后，点击页面右上角的加号，选择New repository：
![gitku](/images/githubHexo/gitku.png)
新建代码库

进入代码库创建页面：

在Repository name下填写yourname.github.io，Description (optional)下填写一些简单的描述（不写也没有关系），如图所示：
![gitre](/images/githubHexo/gitre.png)
注意：比如我的github名称是zy8las ,这里你就填 zy8las.github.io,如果你的名字是xujun，那你就填 xujun.github.io

代码库设置
正确创建之后，你将会看到如下界面：
![gitpe](/images/githubHexo/gitpe.png)
接下来开启gh-pages功能，点击界面右侧的Settings，你将会打开这个库的setting页面，向下拖动，直到看见GitHub Pages，如图：

点击Automatic page generator，Github将会自动替你创建出一个gh-pages的页面。 如果你的配置没有问题，那么大约15分钟之后，yourname.github.io这个网址就可以正常访问了~ 如果yourname.github.io已经可以正常访问了，那么Github一侧的配置已经全部结束了。
![gitset](/images/githubHexo/gitset.png)
到此搭建hexo博客的相关环境配置已经完成，下面开始讲解Hexo的相关配置

## 3.2、安装Hexo

在自己认为合适的地方创建一个文件夹，这里我以E：/hexo 为例子讲解，首先在E盘目录下创建Hexo文件夹，并在命令行的窗口进入到该目录
![cmdhexo](/images/githubHexo/cmdhexo.png)
在命令行中输入：

npm install -g hexo
可能你会看到一个WARN，但是不用担心，这不会影响你的正常使用。 然后输入

npm install hexo --save

然后你会看到命令行窗口刷了一大堆白字，下面我们来看一看Hexo是不是已经安装好了。 在命令行中输入：

hexo -v
![hexov](/images/githubHexo/hexov.png)
如果你看到了如图文字，则说明已经安装成功了。
## 3.21、hexo的相关配置

1、初始化Hexo
接着上面的操作，输入：
hexo init
然后输入：

npm install

之后npm将会自动安装你需要的组件，只需要等待npm操作即可。

2、首次体验Hexo
继续操作，同样是在命令行中，输入：

hexo g
![hexog](/images/githubHexo/hexog.png)
然后输入：

hexo s

然后会提示：

INFO Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
在浏览器中打开http://localhost:4000/，你将会看到：
![hexopage](/images/githubHexo/hexog.png)
到目前为止，Hexo在本地的配置已经全都结束了。

下面会讲解怎样将Hexo与github page 联系起来

# 四、怎样将Hexo与github page 联系起来

大概分为以下几步 
- 配置git个人信息 
- 配置Deployment

配置Git个人信息

如果你之前已经配置好git个人信息，请跳过这一个 步骤，直接来到

1、设置Git的user name和email：(如果是第一次的话)

git config --global user.name "zy8las"
git config --global user.email "464999578@qq.com"

2、生成密钥

 ssh-keygen -t rsa -C "464999578@qq.com"(换成你的邮箱地址)
接着出现的一些步骤都可以回车跳过。
 
2.1、配置github账户的ssh key
打开id_rsa.pub文件将一整串公钥拷贝下来

进入你的github账户设置，在ssh and GPG keys中新增一个ssh key，如下 
![ssh](/images/githubHexo/ssh.png)
 同时需要验证ssh key 
 ssh -T git@github.com
 
 出现下面的语句说明你的ssh key已经配置好了
 Hi wispyoureyes! You've successfully authenticated, but GitHub does not provide shell access.
## 4.1、配置Deployment
 
 同样在_config.yml文件中，找到Deployment，然后按照如下修改：
 
 deploy:
   type: git
   repo: git@github.com:yourname/yourname.github.io.git
   branch: master
   
 比如我的仓库的地址是git@github.com:zy8las/zy8las.github.io.git，所以配置如下
 
 deploy:
   type: git
   repo: git@github.com:zy8las/zy8las.github.io.git
   branch: master

# 五、写博客、发布文章

新建一篇博客，执行下面的命令：
hexo new post "article title"
![blogs](/images/githubHexo/blogs.png)
这时候在我的 电脑的目录下 F:\hexo\source\ _posts 将会看到 article title.md 文件

用MarDown编辑器打开就可以编辑文章了。文章编辑好之后，运行生成、部署命令：
hexo g   // 生成
hexo d   // 部署
当然你也可以执行下面的命令，相当于上面两条命令的效果
hexo d -g #在部署前先生成
部署成功后访问 你的地址，https://yourName.github.io   （这里输入我的地址： https://zy8las.github.io ),将可以看到生成的文章。
### 踩坑提醒
1）注意需要提前安装一个扩展：
npm install hexo-deployer-git --save
如果没有执行者行命令，将会提醒

deloyer not found:git
# 六、主题推荐
每个不同的主题会需要不同的配置，主题配置文件在主题目录下的_config.yml。有两个比较好的主题推荐给大家。
## 6.1、Yilia

Yilia 是为 hexo 2.4+制作的主题。 
崇尚简约优雅，以及极致的性能。
[Yilia地址](https://github.com/litten/hexo-theme-yilia)

## 6.2、NexT

[配置教程](http://theme-next.iissnan.com/getting-started.html)

