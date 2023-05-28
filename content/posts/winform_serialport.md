<!--
+++
title = "winform里面使用串口"
date = 2023-05-09T09:54:42Z
images = []
tags = ["winform", "串口", "断帧"]
categories = []
draft = true
+++
-->

## 前言

记录在winform中使用串口通信遇到的坑，以及解决方式。

## 串口基本

在winform里面使用串口还是比较简单的，注册DataReceived就可以了。

```
public Form1()
        {
            mSerialPortReader = new SerialPortReader(this, serialPort1);
            serialPort1.DataReceived += new SerialDataReceivedEventHandler(OnSerialDataReceived);
        }

private void OnSerialDataReceived(object sender, SerialDataReceivedEventArgs e)
        {
            System.Diagnostics.Debug.WriteLine("OnSerialDataReceived");
            int count = serialPort1.BytesToRead;
        
            if(count>0){
                byte[] buffer = new byte[count];
                count = serialPort1.Read(buffer, 0, count);
            }
        }
```

## 超时断帧

在用串口传输数据的时候，断帧是个问题，数据从什么时候开始，什么时候结束的。如果使用串口打印debug信息，可以使用回车来换行，因为传输的主要是可显示的ascii码。如果传输的是数据呢，从0-255都是有可能出现的有效数据，不可能专门挑一个数字来作为帧的开头和结尾。而不断帧的话，又根本无法解析数据。

首先想到的是，我们可以使用超时来断帧。超过一定时间，就是新一帧数据的开始。

```
public Form1()
        {
            mSerialPortReader = new SerialPortReader(this, serialPort1);
            serialPort1.DataReceived += new SerialDataReceivedEventHandler(OnSerialDataReceived);
            serialPort1.ReceivedBytesThreshold = 32;
        }

private void OnSerialDataReceived(object sender, SerialDataReceivedEventArgs e)
        {
            long lastTick = 0;
            System.Diagnostics.Debug.WriteLine("OnSerialDataReceived");
            int count = serialPort1.BytesToRead;
        
            if(count>0){
                byte[] buffer = new byte[count];
                count = serialPort1.Read(buffer, 0, count);
            }

            if(System.DateTime.Now.Ticks-lastTick>100000){
                lastTick = System.DateTime.Now.Ticks;
                // 新一帧的开头
            }
        }
```

结果发现这样行不通。因为，**在winform里面，DataReceived不是实时的被调用，什么时候被调用也说不清楚。**

同时，如果你使用的是USB转串口话，USB转串口的过程中，也会有延时和缓存的问题。也会导致我们的延时断帧不成功。

有说可以设置ReceivedBytesThreshold，当数据接收超过ReceivedBytesThreshold的时候，DataReceived会被调用。但我们不能使用这个机制来断帧，因为我们不能保证开始接收数据的时间刚好是一帧的开始。如果我们是从一帧数据的中间开始接收的数据，那么等到接收数据超过ReceivedBytesThreshold作为一帧，那是会出错的。另外，如果中间偶尔出现数据接收错误，也会导致帧对不齐。

## 帧头帧尾

看来，还是要使用特殊的字符来断帧。那前面不是说会和数据会产生混淆吗？我们可以把数据转换成ascii字符，然后使用json数据格式来进行传输，这样就不存在断帧的问题了。我也可以使用大括弧，回车符等特殊字符来作为帧头或帧尾的标志。

这样一来，有个明显的缺点是，数据量增加了。本来只有1byte的数据，可能需要好几个byte的字符来传输。但这也是没有办法的办法了。

## 总结

因为，不熟悉winform中串口数据的接收方式，而使用超时断帧失败。最后还是采用把数据转为字符，用特殊字符来断帧。虽然增加了传输的数据量，但这也是没有办法的办法。
