# Apache SkyWalking 客户端 java-agent 接入
skywalking agent 代码位于 skywalking/agent 当中，Skywalking Agent 使用 Javaagent 做字节码植入，无侵入式的收集，并通过HTTP或者gRPC方式发送数据到Skywalking Collector。</br>
agent 配置支持外部参数配置覆盖，配置加载顺序为"探针配置 > 系统配置 > 系统环境变量 > 配置文件参数值"。</br>
https://github.com/apache/skywalking/blob/v7.0.0/docs/en/setup/service-agent/java-agent/README.md
```
# tree -L 1
.
├── activations
├── bootstrap-plugins  # 已有插件
├── config             # 配置文件
├── logs
├── optional-plugins   # 可选插件（未生效，注意性能，需搬到 plugins 下)
├── plugins
└── skywalking-agent.jar   


6 directories, 1 file
```
压缩打包 skywalking-agent
```
# mv ./agent ./skywalking-agent
# tar -zcvf skywalking-agent-7.0.0.tar.gz ./skywalking-agent
```
agent.config 中主要有以下几个参数需要配置：
* 为了单台主机上多个应用可以共用一个 skywalking-agent，所以 agent.service_name 作为 java 应用启动配置覆盖配置文件中的配置，单个应用使用单个 service_name，service_name 以 env_projectname 组成。多节点使用同一个 service_name，skywalking 以实例作为节点区分。
* backend_service、日志信息配置使用 ansible playbook 在部署同时写入


```
# The service name in UI
agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}  


# Backend service addresses.
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:10.0.0.57:11800}


# Logging file_name
logging.file_name=${SW_LOGGING_FILE_NAME:skywalking-agent.log}


# Logging level
logging.level=${SW_LOGGING_LEVEL:INFO}


# Logging dir
logging.dir=${SW_LOGGING_DIR:/sdata/var/log/skywalking-agent}
```
skywalking-agent 包放置在 nginx 代理的目录里，以 jenkins 调用 ansible playbook 请求 url 获取包远程部署到各主机
```
- name: Create Goal Dir
  file: path=/sdata/usr/local/src state=directory mode=0755


- name: Download package
  become: yes
  become_user: root
  get_url:
    url: https://pkg.domainname.com/skywalking/skywalking-agent-7.0.0.tar.gz
    dest: /sdata/usr/local/src/
    mode: '0644'
    url_username: pkg
    url_password: passwd
    
- name: check old skywalking-agent
  shell: ls /sdata/usr/local/skywalking-agent
  ignore_errors: True
  register: result
  
- name: remove old skywalking-agent if exist
  become: yes
  become_user: root
  file:
    path: /sdata/usr/local/skywalking-agent
    state: absent
  when: result|succeeded


- name: release package
  shell: tar -zxf /sdata/usr/local/src/skywalking-agent-7.0.0.tar.gz -C /sdata/usr/local/
  become: yes
  become_user: root


- name: create log dir
  become: yes
  become_user: root
  file: path=/sdata/var/log/skywalking-agent state=directory owner={{ service_user }} group={{ service_user }} mode=0755


- name: set skywalking server config
  become: yes
  become_user: root
  lineinfile:
    dest: /sdata/usr/local/skywalking-agent/config/agent.config
    #regexp: "^collector.backend_service"
    #line: "collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:{{ skywalking_server }}}"
    regexp: '{{ item.regexp }}'
    line: '{{ item.line }}'
    #state: present
  with_items:
    - { regexp: '^collector.backend_service', line: 'collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:{{ skywalking_server }}}'}
    - { regexp: '^logging.file_name', line: 'logging.file_name=${SW_LOGGING_FILE_NAME:skywalking-agent.log}'}
    - { regexp: '^logging.level', line: 'logging.level=${SW_LOGGING_LEVEL:INFO}'}
    - { regexp: '^logging.dir', line: 'logging.dir=${SW_LOGGING_DIR:/sdata/var/log/skywalking-agent}'}
    


- name: execute command
  shell: "cat /sdata/usr/local/skywalking-agent/config/agent.config |grep collector.backend_service"
  register: cmd_stdout


- name: show command stdout
  debug: var=cmd_stdout verbosity=0


- name: chown
  shell: sudo chown -R {{ service_user }}.{{ service_user }} /sdata/usr/local/skywalking-agent
````




## java-agent 接入
https://github.com/apache/skywalking/blob/v7.0.0/docs/en/setup/service-agent/java-agent/README.md </br>

探针部分
```
-javaagent:/sdata/usr/local/skywalking-agent/skywalking-agent.jar -Dskywalking.agent.service_name=envname_appname
```
JAR file 应用接入 skywalking-agent 启动配置，需要注意配置 agent.service_name
```
nohup /sdata/usr/local/jdk/bin/java -jar -javaagent:/sdata/usr/local/skywalking-agent/skywalking-agent.jar -Dskywalking.agent.service_name=prd_nerites-knowledge-centre-backend -Xms1g -Xmx1g /sdata/app/nerites-knowledge-centre-backend/nerites-knowledge-centre-backend.jar --server.port=8786 &>> /sdata/var/log/nerites-knowledge-centre-backend/nerites-knowledge-centre-backend_stdout.log &
```


## jetty 接入
修改 jetty/start.ini 新增 skywalking 启动参数
```
# vim /sdata/usr/local/jetty/start.ini

#===========================================================
# Jetty Startup
#
# Starting Jetty from this {jetty.home} is not recommended.
#
# A proper {jetty.base} directory should be configured, instead
# of making changes to this {jetty.home} directory.
#
# See documentation about {jetty.base} at
# http://www.eclipse.org/jetty/documentation/current/startup.html
#
# A demo-base directory has been provided as an example of
# this sort of setup.
#
# $ cd demo-base
# $ java -jar ../start.jar
#
#===========================================================


#skywalking
-javaagent:/sdata/usr/local/skywalking-agent/skywalking-agent.jar
-Dskywalking.agent.service_name=env_appname
```
## tomcat 接入
在 tomcat catalina.sh 启动脚本中加入 skywalking 启动参数
```
vim /sdata/usr/local/tomcat/bin/catalina.sh

CATALINA_OPTS="-javaagent:/sdata/usr/local/skywalking-agent/skywalking-agent.jar -Dskywalking.agent.service_name=env_appname"
```
