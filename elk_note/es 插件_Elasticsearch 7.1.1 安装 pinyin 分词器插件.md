# es 插件_Elasticsearch 7.1.1 安装 pinyin 分词器插件
## 依赖
### mvn 
安装插件前，需要用 maven 进行编译生成插件包
```
$ mvn -version
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-18T02:33:14+08:00)
Maven home: /sdata/usr/local/maven
Java version: 1.8.0_171, vendor: Oracle Corporation, runtime: /sdata/usr/local/jdk/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.1.3.el7.x86_64", arch: "amd64", family: "unix"
```
### git
用于到 github 下载 elasticsearch-analysis-pinyin 源码包

## elasticsearch-analysis-pinyin
可自行下载源码编译或直接下载编译好的包，此处直接使用编译完成的二进制包
### 自行编译
github 根据文档下载对应分支 elasticsearch-analysis-pinyin 源码包，github 地址  https://github.com/medcl/elasticsearch-analysis-pinyin
```
git clone https://github.com/medcl/elasticsearch-analysis-pinyin.git
```
进入项目目录，修改 pom.xml 将软件依赖修改为和 es 版本一致
```
# cd ./elasticsearch-analysis-pinyin/
# vim pom.xml
<elasticsearch.version>7.1.1</elasticsearch.version>
```
编译并在当前 target/releases 目录下生成了 elasticsearch-analysis-pinyin-7.1.1.zip 包
```
mvn clean install -Dmaven.test.skip
```
### 直接下载二进制包
github 二进制包下载 https://github.com/medcl/elasticsearch-analysis-pinyin/releases
```
wget https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v7.1.1/elasticsearch-analysis-pinyin-7.1.1.zip
```
此处将二进制包放至 nginx 目录进行代理，使用 ansible playbook 部署
```
---

- hosts: dev_es-03
  become: yes
  become_user: root
  tasks:
  - name: Download package
    get_url:
      url: https://pkg.domainname.com/elasticsearch/elasticsearch-7.1.1/elasticsearch-analysis-pinyin-7.1.1.zip
      dest: /sdata/usr/local/src/
      mode: '0644'
      url_username: pkg
      url_password: passwd


  - name: check old ik dir
    shell: ls /sdata/usr/local/elasticsearch/plugins/analysis-pinyin
    ignore_errors: True
    register: result
    
  - name: remove old ik dir if exist
    file:
      path: /sdata/usr/local/elasticsearch/plugins/analysis-pinyin
      state: absent
    when: result|succeeded
    
  - name: create goal dir
    file: path=/sdata/usr/local/elasticsearch/plugins/analysis-pinyin state=directory mode=0755


  - name: release package
    shell: unzip /sdata/usr/local/src/elasticsearch-analysis-pinyin-7.1.1.zip -d /sdata/usr/local/elasticsearch/plugins/analysis-pinyin/
    
  - name: change owner
    shell: chown -R elk:elk /sdata/usr/local/elasticsearch/plugins/analysis-pinyin
    
  - name: show package
    shell: ls -al /sdata/usr/local/elasticsearch/plugins/analysis-pinyin
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

## 使用测试
使用 kibana dev tool 对集群进行测试
```
GET /_analyze?pretty
{
  "text": ["刘德华"],
  "analyzer": "pinyin"
}
```
得到以下结果则 elasticsearch-analysis-pinyin 正常使用
```
{
  "tokens" : [
    {
      "token" : "liu",
      "start_offset" : 0,
      "end_offset" : 0,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "ldh",
      "start_offset" : 0,
      "end_offset" : 0,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "de",
      "start_offset" : 0,
      "end_offset" : 0,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "hua",
      "start_offset" : 0,
      "end_offset" : 0,
      "type" : "word",
      "position" : 2
    }
  ]
}
```
