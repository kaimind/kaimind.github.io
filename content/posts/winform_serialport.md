+++
title = "winform里面使用串口"
date = 2023-05-09T09:54:42Z
images = []
tags = ["winform", "串口", "断帧"]
categories = []
draft = true
+++

在mcu的嵌入式开发中，可能会遇到与PC的通信。比如，由mcu采集的数据，在PC端上接收，处理和展示。在PC端，比较简单的图形界面开发方式，winform算一个。PC与MCU的通信，比较常用的是串口。毕竟，我们本来就使用串口接收mcu的debug打印信息。