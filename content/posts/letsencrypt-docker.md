+++
title = "免费申请域名证书"
date = 2021-06-17T10:23:58+08:00
images = []
tags = ["Letsencrypt", "certbot", "docker", "nginx"]
categories = ["编程"]
draft = false
+++

网络安全越来越重要，现在的浏览器，只要不是https访问，地址栏就会提示不安全。我们幸苦做了个网站，当然不希望被浏览器标记为不安全。网站要切换到https访问，首先需要有域名证书。购买域名证书并不便宜。好在Letsencrypt为我们提供了免费的域名证书申请。

申请域名证书，首先需要我们证明对域名的控制权。Letsencrypt用软件certbot帮助我们完成对证书的申请。我们把域名解析到我们的服务器，再在这个服务器上运行certbot软件，以证明我们对域名，服务器的控制权，然后获得证书。

软件certbot提供了docker版本，下面我们会使用docker来运行certbot软件。

软件certbot验证我们对域名和服务器的控制权，原理是这样的，我们先用域名和服务器建一个网站，certbot会在这个网站的特殊子目录下创建一个文件，Letsencrypt会通过域名地址去访问这个文件，通过验证这个文件，来证明我们对域名和服务器的控制权。

为了在服务器上建立一个可以访问的简单网页，我们下面使用nginx来实现。nginx也用docker来运行。

## 准备docker镜像

```
docker pull nginx
docker pull certbot/certbot
```

## 准备文件

我们先准备几个文件和文件夹，目录结构如下

```
/root/volumes/nginx/nginx.conf
/root/volumes/nginx/html
/root/volumes/nginx/html/index.html
/root/volumes/letsencrypt/etc/letsencrypt
/root/volumes/letsencrypt/var/lib/letsencrypt
/root/volumes/letsencrypt/var/log/letsencrypt
```

其中/root/volumes/nginx/nginx.conf是nginx的配置文件。

```
server {
    listen       80;
    server_name  ooxx.com;

    location ~ /.well-known/acme-challenge {
        allow all;
        root   /usr/share/nginx/html;
    }

    root /usr/share/nginx/html;
    index index.html;
}
```

其中/root/volumes/nginx/html/index.html，是一个简单网页。

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>Let's Encrypt First Time Cert Issue Site</title>
</head>
<body>
    <h1>Oh, hai there!</h1>
    <p>
        This is the temporary site that will only be used for the very first time SSL certificates are issued by Let's Encrypt's
        certbot.
    </p>
</body>
</html>
```

其他是一些文件夹，在启动docker容器的时候会用到。

## 申请证书

用docker启动nginx，让我们的简单网页可以通过域名访问。

```
docker run --network host --rm --name nginx-letsencrypt \
-v /root/volumes/nginx/nginx.conf:/etc/nginx/conf.d/default.conf \
-v /root/volumes/nginx/html:/usr/share/nginx/html \
-d nginx
```

如果一切正常，访问http://ooxxooxx.com/ 就可以看到一个网页了。其中ooxxooxx.com是我们的域名。

申请证书有次数限制，所以我们最好在正式申请前，测试一下命令是否正确。下面这条命令中，加--staging参数是测试，去掉--staging参数是正式申请。

```
docker run -it --rm \
-v /root/volumes/letsencrypt/etc/letsencrypt:/etc/letsencrypt \
-v /root/volumes/letsencrypt/var/lib/letsencrypt:/var/lib/letsencrypt \
-v /root/volumes/letsencrypt/var/log/letsencrypt:/var/log/letsencrypt \
-v /root/volumes/nginx/html:/data/letsencrypt \
certbot/certbot \
certonly --webroot \
--register-unsafely-without-email --agree-tos \
--webroot-path=/data/letsencrypt \
--staging \
-d ooxxooxx.com
```

如果上面的命令运行后没用报错，那么我们再运行正式的证书申请命令。

```
docker run -it --rm \
-v /root/volumes/letsencrypt/etc/letsencrypt:/etc/letsencrypt \
-v /root/volumes/letsencrypt/var/lib/letsencrypt:/var/lib/letsencrypt \
-v /root/volumes/letsencrypt/var/log/letsencrypt:/var/log/letsencrypt \
-v /root/volumes/nginx/html:/data/letsencrypt \
certbot/certbot \
certonly --webroot \
--register-unsafely-without-email --agree-tos \
--webroot-path=/data/letsencrypt \
-d ooxxooxx.com
```

正式申请证书后，查看目录/root/volumes/letsencrypt/etc/letsencrypt多出一个文件夹。里面保存了我们的证书。

## 使用证书

新建一个nginx的配置文件/root/volumes/nginx/https-nginx.conf，它将使用我们申请的证书，并切换到https访问。

```
server {
    listen       443;
    server_name  ooxx.com;

    ssl on;

    ssl_certificate /etc/letsencrypt/live/ooxx.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ooxx.com/privkey.pem; 

    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

重新运行nginx。

```
# 关闭前面申请证书时运行的nginx
docker stop nginx-letsencrypt

# 重新运行使用证书的nginx
docker run --network host --rm --name nginx-https \
-v /root/volumes/nginx/https-nginx.conf:/etc/nginx/conf.d/default.conf \
-v /root/volumes/letsencrypt/etc/letsencrypt:/etc/letsencrypt \
-v /root/volumes/nginx/html:/usr/share/nginx/html \
-d nginx
```

如果一切正常，我们访问https://ooxx.com/，浏览器地址栏就会有一个绿色的锁，已经成功切换到了https访问。

## 更新证书

我们申请的证书三个月后会过期，所以需要每三个月更新依次。同样的，我们先运行申请证书的nginx，再运行certbot的更新证书命令。

```
docker run --network host --rm --name nginx-letsencrypt \
-v /root/volumes/nginx/nginx.conf:/etc/nginx/conf.d/default.conf \
-v /root/volumes/nginx/html:/usr/share/nginx/html \
-d nginx

docker run --rm -it --name certbot \
-v /root/volumes/letsencrypt/etc/letsencrypt:/etc/letsencrypt \
-v /root/volumes/letsencrypt/var/lib/letsencrypt:/var/lib/letsencrypt \
-v /root/volumes/letsencrypt/var/log/letsencrypt:/var/log/letsencrypt \
-v /root/volumes/nginx/html:/data/letsencrypt \
certbot/certbot renew --webroot -w /data/letsencrypt
```

## 参考

- [参考1](https://www.humankode.com/ssl/how-to-set-up-free-ssl-certificates-from-lets-encrypt-using-docker-and-nginx)