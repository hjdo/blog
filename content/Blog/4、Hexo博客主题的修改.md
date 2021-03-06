---
title: 4、Hexo博客主题的修改
date: 2016-03-06T22:54:10+08:00
tags: ["Hexo博客搭建"]
categories: ["Blog"]
permalink: Hexo-Getting-Started-4
description: Hexo博客搭建
---
　　在这一篇中介绍一下Hexo主题的一些修改方法，在这里是基于默认主题介绍的，但是了解一下，对使用其他主题也有些帮助。

# 1. 主题文件说明

　　依次打开博客文件夹根目录下的`themes\landscape\`，可以看见主题包里面一些文件夹和文件，我们先认识一下这些文件分别是什么：
![](http://ww3.sinaimg.cn/mw690/c55a7aeejw1f1njlex0x6j208508hgli.jpg)

　　1. `languages`：语言文件包，存放网页语言文件。
　　2. `layout`：页面模板文件夹，保存生成静态页面的HTML模板文件，使用的是EJS模板引擎编写，会HTML的话就很容易看懂。
　　3. `scripts`：存放网页中要使用的脚本文件。<!--more-->
　　4. `source`：存放资源文件，比如CSS文件和js文件、图片等静态资源。
　　5. `.gitignore`：git提交时的忽略文件配置信息文件，主题自动生成的，不用管。
　　6. `_config.yml`：主题的配置文件。
　　7. `Gruntfile.js`：这个是前端的代码压缩文件，使用Grunt工具压缩代码，减少页面加载时间。这个是新增的文件，其他主题中可能没有，不用在意。
　　8. `LICENSE`：Git生成的版权许可文件，不用管。
　　9. `package.json`：前端工具包配置文件，配置前端工具等信息，比如刚才的Grunt工具的信息就是在这里配置的。不用管。
　　10. `README.md`：github生成的项目说明文件。不用管。

# 2. 主题修改

　　了解了主题中的文件之后，其实修改主题的话，主要用到的有三个：`_config.yml`、`layout`、`source`。其中`_config.yml`为配置文件不用太多的修改，上一篇中介绍过了。Hexo主题的主要修改的文件就在后两个文件夹中。
　　“layout”文件中主要是网页的HTML模板代码，“source”文件夹中是网页的样式文件(css)和一些动态效果的文件(js)。对前端比较熟悉的同学可以研究一下，自己编写想要的风格和效果，由于我的前端知识不多，就不详细介绍了。

## 2.1. source文件夹

　　由于样式文件夹中的内容比较简单一些，我们先来看看：
　　打开之后又三个文件夹：`css`、`fancybox`、`js`。其中`fancybox`是fancybox插件用到的一些静态文件，不用管它。其他两个就是样式和脚本了。
　　这两个文件夹中的内容可以根据需要随意修改，举个例子，我们打开`css`文件夹，可以看到有一个`images`文件夹，这里面就是CSS用到的背景图片之类的资源，把里面的图片替换掉看看效果：
![](http://ww3.sinaimg.cn/mw690/c55a7aeejw1f1nkrahlcrj212n0m1qc9.jpg)


## 2.2. layout文件夹

　　这个文件夹算是比较重要的，博客页面构成就靠它了。
　　点进去看看能够知道，landscape主题的文件分布比较细致，基本每一个模块都分离成单独的文件了，这样容易看懂，修改也比较方便，赞一个。
　　具体的代码就不介绍了，大家感兴趣可以看看，很简单的，尝试修改然后预览，这样大致就明白里面的代码了。在这里只介绍一下给文章添加一个多说评论、侧边栏增加一个微博秀。

### 2.2.1. 添加多说评论

**申请多说评论账号**
　　大家可以去[多说网站](http://duoshuo.com/)申请一个多说账号，具体的申请就不多介绍了，介绍一下怎么用：
　　进入你添加的博客站点后台，找到工具这一栏，可以看到有获取代码的选项，我们使用通用代码就可以了，复制生成的代码：
![](http://ww1.sinaimg.cn/mw690/c55a7aeejw1f1nl81t9bvj212n0j4jvv.jpg)
　　
**粘贴代码**
　　依次打开`\layout\_partial\article.ejs`，找到文件中的`<footer class="article-footer">`节点，在该节点的前面粘贴刚刚复制的代码。
　　1. 将`data-thread-key`的值替换成`<%= page.path %>"`。
　　2. 将`data-title`的值替换成`<%= page.title %>`。
　　3. 将`data-url`的值替换成`<%= page.permalink %>`。

　　现在保存该文件，刷新一下页面试试，看看评论框是不是出现在了合适的位置：
![](http://ww1.sinaimg.cn/mw690/c55a7aeejw1f1nlprob22j21290l70tl.jpg)

　　**大家可以按照这样的方式，举一反三，给博客添加百度分享等插件。**

### 2.2.2. 添加微博秀侧边栏组件

　　大家可以去微博主页，搜索微博秀就可以看到微博秀的网页应用了，然后就可以获得微博秀的插件代码，具体的获取方式就不多做介绍了 。

**增加微博秀侧边栏模板**
　　依次打开`\layout\_widget`，新建一个名为"weiboshow.ejs"的文件，用编辑器打开，添加下面的代码并保存：

```html
<div class="widget-wrap">
    <h3 class="widget-title">微博秀</h3>
    <div class="widget">
      
      在这里粘贴你的微博秀插件代码
      
    </div>
  </div>
```

**在主题配置中添加微博秀组件**
　　我们已将创建好了微博秀的模板，现在需要把组件添加到主题的配置文件中，让Hexo渲染出来。

　　打开主题文件夹根目录下的`_config.yml`文件，找到`widgets`这个配置节点，在下面添加微博秀模板的名字“weiboshow”比如：

```yaml
...

# Sidebar
sidebar: right   #侧边栏的位置
widgets:        #侧边栏小工具组件
- weiboshow     #微博秀组件     
- category      #分类
- tag           #标签
- tagcloud      #标签云
- archive       #存档
- recent_posts  #最新发布

...
```

　　从这里不难看出，Hexo主题会从配置文件中读取侧边栏挂件信息，去对应的寻找文件进行加载。  值得注意的是，这些名称的顺序会影响挂件在页面的显示顺序，比如我把"weiboshow"写在第一行了，那么在加载页面是，我的微博秀就是侧边栏的最上面。
![]()
