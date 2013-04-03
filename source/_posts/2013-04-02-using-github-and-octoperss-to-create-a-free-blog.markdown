---
layout: post
title: "使用github + Octopress 搭建免费博客"
date: 2013-04-01 15:14
comments: true
share: true
categories: tools
---
### 前提条件
注册一个github账号。

任何资料，都不如[Octopress](http://octopress.org/docs/) 和[Github Pages](https://help.github.com/categories/20/articles)的官方文档，建议首先阅读官方文档。

### GitHub Pages快速体验
在GitHub网站上，点击右上角的+号图标，创建一个新的Repo，Repository 的名字必须为 username.github.com。然后点击Settings进入该Repo的设置页面。看到"Automatic Page Generator"，说明这个Repo已经启用了GitHub Page。点击按钮进入设置。

在"Create a GitHub User Page"填写一些基本信息，点击右下角的"Continue to Layout"。布局就用默认的，点击绿色的"Publish"按钮。

大功告成，输入"username.github.com"，看到一个页面没？这就是你刚刚创建的一个页面。

<!--more-->

GitHub Pages分为两种类型，一种是"User and Org Pages"，一种是"Project Pages"。前者是用户的主页，一个用户仅有一个。后者是每个项目的主页。见github page官方的文档 [Creating Pages with the automatic generator](https://help.github.com/articles/user-organization-and-project-pages)。

本文创建的是第一种类型。

这篇博客 [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)  很通俗易懂，不过它创建的是第二种类型，在一个Repo上新建了一个branch，并命名为gh-pages。

下面正式开始折腾。

### 安装 msysgit并配置

* 下载[msysgit](http://msysgit.github.com/), 然后双击exe文件开始安装。
* 双击桌面图标Git Bash，启动一个shell，输入如下命令进行配置：

产生公钥ssh key，默认全部回车  
``` bash
    ssh-keygen -C github-account-email -t rsa
```


Note: username@email.com需要更换成你自己的在Github上注册的Email地址。
这样会在用户目录(C:\Documents and Settings\UserName)下产生一个.ssh文件夹，里面为对应的SSH Keys，其中id_rsa.pub是Github需要的SSH公钥文件。

在Github的Account Settings里选择SSH Keys，在其中将id_rsa.pub文件里内容拷贝至 其中的Key里。

这样以后就可以直接使用Git和GitHub了。  
    
测试一下
``` bash  
ssh -T git@github.com
```  

如果出现 hi xxx! You've successfully authenticated, bug GitHub does not povide shell access。说明SSH链接成功。

接下来配置其他信息。
``` bash  
	git config --global user.name github-username  
	git config --global user.email github-account-email  
	git config --global github.user github-username  
	git config --global credential.helper cache  
	git config --global credential.helper 'cache --timeout=3600'
```
本节参考了 [msysGit 安装后的配置](http://www.cnblogs.com/kysnail/archive/2012/03/16/2399589.html)。

### 克隆Repo到本地
在D盘新建一个文件夹，例如github。
``` bash
cd d:\github  
git clone git@github.com:username/username.github.com.git
```

### 安装Octopress
参考官方文档[setup](http://octopress.org/docs/setup/).  
**安装Ruby**  
Octopress 2.0 需要 Ruby 1.9.3，安装其他版本的Ruby可能会行不通。

如果是Linux，使用RVM来安装Ruby，如果是Windows，则使用[RubyInstaller](http://rubyinstaller.org/downloads/)。在这个[下载页面](http://rubyinstaller.org/downloads/)下载Ruby 1.9.3-p392和DevKit(带mingw的版本)，双击exe文件进行安装。  
**安装DevKit**  
双击DevKit的exe文件，解压到C:\DevKit
``` bash  
cd C:\DevKit
ruby dk.rb init
ruby dk.rb install
gem install rdiscount --platform=ruby
```
  
**安装Octopress**  
下载Octopress。
``` bash  
cd d:\github  
git clone git://github.com/imathis/octopress.git octopress  
cd octopress  
ruby --version  # Should report Ruby 1.9.3
rbenv rehash  # 可选，如果安装了rbenv，就需要执行这一步
```  

**注意**: rubygems.org在中国的下载速度很慢，会导致bundle install这一步下载gems的速度很慢，可能需要等待几个小时。因此需要事先切换到国内的镜像源。

用记事本打开octopress目录下的Gemfile，将第一行修改为
> source "http://ruby.taobao.org"

然后可以开始安装依赖的gems了。
``` bash  
bundle install
```  
正常的话应该可以看到一行行的Installing xxx，表示正在安装所需要的gem。

安装默认的Octopress主题。
``` bash  
rake install
```  
如果这一步出现问题，则试一下 bundle update再执行 rake install。

### 部署到GitHub
将Octopress和自己的Repo关联起来
``` bash  
rake setup_github_pages
```  
编译生成JeKyll所需要的静态文件
``` bash  
rake generate
```  
这个命令主要是根据source目录的内容，编译生成JeKyll所需要的静态文件，存放到public目录下。source 目录对应着git上的source分支。

预览
``` bash  
rake preview
```

部署到github
``` bash  
rake deploy
```  
该命令首先清空\_deploy目录，然后将public目录整个拷贝过来，然后commit到github。\_deploy 目录对应着master分支。

备份source到github
```
git add .
git commit -m 'your message'
git push origin source
```
source 目录下保存了所有的markdown源文件，是博客的原始数据，以及一些模板文件。因此很有必要备份。用上述命令提交到github，这样就用git管理起来了，再也不用担心数据丢失了。

**终止预览**  
启用`rake preview`后，直接按`ctrl+c`无法正常终止该进程，老提示`终止批处理操作吗（Y/N）？`，这时候可以另开一个Git Bash窗口，使用`ps aux | grep ruby`命令找出`pid(第一个数值)`，然后执行`kill <pid>`来停止该进程(参考[octopress on heroku (二)](http://linuxabc.heroku.com/blog/octopress-on-heroku-2))。  
**UTF-8 编码**  
Windows预设是Big5编码,所以要想’rake generate’的时候不报编码错误,我们需要设置一下编码! 方法有两个,一个是直接在Git Bash中设置环境:
``` bash  
set LANG=zh_CN.UTF-8  
set LC_ALL=zh_CN.UTF-8
```  
还有一个是在环境变量中加入这两个变量: 右击电脑->属性，新添加LANG和LC\_ALL两个环境变量，值为为zh_CN.UTF-8.

然后在Git Bash中做如下设置:
``` bash  
echo "export LANG LC_ALL" > ~/.bash\_profile
```
  
### 绑定域名
参考官方文档[Setting up a custom domain with Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-pages)。

非常简单，在master分支的根目录，添加一个文本文件，名字为CNAME，里面的内容就是要绑定的域名，例如本博客CNAME文件的内容是：
> www.yanjiuyanjiu.com

然后去DNSPod，添加一条CNAME，指向 username.github.com。例如我的为：
```
www	CNAME	默认	soulmachine.github.com.		-	600
```

很多人喜欢去掉www，用xxx.com的形式来访问，不过大家去试一下，在浏览器输入qq.com, douban.com, baidu.com，发现都会自动跳转到www，也就是说这些大网站，目前也是用www.example.com的域名为主，因此建议大家也这样做。

用www, blog之类的二级域名，还有个好处是方便升级，比如新版本用www1指向，等测试完成后，改成www指向，无缝切换。

如何让example.com 自动变成www.example.com呢？需要用 301重定向，在DNSPod上非常简单，添加一条显性URL即可，例如我的是：
```
@	显性URL	默认	http://www.yanjiuyanjiu.com	-	600
```

在使用Octopress的时候，每次`rake generate`, `rake deploy`后，master分支下面的CNAME文件消失了。正确的做法是，把CNAME文件放到在 source 目录下，其余的都删掉，`rake generate` 会自动拷贝到public目录下，`rake deploy`再拷贝public目录内容到\_deploy目录，并提交到master分支。



### 参考资料
1. [【原创】用Github和Octopress搭建博客](http://corey600.github.com/blog/2013/02/28/use-github-and-octopress-create-blog/)
1. [试用Octopress](http://www.blogjava.net/lishunli/archive/2012/03/18/372115.html)
1. [windows下安装DevKit](http://rubyer.me/blog/134/)
1. [在新Windows系统中重新部署Octopress](http://blog.sprabbit.com/blog/2012/12/21/recover-octopress/)
1. [Windows 8安装Octopress记录](http://hivan.me/octopress-install-to-windows8/)
1. [关于在64位 Windows 7 中部署中文化的Octopress](http://blog.sprabbit.com/blog/2012/03/23/octopress/)