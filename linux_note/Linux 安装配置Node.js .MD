# Linux 上安装配置 Node.js
Node.js安装包及源码下载地址为：https://nodejs.org/en/download/</br>
安装 node-v8.11.3-linux-x86 LTS(长期支持版本)版本</br>
注意：Linux上安装Node.js需要安装Python 2.6 或 2.7 ，不建议安装Python 3.0以上版本。</br>

## 下载安装包
linux上最新版本Node.js的下载地址：</br>
https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-x86.tar.xz</br>
```
cd /usr/local/src
[root@centOSlearning src]# wget https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-x86.tar.xz
```
## 解压
```
[root@centOSlearning src]# tar -xvJ -f ./node-v8.11.3-linux-x86.tar.xz
```

## 修改配置文件
查看安装路径</br>
```
[root@centOSlearning node-v8.11.3-linux-x86]# pwd
/usr/local/src/node-v8.11.3-linux-x86
```

修改配置文件
```
vim /etc/profile
```
文件末尾添加，保存
```
export NODE_HOME=/usr/local/src/node-v8.11.3-linux-x86    #等号后面填node.js的安装路径
export PATH=$PATH:$NODE_HOME/bin
export NODE_PATH=$NODE_HOME/lib/node_modules
```

## 使配置文件生效
```
source /etc/profile
```

## 查看node.js的版本号
有输出代表node.js安装成功
```
[root@centOSlearning ~]# node -v
v8.11.3
[root@centOSlearning ~]# npm -v 
5.6.0
```
## npm模块安装路径
```
[root@centOSlearning node_modules]# pwd
/usr/local/src/node-v8.11.3-linux-x86/lib/node_modules
```
