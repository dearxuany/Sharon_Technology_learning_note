# es 插件_ ik 中文分词器
es插件包含JAR文件，也可能包含脚本和配置文件，并且必须在集群中的每个节点上安装。安装之后，需要重启集群中的每个节点才能使插件生效。
## ik 中文分词器
https://github.com/medcl/elasticsearch-analysis-ik
### 安装
#### 方法1：es 节点网络下载
```
# es 服务器需能够连接外网
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.1/elasticsearch-analysis-ik-7.1.1.zip
```
#### 方法2： 使用源码包
```
# 下载 ik 二进制包
https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.1/elasticsearch-analysis-ik-7.1.1.zip

# 上传到服务器
# cd /sdata/usr/local/src
# rz
# chown elk:elk ./elasticsearch-analysis-ik-7.1.1.zip

# 解压插件包
# su - elk
$ mkdir -p /sdata/usr/local/elasticsearch/plugins/ik
$ unzip /sdata/usr/local/src/elasticsearch-analysis-ik-7.1.1.zip -d /sdata/usr/local/elasticsearch/plugins/ik/
$ cd /sdata/usr/local/elasticsearch/plugins/ik/ && ls
commons-codec-1.9.jar  commons-logging-1.2.jar  config  elasticsearch-analysis-ik-7.1.1.jar  httpclient-4.5.2.jar  httpcore-4.4.4.jar  plugin-descriptor.properties  plugin-security.policy

# 重启 es，注意启动用户必须为 elk
$ kill $(ps -ef|grep -v grep|grep elasticsearch|awk '{print $2}')
$ sh /sdata/usr/local/elasticsearch/start_elasticsearsh.sh
```
注：只要集群中其中一个节点安装了 ik 分词器则整个集群 ik 分词器可用，但若果装有 ik 分词器的独立节点故障则会直接导致集群 ik 分词器不可用，所以为避免故障需集群内每个节点均安装 ik 分词器

## 测试
dev tool 执行
```
GET _analyze?pretty
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国国歌"
}
```
返回值
```
{
  "tokens" : [
    {
      "token" : "中华人民共和国",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "国歌",
      "start_offset" : 7,
      "end_offset" : 9,
      "type" : "CN_WORD",
      "position" : 1
    }
  ]
}
```
devtool 执行
```
GET _analyze?pretty
{
  "analyzer": "ik_smart",
  "text": "今天中午吃奥尔良鸡扒拼青瓜好吗"
}
```
返回值
```
{
  "tokens" : [
    {
      "token" : "中午",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "吃",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "奥尔良",
      "start_offset" : 5,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "鸡",
      "start_offset" : 8,
      "end_offset" : 9,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "扒",
      "start_offset" : 9,
      "end_offset" : 10,
      "type" : "CN_CHAR",
      "position" : 4
    },
    {
      "token" : "拼",
      "start_offset" : 10,
      "end_offset" : 11,
      "type" : "CN_CHAR",
      "position" : 5
    },
    {
      "token" : "青瓜",
      "start_offset" : 11,
      "end_offset" : 13,
      "type" : "CN_WORD",
      "position" : 6
    },
    {
      "token" : "好吗",
      "start_offset" : 13,
      "end_offset" : 15,
      "type" : "CN_WORD",
      "position" : 7
    }
  ]
}
```
## 使用自定义词典
调整分词格式，使用本地字典 </br>
* 扩展词典：自行定义的词汇，主要为不想被拆分的词语</br>
* 停止词典：不想被识别的词语，比如介词、代词</br>
https://www.cnblogs.com/leixingzhi7/p/6903938.html</br>

