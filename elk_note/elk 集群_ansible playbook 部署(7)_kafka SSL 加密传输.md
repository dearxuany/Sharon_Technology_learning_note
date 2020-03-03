# elk 集群_ansible playbook 部署(7)_kafka SSL 加密传输
由于 kafka 到 logstash 从阿里云主机到本地内网需走公网传输，而传输内容比较敏感，所以必须使用 ssl 认证及加密处理，使数据内容不已明文传输。
## ssl 证书处理
### kafka ssl 证书生成
kafka 所在服务器创建4个目录，分别存放根证书、服务端证书、客户端正式、受信任的证书。
```
# mkdir -p /sdata/usr/local/kafka/ca/{root,server,client,trust}
# chown -R elk:elk /sdata/usr/local/kafka/ca/
```
server.keystore.jks 生成，生成服务端的keystore文件，注意 -validity 参数需填证书有效期，此处设置为 365 天，-alias 填写 kafka 的节点名， -ext SAN=DNS:{FQDN} 填写 kafka 所在主机名。
```
keytool -keystore /sdata/usr/local/kafka/ca/server/server.keystore.jks -alias alihn1-kafka-01 -validity 365 -genkey -keyalg RSA -keypass passwd -storepass passwd   -dname "CN=alihn1-opd-elk-01,OU=ops,O=company,L=guangzhou,S=guangdong,C=cn" -ext SAN=DNS:alihn1-opd-elk-01
```
查看 keystore 文件，需要输入前面设置的 keystore password。
```
# keytool -list -v -keystore /sdata/usr/local/kafka/ca/server/server.keystore.jks
```
生成 client.keystore.jks
```
keytool -keystore /sdata/usr/local/kafka/ca/client/client.keystore.jks -alias alihn1-kafka-01 -validity 365 -genkey -keypass passwd -storepass passwd -dname "CN=alihn1-opd-elk-01,OU=ops,O=company,L=guangzhou,S=guangdong,C=cn" -ext SAN=DNS:alihn1-opd-elk-01
```
生成 CA 证书
```
openssl req -new -x509 -keyout /sdata/usr/local/kafka/ca/root/ca-key -out /sdata/usr/local/kafka/ca/root/ca-cert -days 365 -passout pass:passwd -subj /C=CN/ST=guanzhou/L=guangzhou/O=company/CN=alihn1-opd-elk-01
```
根据 CA  证书创建  client.truststore.jks
```
keytool -keystore /sdata/usr/local/kafka/ca/trust/client.truststore.jks -alias CARoot -import -file /sdata/usr/local/kafka/ca/root/ca-cert -storepass passwd -keypass passwd
```
根据 CA  证书创建  server.truststore.jks
```
keytool -keystore /sdata/usr/local/kafka/ca/trust/server.truststore.jks -alias CARoot -import -file /sdata/usr/local/kafka/ca/root/ca-cert -storepass passwd -keypass passwd
```
### 导出、签发、重新导入 server.cert-file 证书 
导出服务端证书 server.cert-file
```
keytool -keystore /sdata/usr/local/kafka/ca/server/server.keystore.jks -alias alihn1-kafka-01 -certreq -file /sdata/usr/local/kafka/ca/server/cert-file -storepass passwd -keypass passwd
```
使用 CA 证书给 server.cert-file 签名
```
openssl x509 -req -CA /sdata/usr/local/kafka/ca/root/ca-cert -CAkey /sdata/usr/local/kafka/ca/root/ca-key -in /sdata/usr/local/kafka/ca/server/cert-file -out /sdata/usr/local/kafka/ca/server/server.cert-signed -days 365 -CAcreateserial -passin pass:passwd
```
将 CA 证书导入服务端 keystore
```
keytool -keystore /sdata/usr/local/kafka/ca/server/server.keystore.jks -alias CARoot -import -file /sdata/usr/local/kafka/ca/root/ca-cert -storepass passwd -keypass passwd
```
将已签名的 server.cert-signed 导入服务端 keystore
```
keytool -keystore /sdata/usr/local/kafka/ca/server/server.keystore.jks -alias alihn1-kafka-01 -import -file /sdata/usr/local/kafka/ca/server/server.cert-signed -storepass passwd -keypass passwd
```
### 导出、签发、重新导入 client.cert-file 证书 
导出客户端证书
```
keytool -keystore /sdata/usr/local/kafka/ca/client/client.keystore.jks -alias alihn1-kafka-01 -certreq -file /sdata/usr/local/kafka/ca/client/cert-file -storepass passwd -keypass passwd
```
使用 CA 证书给 client.cert-file 签名
```
openssl x509 -req -CA /sdata/usr/local/kafka/ca/root/ca-cert -CAkey /sdata/usr/local/kafka/ca/root/ca-key -in /sdata/usr/local/kafka/ca/client/cert-file -out /sdata/usr/local/kafka/ca/client/client.cert-signed -days 365 -CAcreateserial -passin pass:passwd
```
将 CA 证书导入客户端 keystore
```
keytool -keystore /sdata/usr/local/kafka/ca/client/client.keystore.jks -alias CARoot -import -file /sdata/usr/local/kafka/ca/root/ca-cert -storepass passwd -keypass passwd
```
将已签名的 client.cert-signed 导入客户端 keystore
```
keytool -keystore /sdata/usr/local/kafka/ca/client/client.keystore.jks -alias alihn1-kafka-01 -import -file /sdata/usr/local/kafka/ca/client/client.cert-signed -storepass passwd -keypass passwd
```
注意：必须先导入CA证书，再导入经过签名的证书

