---
title: "hugo的简单的配置"
date: 2019-05-13T19:30:17+08:00
draft: false
---

hugo是基于Go语言的静态网站生成器。  

声明：下面的内容全部是来自于[kikajack](https://blog.csdn.net/kikajack/article/details/80413052)

## 生成站点  
使用Hugo快速生成站点，比如喜欢生成到/path/to/site路径 

> hugo new sizt /pat/to/size

## 添加内容

在项目目录里面执行 hugo new xx.md命令，会在content目录中创建XX.md文件

> hugo new about.md

这个文件的默认内容如下

```
---
title: "About"
date: 2018-05-22T22:04:26+08:00
draft: true
---
```

文件默认的内容，draft表示是否为草稿，编辑完成之后将其改为false，或者编译会跳过草稿文件

## 启动hugo自带的服务器

在项目目录下，通过hugo server命令来调试预览博客--theme选项可以指定主体，--watch选项可以修改文件后自动刷新浏览器，在后期的版本中已经不支持--watch版本了，--buildDrafts包括标记草稿（draft）的内容。

> hugo server --theme=whiteplain --buildDrafts

## 部署

> hugo --theme=whiteplain --baseUrl="http://www.baggk.xin"

上面的部署的命令并不会生成草稿页面，去掉文件头deaft=true在重新生成。如果一切顺利则会生成到public目录中，里面的文件才是我们需要的HTML的文件，将public目录中的文件push到目的web服务器就好了。