注意：修改配置后需重启 es 生效
```
# vim /sdata/usr/local/elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>IK Analyzer 扩展配置</comment>
    <!--用户可以在这里配置自己的扩展字典 -->
    <entry key="ext_dict"></entry>
    <!--用户可以在这里配置自己的扩展停止词字典-->
    <entry key="ext_stopwords"></entry>
    <!--用户可以在这里配置远程扩展字典 -->
    <!-- <entry key="remote_ext_dict">words_location</entry> -->
    <!--用户可以在这里配置远程扩展停止词字典-->
    <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
    <entry key="remote_ext_dict">https://wiki-qa.domain.com/model_data/word_dict.txt</entry>
    <entry key="remote_ext_stopwords">https://wiki-qa.domain.com/model_data/stopwords.txt</entry>
</properties>
```
自定义词典测试
```
GET _analyze?pretty
{
  "analyzer": "ik_smart",
  "text": "三星火灾海上保险公司竟然火灾了，消防员立马出发到现场进行救灾。"
}

# 返回
{
  "tokens" : [
    {
      "token" : "三星火灾海上保险公司",
      "start_offset" : 0,
      "end_offset" : 10,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "火灾",
      "start_offset" : 12,
      "end_offset" : 14,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "消防员",
      "start_offset" : 16,
      "end_offset" : 19,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "出",
      "start_offset" : 21,
      "end_offset" : 22,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "发到",
      "start_offset" : 22,
      "end_offset" : 24,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "现场",
      "start_offset" : 24,
      "end_offset" : 26,
      "type" : "CN_WORD",
      "position" : 5
    },
    {
      "token" : "救灾",
      "start_offset" : 28,
      "end_offset" : 30,
      "type" : "CN_WORD",
      "position" : 6
    }
  ]
}
```
## ansible playbook 部署
由于需要操作多个节点，建议使用 ansible playbook 部署
```
---


- hosts: dev_es-03
  become: yes
  become_user: root
  vars:
    remote_ext_dict_url: "http://10.0.0.149:5000/model_data/word_dict.txt"
    remote_ext_stopwords_url: "http://10.0.0.149:5000/model_data/stopwords.txt"
  tasks:
  - name: Download package
    get_url:
      url: https://pkg.domainname.com/elasticsearch/elasticsearch-7.1.1/elasticsearch-analysis-ik-7.1.1.zip
      dest: /sdata/usr/local/src/
      mode: '0644'
      url_username: pkg
      url_password: passwd
      
  - name: check old ik dir
    shell: ls /sdata/usr/local/elasticsearch/plugins/ik
    ignore_errors: True
    register: result
    
  - name: remove old ik dir if exist
    file:
      path: /sdata/usr/local/elasticsearch/plugins/ik
      state: absent
    when: result|succeeded


  - name: create goal dir
    file: path=/sdata/usr/local/elasticsearch/plugins/ik state=directory mode=0755


  - name: release package
    shell: unzip /sdata/usr/local/src/elasticsearch-analysis-ik-7.1.1.zip -d /sdata/usr/local/elasticsearch/plugins/ik/


  - name: change owner
    shell: chown -R elk:elk /sdata/usr/local/elasticsearch/plugins/ik


  - name: show package
    shell: ls -al /sdata/usr/local/elasticsearch/plugins/ik/
    register: cmd_stdout


  - name: show command stdout
    debug: var=cmd_stdout verbosity=0


  - name: set ik config
    become: yes
    become_user: elk
    lineinfile:
      dest: /sdata/usr/local/elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml
      insertafter: '{{ item.insertafter }}'
      line: '{{ item.line }}'
    with_items:        
      - { insertafter: '\t<!-- <entry key=\"remote_ext_dict\">words_location</entry> -->', line: '  <entry key="remote_ext_dict">{{ remote_ext_dict_url }}</entry>'}
      - { insertafter: '\t<!-- <entry key=\"remote_ext_stopwords\">words_location</entry> -->', line: ' <entry key="remote_ext_stopwords">{{ remote_ext_stopwords_url }}</entry>'}




  - name: show config
    shell: cat /sdata/usr/local/elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml
    register: cmd_stdout


  - name: show command stdout
    debug: var=cmd_stdout verbosity=0


  - name: stop es
    shell: kill $(ps -ef|grep -v grep|grep elasticsearch|awk '{print $2}')
    
  - name: start es
    become: yes
    become_user: elk
    shell: sh /sdata/usr/local/elasticsearch/start_elasticsearsh.sh
```


