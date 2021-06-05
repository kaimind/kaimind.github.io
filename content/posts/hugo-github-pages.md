+++
title = "用hugo在github上建立个人博客"
date = 2021-03-29T16:28:28+08:00
images = []
tags = ["hugo", "markdown"]
categories = ["编程"]
draft = false
+++

hugo是一个工具，可以轻松的把markdown文章转换成静态网站。我们先用markdown写文章，再用hugo把markdown转成html，再推送到github的pages，一个博客就轻松建成了。

hugo的安装和使用参考[这里](https://gohugo.io/getting-started/quick-start/)，可以查看hugo的文档。

hugo有很多主题可以选用，比如这个[kiera](https://themes.gohugo.io/hugo-kiera/)。主题的使用，可以参考exampleSite目录下的示例。

## 配置build目录

hugo默认把生成的html文件放到public目录下，我们可以在config.toml添加如下配置

```
publishDir = "docs"
```

那么hugo会把html文件生成到docs目录。你会发现这很有用。因为github pages使用根目录或者docs目录下的html，如果hugo生成的html不在这里，那么我们需要去手动拷贝。现在我们把hugo生成目录设为docs目录，再到github pages里面把默认目录也设为docs目录，那么就省去了拷贝的动作。