---
layout: post
title: "Octopress搭建个人博客"
date: 2015-11-14 11:21:39 +0800
comments: true
categories: other
---
本人的博客刚刚搭建好，并绑定了个人域名([guimingsu.com](http://guimingsu.com))，现将该过程记录下来。

[Github Pages](https://pages.github.com/)是Github提供的一个免费空间，相当于我们博客的免费托管服务器，我们写的博客就是放这上面的。它可以拥有一个独立的二级域名如xxx.github.io(如果有自己的个人域名，可以用自己的个人域名指向它)，允许开发者提交静态网页文件，用于介绍自己，或者自己的开源项目，可以看作是个人或项目主页。

[Octopress](http://octopress.org/)“A blogging framework for hackers”，即像写代码似的写博客。一个博客网页生成框架，我们按照它要求的格式写博客内容，然后敲命令`rake generate`,`rake deploy`,就可以把网页放到Github服务器上，然后我们就可以看到了啊，它就是一个写博客的工具可以这样理解。

个人域名，这个要自己买了，可以上[阿里云](http://wanwang.aliyun.com/)上买，一年几十块，配置后面再讲。在注册阿里云的时候我用的QQ邮箱，就是收不到邮件，换成Gmail就收到了。不知道是我网络问题还是大公司间的任性撕逼，如果你也收不到邮件就换个邮箱注册吧。<!--more-->

* 在[Github](https://github.com)上New repository, 名字为`yourNmae.github.io`确定提交，这里报错是因为我已经建好了。
![creat_rep](/myimg/other/creat_rep.png)
之后会生成一个HTTPS连接如 https://github.com/yourName/yourName.github.io.git ,这个连接在之后的Octopress绑定配置中要用。

* [Octopress](http://octopress.org/docs/setup/)这个步骤比较多一点, 我用的系统是 OS X10.10.5。Octopress要用到ruby环境，且ruby版本要大于等于`1.9.3`。可以用`ruby -v`命令看一下ruby版本，我的是版本是`2.0.0p643`, 没有的话可以看[这里](https://ruby-china.org/wiki/install_ruby_guide)安装ruby环境。

环境装好后从网上下载Octopress框架到本地电脑上，这里我保存到octopress文件夹且放在桌面上

下载： `git clone git://github.com/imathis/octopress.git /Users/suguiming/Desktop/octopress`

然后进入该文件夹：`cd  /Users/suguiming/Desktop/octopress`

然后：  `gem install bundler`

这里可能会报错，说缺少xxx依赖包，可以用`gem dependency`查看要依赖的包，然后把包都装上再敲上面的命令

然后： `bundle install` 

再然后：  `rake install` 

这里Octopress就下载安装好了，之后要进行关联配置,把Octopress和Github关联起来，我们就可以把博客放到之前建的Github项目`yourNmae.github.io`里了。

但关联前还有一个步骤就是SSH key处理。新建一个命令窗口`cd ~/.ssh`进入目录，`ls -a`查看内容，这时候一般没有`id_rsa`及`id_rsa.pub`这两个文件的，我们现在要创建这两个文件。`ssh-keygen -t rsa -C "Github登录账号邮箱@qq.com"`,然后按回车回车再回车就生成那两个文件了。然后把`id_rsa.pub`里的东西全选复制，在[这里](https://github.com/settings/ssh)添加一个SSH key即可。

回到octopress的命令窗口，输入命令`rake setup_github_pages` 此时要求输入Github项目`yourNmae.github.io`的地址，即前面得到的`https://github.com/yourName/yourName.github.io.git`,也可在Github中项目里的右下角的HTTPS链接复制过来然后

    rake generate
    rake deploy
    git add .
    git commit -m 'say something'
    git push origin source


* 写博客，`rake new_post["file name"] `新建一个写博客的文件，它会在`/source/_posts`文件夹里生成,打开它在里面写博客内容就可以了。它是用markdown的方式写，可以在[这里](http://wowubuntu.com/markdown/)和[这里](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)查看markdown的使用规则，常用的就那几个，熟悉就可以了,大概长这样。推荐使用[Mou](http://mouapp.com/)来编写markdown文件，[马克飞象](https://maxiang.io/)这个在线编写工具也很好。
    ![write_blog](/myimg/other/write_blog.png)
    写好后保存，`rake generate`生成博客，`rake preview`预览博客，在地址栏输入`http://localhost:4000/`查看，退出预览`control+C`

然后提交，之后在地址栏输入`yourName.github.io`就可以查看了

    rake deploy
    git add .
    git commit -m 'say something'
    git push origin source

* 域名绑定，在`yourName.github.io`项目里新建一个文件，文件名必须是大写的`CNAME`,内容是你购买的域名如我的域名`guimingsu.com`,前面不要加www或http,参考[这里](https://help.github.com/articles/adding-a-cname-file-to-your-repository/),然后`git pull`把刚才建的文件同步到本地。
然后在域名购买的服务商那里设置，我用的是[阿里云](http://www.aliyun.com/)。登录账号，在控制管理，域名解析中添加域名指向,把图中的`andyfightting.github.io`换成你自己的，其他的不变,这样域名就配置好了
![img](/myimg/other/yuming.png)
参考[这里](https://help.github.com/articles/my-custom-domain-isn-t-working/)
![img](/myimg/other/dns_error.png)

* 其他个性化配置，如在浏览器显示的icon替换，在source文件夹里面有个文件叫favicon.png，做一个16*16图片替换进去提交，可能不会立即有反应。还有博客的评论系统是用的第三方[disqus](https://disqus.com)。在该网站上注册一个账号，然后把账号名填写在`/octopress/_config.yml`文件中对应的地方，把false改成true。

![img](/myimg/other/discus.png)

由于访问国外disqus比较慢，我又把评论部分改用国内的[多说](http://duoshuo.com/)了，使用方法都类似的，具体配置请看[这里](http://www.tuicool.com/articles/VbqYNjn),里面还说了性能优化可以跟着改改，提高网页的响应速度。









