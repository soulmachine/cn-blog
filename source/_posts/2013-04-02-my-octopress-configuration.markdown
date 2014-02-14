---
layout: post
title: "我的Octopress配置"
date: 2013-04-02 15:35
comments: true
categories: Tools
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

``` javascript
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

上面的代码也可以在`source/_includes/custom/header.html`里添加，不过这样会使得页面的加载速度变慢。还可以在`source/_layouts/default.html`里添加。

<!--more-->

有一个问题，rdiscount这个解析器，对 mathjax 大部分支持，某些细节处理的不好，举个例子，它会在动把公式中的 `^n`转换成`<sup>n</sup>`，例如`$2^n$`会解析成`$2<sup>n</sup>$`，这样就破坏了整个公式，导致公式无法解析。参考[这里](http://christopherpoole.github.io/using-mathjax-on-github-pages/)一段话：
> as discount for example automatically replaces `x^2` with `x<sup>2</sup>` which interrupts the MathJax rendering.

因此要换一个解析器，[Maruku](http://maruku.rubyforge.org/) 和 [Kramdown](http://kramdown.rubyforge.org/) 都可以，由于Maruku主页PR=4，Kramdown的主页PR=5，我选择了Kramdown。

**用Kramdown代替Rdiscount**  
修改Gemfile，增加一行：

```
gem 'kramdown', '~> 0.14'
```
很多博客都说要配套安装coderay这个gem，其实是没有必要的，只要代码块以 \`\`\` 开始和结束，自带的pygments就能实现代码高亮。

在Git Bash输入如下命令：

``` bash
bundle install
```
就会自动安装kramdown。

然后在\_config.yml 文件中，见markdown: rdiscount 修改为  markdown: kramdown。

使用kramdown，感觉它的语法要求比rdiscout严格，例如每个代码块开头，必须有一个空行，否则高亮就会失败，大家可以试试看。每个标题掐面，也必须有一个开头。

kramdown的两种公式，display和inline，都是以`$$`开头和结尾的，display模式时，`$$`要单独占一行。这跟标准的LaTex有点不一样。参考[这里](http://kqueue.org/blog/2012/01/05/hello-world/)。

**右击公式全屏空白**：这时候右击公式，全屏空白。解决这个问题很简单，只需在 `sass/base/_theme.scss`添加"#main"即可：

```
body {
  > div#main {
    background: $sidebar-bg $noise-bg;
```

本节参考了[Writing Math Equations on Octopress](http://www.idryman.org/blog/2012/03/10/writing-math-equations-on-octopress/) 和 [在Octopress中使用Latex公式](http://jasonllinux.github.com/blog/2012/11/06/write-latex-in-octopress/)。

##kramdown的扩展语法
kramdown扩展了标准markdown的语法，有很多使用的功能。[语法见官网文档](http://kramdown.rubyforge.org/syntax.html)。这里选一些我常用的。

**脚注(footnote)**  
脚注定义是：`[^1]:`，数字可以改变，引用语法是`[^1]`。没有被引用到的参考文献，会被忽略掉。

**表格**  
一下是一个示例：

	|-----------------+------------+-----------------+----------------|
	| Default aligned |Left aligned| Center aligned  | Right aligned  |
	|-----------------|:-----------|:---------------:|---------------:|
	| First body part |Second cell | Third cell      | fourth cell    |
	| Second line     |foo         | **strong**      | baz            |
	| Third line      |quux        | baz             | bar            |
	|-----------------+------------+-----------------+----------------|

更详细说明见官网。

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
使用addthis.com的分享按钮，在网站上获取代码，粘贴到`source/_includes/post/sharing.html`中，例如我的代码如下：

``` html
<div class="sharing">
  <!-- AddThis Button BEGIN -->
  <div class="addthis_toolbox addthis_default_style addthis_32x32_style">
    <a class="addthis_button_sinaweibo"></a>
    <a class="addthis_button_facebook"></a>
    <a class="addthis_button_twitter"></a>
    <a class="addthis_button_google_plusone_share"></a>
    <a class="addthis_button_delicious"></a>
    <a class="addthis_button_digg"></a>
    <a class="addthis_button_reddit"></a>
    <a class="addthis_button_compact"></a><a class="addthis_counter addthis_bubble_style"></a>
  </div>
  <script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=undefined"></script>
  <!-- AddThis Button END -->
```

在_config.yml 中，将twitter, google+ 和facebook like的按钮设置为false，取消显示，因为 AddThis 已经集成了这三者。

## 社会化评论
<del>启用Disqus，填入 short name即可。</del>Disqus在国外流行，在国内的加载速度太慢，而且只有twitter, facebook, g+，没有照顾到国内的用户习惯，因此替换成国内的[多说](www.duoshuo.com)。参考这篇博客 [为 Octopress 添加多说评论系统](http://ihavanna.org/Internet/2013-02/add-duoshuo-commemt-system-into-octopress.html)。`source/_includes/post/duoshuo_thread.html`的代码略有不同，添加了`data-title="{{ page.title }}"`，否则侧边栏的最近评论，标题为空白，感谢[碟姐 - 在octopress中添加多说的最近评论](http://yrzhll.com/blog/2012/12/12/comment/)指出了这一点，代码如下：

``` javascript
<!-- Duoshuo Comment BEGIN -->
<div class="ds-thread" data-title="{{ page.title }}"></div>
<script type="text/javascript">
var duoshuoQuery = {short_name:"yanjiuyanjiu"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = 'http://static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0] 
		|| document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
</script>
<!-- Duoshuo Comment END -->
```

_config.yml 中的配置也略有不同： 

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

## 添加统计代码
在_config.yml填入 Google Analytics Tracking ID，例如 `UA-7583537-4`。

##第三方主题和插件
主题：[3rd Party Octopress Themes](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)  
插件：[3rd party plugins](https://github.com/imathis/octopress/wiki/3rd-party-plugins)

##在一台新电脑上恢复
如果换了一台电脑，怎样迅速恢复环境呢？参考 [Clone Your Octopress to Blog From Two Places](http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/)。 **注意，在windows上，要首先安装python，否则，虽然所有操作可以成功，不报错误，但是你发现打开后首页一篇空白，我当时百思不得其解，因为没有任何错误信息，最后去看生成的文件，所有index.html都是0字节，就猜测应该是编译出了问题。安装python就好了，linux默认是有Python的，就没有这个问题，windows真是坑爹！以后只在windows下做编辑类的工作，编译和运行都放到Linux下。**

##TODO
修改字体大小

添加TAG支持

##参考资料
1. [Octopress主题改造](http://shanewfx.github.com/blog/2012/08/13/improve-blog-theme/)
1. 
