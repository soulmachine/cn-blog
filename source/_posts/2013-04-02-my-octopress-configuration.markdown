---
layout: post
title: "我的Octopress配置"
date: 2013-04-02 15:35
comments: true
categories: tools
---
## 实时预览
使用如下命令可以实现实时预览：
``` bash
rake preview  
```

`rake preview` 会自动监视文件的变化，重新生成静态页面。因此修改markdown文件后，只需要在浏览器里刷新一下页面，就立刻可以看到效果。不过如果修改了_config.yml的话，则需要Ctrl+C终止，用`rake generate`重新生成，才能看到效果。

## 嵌入代码块
见官方文档[Sharing Code Snippets](http://octopress.org/docs/blogging/code/)。

Octopress是一款为hacker量身定制的博客系统，当然内置了代码高亮的功能！它的代码高亮功能是通过Pygments实现的，配色方案用的是Solarized，堪称完美。

Octopress支持多种方式嵌入代码，可以直接嵌入代码，也可以引用github上的gist 。

我喜欢用**三个反引号**直接嵌入代码，比 `codeblock`要简洁。

###启用MathJax
在`source/_includes/custom/footer.html`的第一行加入如下代码：
```
<!-- mathjax config similar to math.stackexchange -->
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  jax: ["input/TeX", "output/HTML-CSS"],
  tex2jax: {
    inlineMath: [ ['$', '$'] ],
    displayMath: [ ['$$', '$$']],
    processEscapes: true,
    skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
  },
  messageStyle: "none",
  "HTML-CSS": { preferredFont: "TeX", availableFonts: ["STIX","TeX"] }
});
</script>
<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML" type="text/javascript"></script>
```
这样就引入了MathJax的JS包，可以直接在markdown文件里直接写公式了，例如 $\dfrac {\pi}{2}$。

上面的代码也可以在head.html里添加，不过这样会使得页面的加载速度变慢。

<!--more-->

本节参考了[Writing Math Equations on Octopress](http://www.idryman.org/blog/2012/03/10/writing-math-equations-on-octopress/)，不过省去了安装kramdown的步骤，因为引入了MathJax的JS后，就可以直接写公式了，可以看看[edchen博客的_config.yml](https://github.com/echen/echen.github.com/blob/source/_config.yml)，依然用的rdiscount，再看看它的网页源码，引用了MathJax的JS。

**右击公式全屏空白**：这时候右击公式，全屏空白。解决这个问题很简单，参考[在Octopress中使用Latex公式](http://jasonllinux.github.com/blog/2012/11/06/write-latex-in-octopress/)，只需在 `sass/base/_theme.scss`添加"#main"即可：
```
body {
  > div#main {
    background: $sidebar-bg $noise-bg;
```
看blog.echen.me的[改动](https://github.com/echen/echen.github.com/commit/e0f9b550e564c39239e2dbe10ce8d20e2b1102e8#sass/base/_layout.scss)，两处都改为了 div#main，暂时不知道为什么，不过我也两处都改。

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

## 添加about me 边栏
编辑 source\_includes\custom\asides\about.html，内容如下：
```
<section>
  <h1>About Me</h1>
  <p>一句话自我介绍.</p>
  <p>新浪微博: <a href="http://weibo.com/soulmachine">@soulmachine</a><br/>
     Twitter: <a href="https://twitter.com/#!/soulmachine">@soulmachine</a><br/>
     Other: <a href="https://github.com/soulmachine">Github</a>, <a href="https://plus.google.com/103519507226474510310">Google+</a>, <a href="http://www.linkedin.com/in/soulmachine">LinkedIn</a>, <a href="http://www.quora.com/Jason-Day-2">Quora</a></p>
  </p>
</section>
```
在 _config.yml 的 default_asides 里添加 custom/asides/about.html。

##添加about页面
```
rake new_page[about]
```
会生成 source/about/index.markdown 文件。

编辑该文件的内容。

然后在头部导航菜单中添加页面的超链接。具体做法是编辑 /source/_includes/custom/navigation.html 文件。

## 社会化分享
启用 twitter 分享， facebook like 和Google +1，设置如下：
```
google_plus_one: true
twitter_tweet_button: true
facebook_like: true
```

**添加新浪微博分享**  
参考这篇博客 [为Octopress追加[分享到微博]按钮](http://programus.github.com/blog/2012/03/04/share-weibo-button/)。

在`source/_includes/post/sharing.html`中，加入代码：
```
{% if site.weibo_share %}
  <iframe 
    allowTransparency="true"
	frameborder="0"
	scrolling="no"
    width="72" 
    height="16" 
    src=
      "http://hits.sinajs.cn/A1/weiboshare.html?url={{ site.url }}{{ page.url }}&amp;type=3&amp;count=1&amp;{% if site.weibo_uid %}ralateUid={{ site.weibo_uid }}&amp;{% endif %}language=zh_cn">
  </iframe>
  {% endif %}
```
同时要在_config.yml文件中加入weibo_share 字段，设置其值为true。

推荐使用jiathis.com 的分享按钮，集成了国内主流的网站，非常方便。获取JS代码后，替换掉上面代码即可。这时，可以在_config.yml 中，将twitter, google+ 和facebook like的按钮设置为false，取消显示，因为JiaThis已经集成了这三者，在本文底部可以看到效果。

## 社会化评论
<del>启用Disqus，填入 short name即可。</del>Disqus在国外流行，在国内的加载速度太慢，而且只有twitter, facebook, g+，没有照顾到国内的用户习惯，因此替换成国内的[多说](www.duoshuo.com)。参考这篇博客 [为 Octopress 添加多说评论系统](http://ihavanna.org/Internet/2013-02/add-duoshuo-commemt-system-into-octopress.html)。不过配置略有不同：
```
duoshuo_comments: true
duoshuo_short_name: yanjiuyanjiu
duoshuo_asides_num: 5      # 侧边栏评论显示条目数
duoshuo_asides_avatars: 1   # 侧边栏评论是否显示头像
duoshuo_asides_time: 1      # 侧边栏评论是否显示时间
duoshuo_asides_title: 1     # 侧边栏评论是否显示标题
duoshuo_asides_admin: 0     # 侧边栏评论是否显示作者评论
duoshuo_asides_length: 32   # 侧边栏评论截取的长度
```

## 设置固定链接
在 _config.yml 里，找到 permalink，设置如下：
```
permalink: /blog/:year:month:day/ 
```
效果就是`www.example.com/blog/20130403/`。模仿的是豆瓣的URL格式。

参考官方文档[jekyll Permalinks](https://github.com/mojombo/jekyll/wiki/Permalinks)。

## 侧边栏显示分类目录
使用第三方插件 [octopress-tagcloud](https://github.com/tokkonopapa/octopress-tagcloud)。

##友情链接
在`source\_includes\custom\asides` 目录下添加一个blogroll.html文件，模仿about.html，添加一些友情链接，例如：
```
<section>
  <h1>友情链接</h1>
  <ul>
    <li>
      <a href="http://coolshell.cn/">酷壳CoolShell</a>
    </li>
    <li>
      <a href="http://mindhacks.cn/">刘未鹏MIND HACKS</a>
    </li>
    <li>
      <a href="http://blog.codingnow.com/">云风</a>
    </li>
    <li>
      <a href="http://www.cnblogs.com/Solstice/">陈硕</a>
    </li>
  </ul>
</section>
```
然后在 \_config.yml 文件中，在 default_asides 的数组中添加 `custom/asides/blogroll.html`。

##中文目录
TODO

##修改字体
Octopresss默认使用的是 google webfonts，见`source/_includes/custom/head.html`里的两行代码。Google Webfonts是个好东西，但遗憾的是它没有中文字体。所以你用**粗体**，发现并没有变粗，就是这个原因。

首先，将head.html中的两行代码注释掉，省去了加载字体，加快网页加载速度。
```
<!--Fonts from Google"s Web font directory at http://google.com/webfonts -->
<!-- <link href="http://fonts.googleapis.com/css?family=PT+Serif:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css"> -->
<!-- <link href="http://fonts.googleapis.com/css?family=PT+Sans:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css"> -->

```
参考 这篇博客 [最佳 Web 中文默认字体](http://lifesinger.wordpress.com/2011/04/06/best-web-default-fonts/)，在`sass/custom/_fonts.scss`中添加如下三行代码：
```
$heading-font-family: arial, sans-serif;
$header-title-font-family: arial, sans-serif;
$header-subtitle-font-family: arial, sans-serif;
```
刷新网页，可以看见中文的粗体变黑了。

##一些汉化工作
在 _config.yml中，把 `Read on` 改为 "继续阅读"。
在 `source/_includes/custom/asides`目录下，将"Recent Comments"改为“最新评论”，"Categories"改为“分类目录”，将`source/_includes/asides/recent_posts.html`中"Recent Posts"改为“最新文章”。

##修改顶部Header
类似这样 http://xueran.github.com/blog/2012/12/26/blog-theme2/

## 添加统计代码
填入 Google Analytics Tracking ID，例如 `UA-7583537-4`。

##第三方主题和插件
主题：[3rd Party Octopress Themes](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)  
插件：[3rd party plugins](https://github.com/imathis/octopress/wiki/3rd-party-plugins)

##参考资料
1. [Octopress主题改造](http://shanewfx.github.com/blog/2012/08/13/improve-blog-theme/)
1. 