# docker 常用命令、dockerfile_python flask example
## 源代码及依赖
准备代码及环境依赖
```
# app.py


from flask import Flask
import socket
import os


app = Flask(__name__)


@app.route('/')
def hello():
    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>"           
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname())
    
if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```
```
# vim requirements.txt
Flask
```
## Dockerfile
编写 Dockerfile
```
# cat Dockerfile
# 使用官方提供的Python开发镜像作为基础镜像
FROM python:2.7-slim


# 将工作目录切换为/app
WORKDIR /app


# 将当前目录下的所有内容复制到/app下
ADD . /app


# 使用pip命令安装这个应用所需要的依赖
RUN pip install --trusted-host pypi.python.org -r requirements.txt


# 允许外界访问容器的80端口
EXPOSE 80


# 设置环境变量
ENV NAME World


# 设置容器进程为：python app.py，即：这个Python应用的启动命令
CMD ["python", "app.py"]

```
## 构建镜像
docker build -t 容器名 Dockerfile路径
```
# docker build -t helloworld .
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM python:2.7-slim
2.7-slim: Pulling from library/python
68ced04f60ab: Pull complete
08b47f0371a2: Pull complete
afa5f7c888db: Pull complete
1fad109bc0a5: Pull complete
Digest: sha256:2117cd714ad2212aaac09aac7088336241214c88dc7105e50d19c269fde6e0f6
Status: Downloaded newer image for python:2.7-slim
---> 9458934200c5
Step 2/7 : WORKDIR /app
---> Running in 7cfbd77679cc
Removing intermediate container 7cfbd77679cc
---> 3079f5c78149
Step 3/7 : ADD . /app
---> cef290bc94c0
Step 4/7 : RUN pip install --trusted-host pypi.python.org -r requirements.txt
---> Running in a69736f93166
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support
Collecting Flask
  Downloading Flask-1.1.1-py2.py3-none-any.whl (94 kB)
Collecting click>=5.1
  Downloading click-7.1.1-py2.py3-none-any.whl (82 kB)
Collecting itsdangerous>=0.24
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting Werkzeug>=0.15
  Downloading Werkzeug-1.0.0-py2.py3-none-any.whl (298 kB)
Collecting Jinja2>=2.10.1
  Downloading Jinja2-2.11.1-py2.py3-none-any.whl (126 kB)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1-cp27-cp27mu-manylinux1_x86_64.whl (24 kB)
Installing collected packages: click, itsdangerous, Werkzeug, MarkupSafe, Jinja2, Flask
Successfully installed Flask-1.1.1 Jinja2-2.11.1 MarkupSafe-1.1.1 Werkzeug-1.0.0 click-7.1.1 itsdangerous-1.1.0
Removing intermediate container a69736f93166
---> a62669fbf284
Step 5/7 : EXPOSE 80
---> Running in 5e8db8e6e520
Removing intermediate container 5e8db8e6e520
---> d27b5d6c1129
Step 6/7 : ENV NAME World
---> Running in ea92e5a4a16b
Removing intermediate container ea92e5a4a16b
---> a8875e11a7cd
Step 7/7 : CMD ["python", "app.py"]
---> Running in eb280cb85081
Removing intermediate container eb280cb85081
---> 7c9c4d071818
Successfully built 7c9c4d071818
Successfully tagged helloworld:latest
```
启动容器
```
# docker run -p 4000:80 helloworld
* Serving Flask app "app" (lazy loading)
* Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
* Debug mode: off
* Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```
查看容器启动
```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
3358c370f314        helloworld          "python app.py"     6 minutes ago       Up 6 minutes        0.0.0.0:4000->80/tcp   musing_dubinsky
```
访问 flask
```
#  curl http://localhost:4000
<h3>Hello World!</h3><b>Hostname:</b> 3358c370f314<br/>
```
容器日志输出（访问来源非实际访问来源 ip）
```
172.17.0.1 - - [20/Mar/2020 07:11:14] "GET / HTTP/1.1" 200 -
```

## 镜像上传
根据镜像设置镜像版本
```
# docker tag helloworld sharon/helloworld:v1
```
新增了一个带版本信息、开发者信息的镜像
```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
helloworld          latest              7c9c4d071818        11 minutes ago      158MB
sharon/helloworld   v1                  7c9c4d071818        11 minutes ago      158MB
python              2.7-slim            9458934200c5        3 weeks ago         148MB
```
上传镜像到仓库
```
docker push sharon/helloworld:v1
```
查看容器内容
```
# docker exec -it 8221656366be /bin/sh
# pwd
/app
# ls
Dockerfile  app.py  requirements.txt
```
在容器中新增文件后，根据运行容器构建镜像
```
# docker commit 8221656366be sharon/helloworld:v2
sha256:915559936b8c859158d498e62112d3ecf3b92cbcfe8f6b2d394b9dc227b1fe3f
```
运行新版本容器
```
# docker run -p 4000:80 sharon/helloworld:v2
```
查看容器在宿主机的 pid
```
# docker inspect --format '{{ .State.Pid }}' 12c5e4aa81c0
7854
```
查看宿主机进程 namespace
```
[root@lyx-opd-jenkins-01 ~]# ls -l /proc/7854/ns
总用量 0
lrwxrwxrwx. 1 root root 0 3月  20 15:24 ipc -> ipc:[4026532502]
lrwxrwxrwx. 1 root root 0 3月  20 15:24 mnt -> mnt:[4026532500]
lrwxrwxrwx. 1 root root 0 3月  20 15:24 net -> net:[4026532505]
lrwxrwxrwx. 1 root root 0 3月  20 15:24 pid -> pid:[4026532503]
lrwxrwxrwx. 1 root root 0 3月  20 15:30 user -> user:[4026531837]
lrwxrwxrwx. 1 root root 0 3月  20 15:24 uts -> uts:[4026532501]
```
## volume 挂载
如果没有对应在 Dockerfile 中注明的挂载路径，则默认使用  /var/lib/docker/volumes/[VOLUME_ID]/_data，这个路径可在 /etc/docker/daemon.json 中的 data-root 参数设置。本机是在 /sdata/docker/overlay2/[可读写层 ID]/merged
```
# docker run -p 4000:80 -v /sdata/app/docker-test/docker-python-test/flask-test:/v-test sharon/helloworld:v2
```
容器持久化目录
```
# docker exec -it 01b9aec725fd /bin/bash
root@01b9aec725fd:/app# cd /v-test/
root@01b9aec725fd:/v-test# ls
Dockerfile  app.py  requirements.txt  volumn-test
root@01b9aec725fd:/v-test# touch v-test.txt
```
宿主机查看挂载点
```
[root@lyx-opd-jenkins-01 flask-test]# [root@lyx-opd-jenkins-01 flask-test]# tree -L 2
.
├── app.py
├── Dockerfile
├── requirements.txt
├── volumn-test
│   └── hello.txt
└── v-test.txt
```
容器中的内容和宿主机中挂载目录的内容是一样的，但 commit 镜像时这些内容不会被打包到镜像中。
```
# 不注明挂载点
# docker run -p 4000:80 -v /v-test sharon/helloworld:v2
```
查看 Volume ID
```
# docker volume ls
DRIVER              VOLUME NAME
local               93296333cb6001390801e58c8d55d562784fac4d311ee9085b317c29c86d62f8
```
docker 在宿主机中创建的持久化目录
```
/sdata/docker/volumes/93296333cb6001390801e58c8d55d562784fac4d311ee9085b317c29c86d62f8/_data
```