## kafka ssl 配置
由于生产者处于阿里云内网，使用内网 ip 和 kafka 通信不需要加密，而 kafka 到消费者 logstash 走公网需加密传输，所以阿里云 vpc 内网机器开放 9092 来进行明文传输，而外网使用 19092 来进行 SSL 传输。注意：listeners 和 advertised.listeners 开放的端口必须是对应的，所以 listeners 还是需要开放 SSL 端口，advertised.listeners 还是需要开放 PLAINTEXT。
由于安全考虑，外网需要禁止访问 ecs 的 9092 明文传输端口，需配置安全策略让 9092 仅能让阿里云 vpc 内机器访问。
```
# vim server.properties
listeners=PLAINTEXT://alihn1-opd-elk-01:9092,SSL://alihn1-opd-elk-01:9092
advertised.listeners=PLAINTEXT://alihn1-opd-elk-01:9092,SSL://alihn1-opd-elk-01:19092
```
将 ssl 配置写于 server.properties 最上面，防止由于配置文件读取问题导致报错。
```
# vim server.properties
ssl.keystore.location=/sdata/usr/local/kafka/ca/server/server.keystore.jks
ssl.keystore.password=passwd
ssl.key.password=passwd
ssl.truststore.location=/sdata/usr/local/kafka/ca/trust/server.truststore.jks
ssl.truststore.password=passwd
ssl.client.auth=required
ssl.enabled.protocols=TLSv1.2,TLSv1.1,TLSv1
ssl.keystore.type=JKS
ssl.truststore.type=JKS
ssl.secure.random.implementation=SHA1PRNG
security.inter.broker.protocol=SSL
```
启动 zookeeper 和 kafka 后，使用 openssl 进行加密验证
```
$ openssl s_client -debug -connect alihn1-opd-elk-01:19092 -tls1
CONNECTED(00000003)
Secure Renegotiation IS supported
```
## 生产消费者测试
将在 kafka 服务器上生成的 client.truststore.jks 和 client.keystore.jks 复制到 kafka 的各个客户端机器并新增一个 cluster-ssl.properties 用于描述 ssl 的相关信息
```
# vim client-ssl.properties
security.protocol=SSL
ssl.truststore.location=/sdata/usr/local/kafka/ca/trust/client.truststore.jks
ssl.truststore.password=passwd
ssl.keystore.location=/sdata/usr/local/kafka/ca/client/client.keystore.jks
ssl.keystore.password=passwd
ssl.key.password=passwd
```
在 logstash 服务器(本地内网)上进行测试
```
# 加密生产
# ./bin/kafka-console-producer.sh --broker-list alihn1-opd-elk-01:19092 --topic kafkatest --producer.config ./config/client-ssl.properties
>aaa
>dddd
>ccc

# 加密消费
# ./bin/kafka-console-consumer.sh --bootstrap-server  alihn1-opd-elk-01:19092 --topic kafkatest --from-beginning --consumer.config ./config/client-ssl.properties
aaa
dddd
ccc

# 明文生产
# ./bin/kafka-console-producer.sh --broker-list alihn1-opd-elk-01:9092 --topic kafkatest
>aaaa
[2019-09-19 21:31:49,488] ERROR Error when sending message to topic kafkatest with key: null, value: 4 bytes with error: (org.apache.kafka.clients.producer.internals.ErrorLoggingCallback)
org.apache.kafka.common.errors.TimeoutException: Failed to update metadata after 60000 ms.
>[2019-09-19 21:32:56,831] WARN [Producer clientId=console-producer] Connection to node -1 could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)

# 明文消费
# ./bin/kafka-console-consumer.sh --bootstrap-server  alihn1-opd-elk-01:9092 --topic kafkatest --from-beginning
[2019-09-19 21:41:44,449] WARN [Consumer clientId=consumer-1, groupId=console-consumer-53660] Connection to node -1 could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
```
在 filebeat 服务器（阿里云内网）上进行测试
```
# 明文生产
# ./bin/kafka-console-producer.sh --broker-list alihn1-opd-elk-01:9092 --topic kafkatest
>ddd
>ff
>jad

# 明文消费
# ./bin/kafka-console-consumer.sh --bootstrap-server  alihn1-opd-elk-01:9092 --topic kafkatest --from-beginning
ddd
ff
jad
在 filebeat 服务器（阿里云内网）明文生产，在 logstash （本地内网）SSL 加密消费

# 阿里云主机明文生产
# ./bin/kafka-console-producer.sh --broker-list alihn1-opd-elk-01:9092 --topic kafkatest
>addfddd
>jjjj

# 本地内网主机加密消费
# ./bin/kafka-console-consumer.sh --bootstrap-server  alihn1-opd-elk-01:19092 --topic kafkatest --from-beginning --consumer.config ./config/client-ssl.properties
ddfddd
jjjj
```

