---
layout: post
title: jekyll 搭建自己的 blog
date: 2017-12-06 00:00:00 +0300
description: 介绍怎么用 jekyll 搭建自己的 blog
img:
---

我的 blog 是用 jekyll 搭建成的，然后使用 GitHub 的免费主机。我这篇博客就介绍怎样搭建免费的 blog，给想自己搭建 blog 的小伙伴，提供一些帮助。

## 目录
- 介绍 jekyll
- 介绍 GitHub 的 Github Pages
- 用 jekyll 搭建 blog
- 将 jekyll 部署到 Github Pages 上

## 介绍 jekyll
jekyll 是一个简单的免费的 Blog 生成工具，将纯文本转化为静态网站和博客，不需要数据库、评论功能，不需要不断的更新版本——只用关心你的博客内容。最关键的是 jekyll 可以免费部署在 Github 上，而且可以绑定自己的域名。这里有<a href="https://jekyllrb.com">jekyll官网</a>，还有<a href="http://jekyll.com.cn">中文文档</a>。
#### 安装 jekyll

安装 jekyll 之前，确保自己电脑上已经装了 ruby，如果没有装，可以 Google 一下，每个系统的安装方法都有，我在这就不再介绍了。

有了 ruby 安装 jekyll 就很简单:

```ruby
~$ gem install jekyll
```
等几分钟，就安装成功。关于怎么使用 jekyll ，我会在后面介绍。

## 介绍 GitHub 的 Github Pages

Github Pages 是面向用户、组织和项目开放的公共静态页面搭建托管服务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默认提供的域名 github.io 或者自定义域名来发布站点。Github Pages 支持自动利用 Jekyll 生成站点，也同样支持纯 HTML 文档。<a href="https://pages.github.com">Github Pages使用方法</a>，当然使用 Github 是要科学上网的，所以托管在 Github 的blog也要科学上网才能访问。

## 用 jekyll 搭建 blog

### 获取最简单 Jekyll 模板并生成静态页面

在你想建blog的本地路径，输入以下命令：

```ruby
~ $ jekyll new myblog
```
等待一会，jekyll site 就安装成功，然后运行

```ruby
~ $ cd myblog
~ $ jekyll serve
```
打开浏览器，输入“http://127.0.0.1:4000/”，会看见你建的站点，内容是 Jekyll 的默认模版，从现在开始，你可以通过创建文章、改变头信息来控制模板和输出、修改 Jekyll 设置来使你的站点变得更有趣。

### 目录结构

一个基本的 Jekyll 网站的目录结构一般是像这样的：

```ruby
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.markdown
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _site
└── index.html
```
##### _config.yml 
- 保存配置数据，比如baseurl、name都可以放这，统一设置。很多配置选项都会直接从命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。

##### _drafts
- drafts 是未发布的文章。这些文件的格式中都没有 title.MARKUP 数据

##### _includes 
- 你可以加载这些包含部分到你的布局或者文章中以方便重用。

##### _layouts 
- layouts 是包裹在文章外部的模板。布局可以在 YAML 头信息中根据不同文章进行选择。

##### _posts
- 这里放的就是你的文章了。文件格式很重要，必须要符合: YEAR-MONTH-DAY-title.MARKUP。

##### _site
- 一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 .gitignore 文件中。

##### index.html
- 如果这些文件中包含 YAML 头信息部分，Jekyll 就会自动将它们进行转换。

像任何网站的配置一样，需要按约定在站点的要目录下找到 index.html 文件， 这个文件将被做为主页显示出来。除非你的站点设置了其它的文件作为默认文件， 这个文件就将是你的 Jekyll 生成站点的主页。将 HTML 文件放在哪里取决于你想让它们如何工作。有两种方式可以创建页面：

1. 命名 HTML 文件：将命名好的为页面准备的 HTML 文件放在站点的根目录下。
2. 命名文件夹：在站点的根目录下为每一个页面创建一个文件夹，并把 index.html 文件放在每 个文件夹里。

这两种方法都可以工作（并且可以混合使用），它们唯一的区别就是访问的 URL 样式不同。

```ruby
.
|-- _config.yml
|-- _includes/
|-- _layouts/
|-- _posts/
|-- _site/
|-- about.html    # => "http://example.com/about.html"
|-- index.html    # => "http://example.com/"
└── contact.html  # => "http://example.com/contact.html"
```


上面介绍的目录，有些在刚才建的blog里是没有的，因为这个blog是最简单的模版，功能很简单，所以有些目录是不包含的，可以看这个<a href="http://jekyll.com.cn/docs/home/">文档</a>，有详细的介绍。

### 模版主题

现在我们就可以看着文档，自由的构建自己的blog了，但是我们想实现一些界面丰富、功能强大的blog，需要我们具备一定的前端能力，所以模版主题应运而生，<a href="http://jekyllthemes.org">jekyllthemes</a>，这个网站有大量的模版，可以找自己喜欢的，然后在这基础上，修改完善自己的blog。

## 将 jekyll 部署到 Github Pages 上

最后，就是把我们写好的jekyll，部署到 Github Pages 上。

### 第一步

需要你有 Github 的账号，创建一个仓库，username 换成你的账户名。

![](http://p0iv8hbe9.bkt.clouddn.com/createRepository.png)

### 第二步

把仓库 clone 到本地

```ruby
~ $ git clone https://github.com/username/username.github.io
```
### 第三步

进到自己本地的 clone 路径，把我们写好的 jekyll ，复制到这个路径下，或者提前建好仓库，就在这个本地仓库里写 jekyll 。然后运行 jekyll 
```ruby
~ $ jekyll serve
```
在"http://localhost:4000/"，看效果，满意之后，把代码推到 Github 仓库。

```ruby
~ $ git add -all
~ $ git commit -m "Initial commit"
~ $ git push -u origin master
```
最后在浏览器上输入："https://username.github.io"(username换成自己的账号名)，就能看到自己的blog，而且别人也可以访问。就是这么简单。

## 总结

用 jekyll 搭建成的blog，然后托管在 GitHub ，简单，方便，不用和主机服务商打交道，就能完成界面丰富、功能强大的blog。小伙伴们快来搭建自己的blog吧。


