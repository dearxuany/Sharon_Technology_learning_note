# es 集群优化_设置 fielddata 缓存自动清理
## 背景
es 集群在程序开发使用过程中报 429 Too Many Requests circuit_breaking_exception ES fieldData，kibana 无法访问。</br>
```
# 实际开发使用集群为单节点 es jvm 4GB，但没有把日志记录下来，找了一条内容近似的。

{
  "statusCode": 429,
  "error": "Too Many Requests",
  "message": "[circuit_breaking_exception] 
  [parent] Data too large, data for [<http_request>] would be [2087772160/1.9gb], 
  which is larger than the limit of [1503238553/1.3gb], 
  real usage: [2087772160/1.9gb],
  new bytes reserved: [0/0b], 
  usages [request=0/0b, fielddata=1219/1.1kb, in_flight_requests=0/0b, accounting=605971/591.7kb], 
  with { bytes_wanted=2087772160 & bytes_limit=1503238553 & durability=\"PERMANENT\" }"
}
```

## 原因
* 内存分配 </br>
ES的JVM Heap中的状况，可以看到有两条界限：驱逐线 和 断路器。当缓存数据到达驱逐线时，会自动驱逐掉部分数据，把缓存保持在安全的范围内。
当用户准备执行某个查询操作时，断路器就起作用了，缓存数据+当前查询需要缓存的数据量到达断路器限制时，会返回Data too large错误，阻止用户进行这个查询操作。

* FieldData </br>
ES把缓存数据分成两类，FieldData和其他数据。ES配置中提到的FieldData指的是字段数据。当排序（sort），统计（aggs）时，ES把涉及到的字段数据全部读取到内存（JVM Heap）中进行操作。相当于进行了数据缓存，提升查询效率。</br>

* indices.fielddata.cache.size </br>
Data too large异常是ES默认配置的一个坑，es 没有对该值进行默认设置，我们没有配置indices.fielddata.cache.size，它就不回收缓存了。缓存到达限制大小，无法往里插入数据。


## 解决办法
在文件 config/elasticsearch.yml 文件中设置缓存使用回收
```
# 设置为当缓存大小达到 jvm 分配的内存 70% 时进行缓存回收
indices.fielddata.cache.size: 70%
```
https://www.cnblogs.com/sanduzxcvbnm/p/11982476.html </br>
https://www.jianshu.com/p/49260d54beaf </br>
