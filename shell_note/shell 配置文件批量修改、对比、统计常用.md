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
