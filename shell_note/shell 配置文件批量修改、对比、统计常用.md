# shell 配置文件批量修改、对比、统计常用
## shell 对比两个文件找不同行
统计两个文本文件的相同行
```
grep -Ff file1 file2
```
统计file2中有，file1中没有的行
```
grep -vFf file2 file1
```
example
```
# grep -vFf dev-helm  qas-helm
--set external_ingress.http.paths.path
--set dp.hpa_memory
--set dp.hpa_cpu
--set dp.containers.cont1.name
--set dp.containers.cont1.port
--set dp.containers.cont1.args
```

## sed 在指定关键字行前后添加内容
多个文本，关键字行前新增一行内容
```
grep "jar" * -R| awk -F: {'print $1'}| sort| uniq |xargs sed -i '/start_command/i\skywalking_conf="-javaagent:/sdata/usr/local/skywalking-agent/skywalking-agent.jar -Dskywalking.agent.service_name={{ deploy_env }}_{{ project_env }}"\n'
```
多个文本，关键字行后新增一行内容，添加多行，行间加\n
```
grep "Dev:" * -R| awk -F: {'print $1'}| sort| uniq |xargs sed -i '/Dev:/a\Dev_NodeAppTag_Status=no\nDev_nodeSelect_appTag_value='
```
单个文本，关键字后新增一行内容
```
sed -i '/Allow\ root\ to\ run\ any\ commands\ anywhere/aosp     ALL=(ALL)       NOPASSWD: ALL'  /etc/sudoers
```