## logstash input ssl 配置
修改 logstash 配置文件中 kafka input 部分，添加 ssl 相关参数
```
input {
  kafka {
    bootstrap_servers => "alihn1-opd-elk-01:19092"

    security_protocol => "SSL"
    ssl_keystore_location => "/sdata/usr/local/kafka/ca/client/client.keystore.jks"
    ssl_key_password => "passwd"
    ssl_keystore_password => "passwd"
    ssl_truststore_location => "/sdata/usr/local/kafka/ca/trust/client.truststore.jks"
    ssl_truststore_password => "passwd"

    topics => ["kafkatest"]
    codec => "json"
    decorate_events => true
    type => "kafkatest"
  }
}
```
重启 logstash，查看 logstash 启动日志
```
tail -f /sdata/var/log/logstash/logstash-star.log
[2019-09-18T04:00:17,232][INFO ][org.apache.kafka.clients.consumer.ConsumerConfig] ConsumerConfig values:
bootstrap.servers = [alihn1-opd-elk-01:19092]
    ssl.cipher.suites = null
    ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
    ssl.endpoint.identification.algorithm = https
    ssl.key.password = [hidden]
    ssl.keymanager.algorithm = SunX509
    ssl.keystore.location = /sdata/usr/local/kafka/ca/client/client.keystore.jks
    ssl.keystore.password = [hidden]
    ssl.keystore.type = JKS
    ssl.protocol = TLS
    ssl.provider = null
    ssl.secure.random.implementation = null
    ssl.trustmanager.algorithm = PKIX
    ssl.truststore.location = /sdata/usr/local/kafka/ca/trust/client.truststore.jks
    ssl.truststore.password = [hidden]
    ssl.truststore.type = JKS
```
logstash 能正常启动，启动日志能正常输出 kafka 相关配置参数则 ssl 认证配置成功。
