+++
title = "可信任的时间戳"
date = 2021-06-05T10:15:02+08:00
images = []
tags = ["linux"]
categories = ["编程"]
draft = false
+++

时间的证明很重要。比如，电影里面，绑匪让被绑架的肉票拿着当天的报纸录像。这就是为了证明，肉票在报纸上的日期那天还活着。因为绑匪有再大的本事，也没办法拿到几天之后的报纸。再比如，维基解密的阿桑奇，会在录像的时候读出比特币区块链的哈希表，这就证明了录像时间是在产生这个区块链的时间之后。

前面两个例子都是在说，证明事件发生在某个时间之后。那要证明事件发生在某个时间之前呢？考古上用碳14来鉴定，某个文物是几千年前的产物，而不是昨天伪造的东西。发展到了区块链时代，我们可以把一个事情写入到区块链中，来证明它发生的时间点，没有人能篡改。这也是区块链的重要作用之一，解决信任问题。下面，我们用可信任的时间戳来证明事件发生的时间。

我们把某件事情的文本发生到时间服务器，服务器返回给我们一个数字签名。验证这个数字签名的时间戳，就能证明该事件发生的事件点了。

```
openssl ts -query -data inputfile.txt -cert -sha256 -no_nonce \
| curl -k -H "Content-Type: application/timestamp-query" \
-H "Host:timestamp.globalsign.com" --data-binary \
@- "http://timestamp.globalsign.com" \
> inputfile.txt.tsr
```

上面的命令会生成一个inputfile.txt.tsr文件，就是我们需要的时间戳证明文件。其中inputfile.txt是我们的输入文件，也就是需要时间证明的事件。

下面来验证这个时间戳。

```
openssl ts -reply -in inputfile.txt.tsr -text
```

会得到类似下面的信息。

```
Hash Algorithm: sha256
Message data:
    0000 - 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00
    0010 - 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00
Time stamp: Dec 24 03:55:24 2019 GMT
```

它证明了我们的事件inputfile.txt存在于（Time stamp: Dec 24 03:55:24 2019 GMT）这个时间点。其中Message data是我们输入文件inputfile.txt的哈希值。可以用下面的命令，同样可以计算出文件的哈希值。

```
gpg --print-md sha256 inputfile.txt
```

这其中用到了数字签名的概念。数字签名又涉及到哈希算法，和非对称RSA加密算法。