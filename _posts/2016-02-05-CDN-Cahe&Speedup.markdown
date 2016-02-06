---
layout: post
title:  "CDN，缓存和加速"
date:   2016-02-04 13:29:48 -0500
categories: [jekyll]
tags: [jekyll]
---
在 [某人](http://ryuya1995.com/) 的怂恿下，开通了Github Page。今天让其测试，结果发现
只能看到首页。分析了一下链接，发现了几个原因：

1. 因为各种因素，
   他所在的地方无法访问 **Google**，
   而我采用的模板中，
   默认使用 **[Google CDN](http://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js)** 加载
   **[JQuery](https://jquery.com/)**，遭遇 *connection reset*。
   把地址换成了 **[Microsoft CDN](https://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.11.3.min.js)**，
   结果再一次感受到了墙的威力：
   加载 *15 s*
   无奈换成本地存储

2. 当初选择背景图片 **Cover.jpg** 的时候，图省事，直接从手机里取出一张
   没想到， ios 能把噪点如此多的图片拍成 **4 MB**
   只得压缩图片，顺便去个噪。
   *PS:反正你们也不在意，下次直接盗个外链算了*

3. 模板里面有一套字体了，居然又使用了两个在线字体，
   **[Google Font](https://www.google.com/fonts)** 卒 :dizzy_face:
   又一次感受了墙的威力后，换成本地存储

## 测试
---

全部资源本地存储了之后，完全失去了 **CDN** 的优势:

#### [CDN](https://zh.wikipedia.org/wiki/%E5%85%A7%E5%AE%B9%E5%82%B3%E9%81%9E%E7%B6%B2%E8%B7%AF) (内容分发网络)

属于一种分摊负荷提，高吞吐量的有效方式，有传言(来源不明)

>IE6 只支持同时对一个域名的 2个链接，因此页面内采用多个资源可以有效避免 1-2 个较慢资源对整体网页加载的影响

因此，每次加载页面时，总计会对相同的地址请求 16 次，耗时大约 600 ms, 但有时遇到
网络延时，单次请求超过 400 ms 总体的加载时间会超过 1400 ms


## 修正
---

#### 1.使用CDN和本地混合的加载方式：

修改之前
{% highlight html %}
<script type="text/javascript"
  src="https://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.11.3.min.js">
</script>
{% endhighlight %}

修改之后
{% highlight html %}
<script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
<script>
  window.jQuery||document.write("<script type=\"text/javascript\" src=\"//cdn.bootcss.com/jquery/1.11.3/jquery.min.js\"></script>");
  ||document.write("<script type=\"text/javascript\" src=\"{{ "js/jquery-1.12.0.min.js" | prepend: site.baseurl }}\"></script>");
</script>
{% endhighlight %}

测试发现 Chrome 浏览器的解析器，会把
{% highlight javascript %}
document.write("<script type=\"text/javascript\" src=\"{{ "js/jquery-1.12.0.min.js" | prepend: site.baseurl }}\"></script>");
{% endhighlight %}
中的`</script>`解析为 **script** 标签的结束记号，所以采用转译符
{% highlight html %}
<script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
<script>
  window.jQuery||document.write("<script type=\"text/javascript\" src=\"//cdn.bootcss.com/jquery/1.11.3/jquery.min.js\">\u003C/script>");
  ||document.write("<script type=\"text/javascript\" src=\"{{ "js/jquery-1.12.0.min.js" | prepend: site.baseurl }}\">\u003C/script>");
</script>
{% endhighlight %}

#### 2.压缩：

1. 使用 **[Gimp](https://www.gimp.org/)** 对 **cover.jpg** 和 **profile.jpg**
进行高斯模糊，图像缩放，并去除注释，提高压缩率

2. 对 **CSS** 文件进行压缩，有个在线站点 **[CSS Compressor](http://csscompressor.com/)**
(只能进行单文件压缩)，压缩工具可以 Google 或 Baidu，**minify**， 有个工具 **[minify](https://github.com/mrclay/minify)**
