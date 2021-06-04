+++
title = "把图片直接插入到markdown文件中"
date = 2021-06-04T17:35:33+08:00
images = []
tags = ["markdown"]
categories = []
draft = false
+++

在markdown中插入图片很简单，只需要添加图片的外部链接或路径就可以了。但这会有问题，比如，外部链接的图片消失的时候，那么我们的markdown文件也就无法正常显示图片了。再比如，当我们要拷贝一个markdown文件的时候，肯定会容易忘记去拷贝相应路径下的图片，这也会导致图片无法显示。这都是因为图片独立于markdown文件，而带来的问题。

有没用办法把图片直接插入到markdown文件中？方法是有的。我们可以把图片编码成base64代码，再插入到markdown文件中。

在markdown文件中，需要插入图片的位置：

```
![pic][pic3355]
```

在markdown文件的末尾，插入图片的base64编码：

```
[pic3355]:data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA9AAAADNCAYA
```

其中pic3355是我们插入图片的文件名。

其中“iVBORw0K...”这段看不懂的代码，是插入图片的base64编码。简单来说，就是把图片的二进制文件，用文本ascii码字符来编码。

如何获取图片文件的base64编码呢？我们用下面的python代码来获得。

```
import base64

with open('./pic3355.png','rb') as f:
    bf = base64.b64encode(f.read())

with open('./pic3355b.txt', 'w') as f:
    f.write(str(bf))
```

其中pic3355.png是我们的图片文件及其路径。运行完这段python代码后，就可以到pic3355b.txt文件中拷贝出base64编码，插入到markdown文件了。