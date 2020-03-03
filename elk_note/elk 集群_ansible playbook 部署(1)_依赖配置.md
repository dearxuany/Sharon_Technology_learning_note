# elk 集群_ansible playbook 部署(1)_依赖配置
## ansible 配置
### ansible ssh 配置
#### 服务端配置
客户端添加 ansible ssh 链接用户
```
# useradd ansible
# passwd ansible
```
ansible 服务端生成密钥对
```
$ ssh-keygen
```
ansible 客户端 ssh 配置 
```
# 由于一般使用堡垒机登录，无法得知客户端用户密码，所以直接登录 ansible 客户端直接粘贴公钥给远程用户
# mkdir ~/.ssh
# touch ~/.ssh/authorized_keys
# echo "ssh-rsa passwdcode ansible@opd-ansible-01"  >> ./.ssh/authorized_keys
```
ansible 公钥文件权限配置
```
# chmod 700 ~/.ssh
# chmod 600 ~/.ssh/authorized_keys
```
启动 ansible 虚拟环境
```
[ansible@opd-ansible-01 ~]$ source /sdata/usr/local/.py3-a2.5-env/bin/activate
(.py3-a2.5-env) [ansible@opd-ansible-01 ~]$ source /sdata/usr/local/.py3-a2.5-env/ansible/hacking/env-setup  -q
```
使用 shell 脚本激活虚拟环境
```
# vim ./venvActivate.sh
#! /bin/bash
source /sdata/usr/local/.py3-a2.5-env/bin/activate
source /sdata/usr/local/.py3-a2.5-env/ansible/hacking/env-setup  -q
```
注意：使用 shell 脚本激活虚拟环境不能使用 sh，否则无法激活虚拟环境
```
[ansible@opd-ansible-01 playbook]# . venvActivate.sh
(.py3-a2.5-env) [ansible@opd-ansible-01 playbook]#
```
#### 客户端配置
客户端 ops 用户添加 sudo 权限
```
# vim /etc/sudoers
ops  ALL=(ALL)   NOPASSWD:ALL
```
免密 sudo 用户 ansible playbook
```
---
- hosts: dev
  gather_facts: F
  become: yes
  become_user: root
  tasks:
  - name: add sudo user without passwd
    lineinfile:
      dest: /etc/sudoers
      insertbefore: "^## Allows members of the 'sys' group to run networking, software"
      line: "ops     ALL=(ALL)       NOPASSWD:ALL"
      state: present
  - name: checkout sudo user
    shell: cat /etc/sudoers|grep NOPASSWD:ALL
    register: check_output
  - name: show sudo user checkout output
    debug: var=check_output verbosity=0
```
执行 playbook
```
#! /bin/bash
ansible-playbook -i /sdata/app/ansible-deploy/gzyw53-playbook/inventory/production/all_hosts_ip --user=ops --private-key=/home/ansible/.ssh/id_rsa -s /sdata/app/ansible-deploy/gzyw53-playbook/roles/system/addSudo.yml
```
使用 ansible ping 客户端主机
```
(.py3-a2.5-env) [ansible@opd-ansible-01 ~]# ansible -i /sdata/usr/local/ansible/inventory/staging/testenv all --user=ansible -m ping --private-key=/home/ansible/.ssh/id_rsa -s
[DEPRECATION WARNING]: The sudo command line option has been deprecated in favor of the "become" command line arguments. This feature will be
removed in version 2.6. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
10.0.0.151 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### elk 集群 ansible inventory
```
[opd_ansible-01]
127.0.0.1

[opd_logstash-01]
10.0.0.151
[opd_logstash-01:vars]
server_names="opd-elk-01"

[opd_es-01]
10.0.0.152
[opd_es-01:vars]
server_names="opd-elk-02"

[opd_es-02]
10.0.0.153
[opd_es-02:vars]
server_names="opd-elk-03"

[opd_es-03]
10.0.0.154
[opd_es-03:vars]
server_names="opd-elk-04"

[opd_kibana-01]
10.0.0.151
[opd_kibana-01:vars]
server_names="opd-elk-01"
```

## 服务器目录初始化
服务器目录初始化 ansible playbook
```
---
- hosts: all
  tasks:
  - name: make src directory    
    file: path=/sdata/usr/local/src state=directory mode=0755  
  - name: make etc directory  
    file: path=/sdata/etc  state=directory mode=0755
  - name: make log directory
    file: path=/sdata/var/log state=directory mode=0755
  - name: make scripts directory
    file: path=/sdata/scripts state=directory mode=0755
```
ansible 执行部署
```
#! /bin/bash
ansible-playbook -i /sdata/app/ansible-deploy/gzyw53-playbook/inventory/production/all_hosts_ip --user=ops --private-key=/home/ansible/.ssh/id_rsa -s /sdata/app/ansible-deploy/gzyw53-playbook/roles/system/mkdir.yml
```

## java 安装
kafka、logstash、es、kibana 均需要用到 java，需在对应节点部署 jdk 8
### java 安装
ansible playbook 如下所示
```
---
- hosts: all
  tasks:
  - name: Copy Package
    copy: src=/sdata/usr/local/ansible/roles/java/files/jdk-8u171-linux-x64.tar.gz dest=/sdata/usr/local/src/jdk.tar.gz
  - name: Tar Package
    shell: cd /sdata/usr/local; tar -xvz -f /sdata/usr/local/src/jdk.tar.gz -C /sdata/usr/local
  - name: Rename jdk
    shell: mv /sdata/usr/local/jdk1.8.0_171 /sdata/usr/local/jdk
  - name: Set Java Path
    shell: sed -i.bak '$a export JAVA_HOME=/sdata/usr/local/jdk\nexport PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH\nexport CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar' /etc/profile
  - name: Source Profile
    shell: sudo source /etc/profile
```
ansible 执行部署
```
ansible-playbook -i /sdata/usr/local/ansible/inventory/staging/elk_hosts --user=ansible --private-key=/home/ansible/.ssh/id_rsa -s /sdata/usr/local/ansible/roles/java/main.yml
```
### java 验证
```
# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

## git 安装
为方便统一自动化管理，使用 gitlab 统一管理 ansible playbook 以及 filebeat/logstash/elasticsearch/kibana 的配置。
```
yum -y install git
```
