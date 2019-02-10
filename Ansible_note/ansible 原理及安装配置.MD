# ansible 原理及安装配置
4 大主流的组态管理工具 (Puppet, SaltStack, Chef, Ansible) ，Python 阵营的组态管理工具</br>
Puppet 和 Chef 基于 Ruby，SaltStack 和 Ansible 基于 Python</br>

## ansible 原理
ansible 通过 inventory 档案来定义有哪些 Managed node (被控端)，并借由 SSH 和 Python 进行沟通。</br>
只需在 Control Machine 里安装 Ansible 即可，通过SSH被连上的机器里有预载 Python 时，Ansible 就可以运作。</br>

## ansible 安装
CentOS 上可用 yum、源码、pip 安装
```
# 使用 yum 安装
[sunnylinux@centOSlearning ~]$ sudo yum install ansible
```
查看版本号
```
[sunnylinux@centOSlearning ~]$ ansible --version
ansible 2.4.2.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/sunnylinux/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr 11 2018, 08:07:30) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
```
可以看到默认的 python version 是2.7，所以想用 python3 最好还是直接用 pip3 安装 ansible 比较好
详情： https://docs.ansible.com/ansible/2.3/python_3_support.html
```
# 卸载 ansible
[sunnylinux@centOSlearning ~]$ sudo yum remove ansible

# 和装python包不一样，这个必须有root权限，因为是装软件
[sunnylinux@centOSlearning ~]$ sudo pip3 install ansible 

Successfully installed PyYAML-3.13 ansible-2.7.7 bcrypt-3.1.6 paramiko-2.4.2 pyasn1-0.4.5 pynacl-1.3.0
```
pip3 安装 ansible 是源码安装，所以不能直接使用 ansible 命令
```
# 找一下 ansible 的启动脚本被pip3 安装在哪儿
[sunnylinux@centOSlearning usr]$ sudo find ./ -name ansible
[sudo] sunnylinux 的密码：
./local/python3/bin/ansible
./local/python3/lib/python3.7/site-packages/ansible

[sunnylinux@centOSlearning usr]$ sudo find ./ -name playbook
./local/python3/lib/python3.7/site-packages/ansible/playbook
```
查看版本号
```
[sunnylinux@centOSlearning usr]$ /usr/local/python3/bin/ansible --version
ansible 2.7.7
  config file = None
  configured module search path = ['/home/sunnylinux/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/python3/lib/python3.7/site-packages/ansible
  executable location = /usr/local/python3/bin/ansible
  python version = 3.7.0a1 (default, May 18 2018, 14:10:18) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
```
为直接使用 ansible 命令，需要创建一下链接到/usr/bin下
```
[sunnylinux@centOSlearning usr]$ sudo ln -s /usr/local/python3/bin/ansible /usr/bin/ansible
[sunnylinux@centOSlearning usr]$ ansible --version
ansible 2.7.7
  config file = None
  configured module search path = ['/home/sunnylinux/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/python3/lib/python3.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.7.0a1 (default, May 18 2018, 14:10:18) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
```
测试一下
```
[sunnylinux@centOSlearning usr]$  python3 /usr/bin/ansible localhost -m ping
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

```
