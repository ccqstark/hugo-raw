---
title: "使用Hugo+github/gitee搭建个人博客"
date: 2020-10-04T02:44:33+08:00
draft: true
image: "hugo.png"
tags:
    - Hugo
categories:
    - 其它技术
---

### 基本操作

#### 下载hugo

首先要有Golang的环境

然后在GitHub上选择对应平台下载

https://github.com/gohugoio/hugo/releases

Windows下载完要设置环境变量


#### 创建新的站点

`hugo new site <path> `

在指定路径下创建博客站点目录，目录最后是博客站点名


#### 找到心仪主题

在下面这个网站上找到喜欢的主题，按照各自的文档进行设置

https://themes.gohugo.io/


#### 本地预览

```shell
hugo server -t <theme> --buildDrafts
```
\<theme\>的位置填写主题的名称



#### 创建博客

```shell
hugo new post/blog.md
```

博客的markdown文件一开始都是放在`\content\post`目录下


#### 创建GitHub/Gitee仓库

Github把仓库命名为`<name>.github.io`即可开启博客托管服务

Gitee直接命名为自己的用户名，一字不差，同时需要手动开启Gitee Page服务


#### 部署到远端仓库

* 生成`\public`目录

```shell
hugo --theme=hugo-theme-stack --baseUrl="https://ccqstark.github.io/" --buildDrafts
```

根据具体仓库修改，也可以是`"https://ccqstark.gitee.io/"`
然后cd进`public`目录
在这个目录下创建git仓库，部署也是部署这个目录中的内容

* 三部曲

```shell
git add .
git commit -m '...'
git push github master
```


#### 更新博客

要新增一篇博客就继续按下面这个步骤走

```shell
hugo new post/name.md
hugo --theme=hugo-theme-stack --baseUrl="https://ccqstark.github.io/" --buildDrafts
cd public
git add .
git commit -m '...'
git push github master
```

如果只是更新就重新运行生成`\public`目录的命令，再重新部署即可


### 主题Stack配置

此博客的主题用的是`Stack`，下面是主题地址，可以找到文档和Demo

https://themes.gohugo.io/hugo-theme-stack/


#### icon设置

使用`.ico`文件类型的图标，并将其放置在`\public`目录下

`\layouts\partials\head`下新建一个`custom.html`，写上

```shell
<link rel="shortcut icon" href="https://ccqstark.github.io/icon.ico"/>
```

为了访问稳定也可以改为gitee


#### 文章外部展示图

在`\content\post`目录下创建博客时，先创建博客标题命名的文件夹，再在其下创建md文件`index.md`

图片也是放在其下，在md文件开头的设置中添加`image: "image.jpg"`即可


#### Tags和分类

文章开头设置

```
tags:
    - Spring
categories:
    - Tech
```

一个是打标签，一个是分类


#### About和Archives

在`\content\page`目录下创建`about.md`和`archives.md`

`about.md`用于写个人信息页面，需要文章顶部的设置写法改为字段加冒号的形式，配置`slug`为`about`

`archives.md`用于分类和归档页面，用exampleSite的就行，不用做任何修改


#### Categories

在`\content\categories`目录下创建以各分类命名的文件夹

各分类的文件夹下放置`_index.md`和分类的展示图

`_index.md`中的内容就是普通文章顶部的配置项，注意图片名配置对了就行





