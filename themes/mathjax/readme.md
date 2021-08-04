# 让theme支持数学公式

1. 拷贝mathjax.html到themes/kiera/layouts/partials目录

2. 文件themes/kiera/layouts/partials/header.html添加mathjax.html部分

```
<head>
  ......
  {{ partial "mathjax.html" . }}
</head>
```

3. 在有数学公式的markdown文件添加math变量

```
+++
title = "理解线性代数"
date = 2021-08-03T20:38:16+08:00
images = []
tags = ["math"]
categories = ["编程"]
draft = false
math = true
+++
```

4. 数学公式不换行，双反斜杠，换为四个反斜杠