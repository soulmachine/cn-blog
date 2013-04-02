---
layout: post
title: "我的Octopress配置"
date: 2013-04-02 15:35
comments: true
categories: 
---
## 实时预览
使用如下命令可以实现实时预览：
``` bash
rake preview  
```

`rake preview` 会自动监视文件的变化，重新生成静态页面。因此修改markdown文件后，只需要在浏览器里刷新一下页面，就立刻可以看到效果。

## 嵌入代码块
见官方文档[Sharing Code Snippets](http://octopress.org/docs/blogging/code/)。

Octopress是一款为hacker量身定制的博客系统，当然内置了代码高亮的功能！它的代码高亮功能是通过Pygments实现的，配色方案用的是Solarized，堪称完美。

Octopress支持多种方式嵌入代码，可以直接嵌入代码，也可以引用github上的gist 。

我喜欢用**三个反引号**直接嵌入代码，比 `codeblock`要简洁。

## 首页只显示部分正文(Excerpts)
Octopress中，可以使用 `<!--more-->`，这样首页只显示一部分正文，并在每篇文章底下加一个Read on超链接。

## 插入图片
使用[Image Tag](http://octopress.org/docs/plugins/image-tag/)。

语法
```
{% img [class names] /path/to/image [width] [height] [title text [alt text]] %}
```

例子
```
{% img http://placekitten.com/890/280 %}
{% img left http://placekitten.com/320/250 Place Kitten #2 %}
{% img right http://placekitten.com/300/500 150 250 Place Kitten #3 %}
{% img right http://placekitten.com/300/500 150 250 'Place Kitten #4' 'An image of a very cute kitten' %}
```

<!--more-->

###定制主题
### 添加about me 边栏
编辑 source\_includes\custom\asides\about.html，内容如下：
```
<section>
  <h1>About Me</h1>
  <p>一句话自我介绍.</p>
  <p>Sina Weibo: <a href="http://weibo.com/soulmachine">@soulmachine</a><br/>
     Twitter: <a href="https://twitter.com/#!/soulmachine">@soulmachine</a><br/>
     Other: <a href="https://github.com/soulmachine">Github</a>, <a href="https://plus.google.com/103519507226474510310">Google+</a>, <a href="http://www.linkedin.com/in/soulmachine">LinkedIn</a>, <a href="http://www.quora.com/Jason-Day-2">Quora</a></p>
  </p>
</section>
```
在 _config.yml 的 default_asides 里添加 custom/asides/about.html。

### 其他
启用 twitter 分享， facebook like 和Google +1，设置如下：
```
google_plus_one: true
twitter_tweet_button: true
facebook_like: true
```

启用Disqus，填入 short name即可。

填入 Google Analytics Tracking ID，例如 UA-7583537-4。

