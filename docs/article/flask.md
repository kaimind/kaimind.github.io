# Flask实践

## 最小app

### python环境

```
# 新建文件夹
mkdir HelloFlask
cd HelloFlask

# python虚拟环境
python -m venv venv

# vscode设置python解释器
# ctrl+shift+p
# 输入python:select interpreter
# 找到虚拟环境路径
# 选择script目录下的python.exe

# 在vscod里面下面快捷键，打开vscode的终端
# ctrl+`
# 查看python版本
python --version
# 安装Flask
pip install Flask
```

### 最小app

新建hello.py文件。

```
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"
```

### 运行app

```
flask --app hello run
# hello是app所在文件名
# 然后在浏览器中输入，终端输出的网址
# 就能在浏览器中看到网页

# 默认app监听http://127.0.0.1:5000，只能在本机访问
# 下面的命令指定ip和端口，可用从别的机器访问
flask --app hello run --host=0.0.0.0 --port=6000
```

## 部署

新建requirements.txt
这是python依赖的库，用于安装python库

```
Flask
gunicorn
```

Dockerfile
新建Dockerfile文件

```
FROM python
WORKDIR /project/demo

COPY requirements.txt ./
RUN pip install -r requirements.txt

COPY hello.py .
CMD [ "gunicorn", "--bind=0.0.0.0:5000", "hello:app" ]
```

构造docker镜像并运行

```
# 构造docker镜像
docker build -t helloflask .

# 运行镜像
docker run -d -p 127.0.0.1:5000:5000 --name helloflask helloflask

# 访问app
curl http://127.0.0.1:5000/

# 用命令行连接运行的容器
# 进入容器内部
# docker exec -it helloflask /bin/bash
# 主机和容器之间拷贝文件
# docker cp ./hello.py helloflask:/project/demo/
# 重启容器
# docker restart helloflask
# docker stop helloflask
# docker start helloflask
# 删除容器
# docker rm helloflask
# 删除镜像
# docker rmi helloflask
```