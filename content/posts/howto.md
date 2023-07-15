+++
title = "Howto"
date = 2023-07-15T12:50:42Z
images = []
tags = []
categories = []
draft = true
+++


```
submodule:
git submodule update --init --recursive

docker run -itd --name=myhugo -v path:/root -p 1313:1313 ubuntu 

docker exec -it myhugo /bin/bash

hugo server --bind=0.0.0.0

hugo new posts/howto.md
```