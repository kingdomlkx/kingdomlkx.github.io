---
title: 打造个人博客网站
auther: 雁引愁心去
layout: post
---

---

上周心血来潮想要做一个博客来写点东西,一方面记录一下我的学习生活,另一方面是感觉很有趣.百度了之后,最终还是选择了自己建一个博客网站---利用Github-Page.成果就是这个网站.所以第一篇博客就记录一下建立的过程.

- 在这之前需要自己熟练掌握一些知识:  
  > 1. Git工具  
  > 2. jekyll  
  > 3. markdown  
  > 4. 一些必要的HTML语言知识  
  > 5. liquid模板语言  

这些知识在建立博客和写作博客中时常会用到,如果现在还没入门,也不用担心,我会的也不多.但相信,只要经常写博客,这些东西早晚会一点一点学会并熟练掌握的.

*本文是在manjaro上实现的*  

---

### 1. 介绍一下Github-Page和jekyll

**Github-Page**

Github-Page是面向用户,组织和项目开放的公共静态页面托管服务,站点可以被免费托管在Github上,可以选择用Github-page默认提供的域名,也可以自己花钱买个域名.

**Jekyll**

jekyll是一个简单免费的博客生成工具,而且生成的是静态网页,不需要数据库支持,而且可以配合第三方服务.最关键的是Jekyll也是Github官方支持的.

### 2. 安装Jekyll和Git

- 安装jkeyll  
*注意Jekyll是用ruby写的,需要先安装ruby和gem,bundler*  
  >`$ sudo pacman -S ruby`  
  >`$ sudo pacman -S gem`  
  >`$ sudo gem install bundler Jekyll`  
如果运行下列命令正常,说明安装成功  
  >`$ ruby --version`  
  >`$ gem --version`  
  >`$ budle --version`  
  >`$ jekyll --version`  

- 安装Git工具
  >`$ sudo pacman -S Git`

### 3. 申请Github page
- 打开你的github,新建一个新的仓库,仓库的名字为"your-github-name.github.io"

![图1](/assets/images/2019-9-11/2019-9-11-1.jpg)

完成后点击creat

### 4. 本地博客

- 新建一个blog文件夹
- 在[jekyll主题网站](http://jekyllthemes.org/)选择一个喜欢的主题.
![图2](/assets/images/2019-9-11/2019-9-11-2.jpg)

这里可以点击Demo预览主题.选择喜欢的可以直接点击Download下载,也可以点击Homepage进入其github主页.用git工具clone.这里使用第二种方法.  
进入刚刚新建的blog文件夹,以此主题为例.

>`$ git clone git@github.com:murraco/jekyll-theme-minimal-resume.com`  
此时文件夹下已经新建了一个 **jekyll-theme-minimal-resume** 的文件夹  
>`$ cd jekyll-theme-minimal-resume`  
>`$ bundle exec jekyll serve`  
这时会自动生成一个 **_site** 的文件夹,这个文件夹中的内容仅仅是用来生成本地网页的,一般来说会自动加入 **.gitignore** 文件中,如果没有请手动加入.

打开浏览器,进入 **127.0.0.1:4000** 就可以看到博客的本地网页了.使用博客的本地网页,可以在上传之前预览,并提早修改,避免麻烦.

### 5. 上传到远程 github page
>*生成SSH密钥,注意这个email需要是你自己的email*  
>`ssh-keygem -t rsa -C "your_email@example.com"`

进入主目录下的 **.ssh/** 目录,复制 **id_rsa.pub** 中的内容.登陆github,添加ssh公钥.  

![图3](/assets/images/2019-9-11/2019-9-11-3.jpg)  
>*关联到远程github page 库*  
>`$ git remote add origin git@github.com:your-username/your-username.gihub.io`  
>*获取远程库的分支情况*  
>`$ git pull`  
>*git三部曲*
>`$ git add .`  
>`$ git commit -m "first push"`  
>`$ git push origin master`  
>__*注意github page展示的网页必须上传到master分支上*__

这时你的博客站点已经上传到github page上了,初次生成网页可能会有点慢,稍等四五分钟就好.浏览器输入 **https://your-username.github.io** 就可打开你的博客网站了.当然目前仍然只是模板主题,之后就需要自己掌握了解jekyll并进行写作和上传了.

### 6. 申请域名  
如果你不想使用github page的域名,你也可以自己购买一个域名,添加并解析到github page上.
- 添加到gihub page上  
打开你的github page 仓库.点击setting.向下找到github page的设置,在 **Custom domain** 中输入你的域名,点击save会生成一个新的名为 **CNAME** 的文件.之后需要在本地重新执行 `git pull` 来解决文件冲突.  
![图4](/assets/images/2019-9-11/2019-9-11-4.jpg)
*注意:购买域名之后需要先实名认证之后解析到github page.实名认证的的时间较长,可能需要一两天,在此之前不能解析域名.并且如果没有解析就将其添加到github page上会产生域名错误.所以需要解析完成之前不要添加域名.*

到此个人博客网站已经初步完成了,但之后需要完成和学习的还有很多.
