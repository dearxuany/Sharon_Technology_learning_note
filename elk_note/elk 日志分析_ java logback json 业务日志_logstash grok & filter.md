# elk 日志分析_ java logback json 业务日志_logstash grok & filter
## java logback json 日志配置
### logback 配置
* 业务日志以 json 格式输出，方便 logstash 拆分解析
* 使用 utf8 编码，防止中文被输出为十六进程编码
* 日志每 100MB 进行自动切分，防止单个日志过大导致日志原文文本文件难以打开及备份
* 日志名按照 gitlab 上标示的工程名为准，但必须以 *.log 格式命名，所有实时日志必须一直输出在该日志中，防止elk收集日志使用正则匹配文件名会导致性能损耗，日志实时收集延时
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="10 seconds">

    <!-- 对应pattern的msg -->
    <conversionRule conversionWord="message" converterClass="com.companyname.common.log.filter.CustomConverter"/>

    <property name="log_path" value="/sdata/var/log"/>
    <property name="service_name" value="companyname-message-service"/>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{60} - %message%n</pattern>
            <!--<charset>UTF-8</charset>-->
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log_path}/${service_name}/${service_name}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${log_path}/${service_name}/${service_name}.%d{yyyy-MM-dd}-%i.log</FileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>365</maxHistory>
            <totalSizeCap>50GB</totalSizeCap>
        </rollingPolicy>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <jsonFactoryDecorator class="com.companyname.common.util.DefaultJsonFactoryDecorator"/>
            <providers>
                <pattern>
                    <pattern>
                        {
                        "timestamp":"%d{yyyy-MM-dd HH:mm:ss.SSS}",
                        "level": "%level",
                        "thread": "%thread",
                        "class": "%logger{60}",
                        "message": "%message",
                        "service_name": "${service_name}",
                        "stack_trace": "%exception{10}"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```
### 日志级别
#### inteceptor
1. ip记录 - INFO</br>
#### api
1. 入参校验失败结果记录 - WARN</br>
2. 入参校验成功的参数内容记录 - INFO</br>
3. 没有入参的成功进入controller记录 - INFO</br>
#### service
1. 捕获异常日志记录 - ERROR</br>
2. 业务逻辑校验不通过 - WARN</br>
3. 参数校验为空，走默认策略且需记录的情况 - WARN</br>
4. 信息记录 - INFO</br>
#### component
1. 捕获异常日志记录 - ERROR</br>
2. 入参或业务逻辑校验不通过 - WARN</br>
3. 参数校验为空，走默认策略且需记录的情况 - WARN</br>
4. 信息记录 - INFO</br>
#### util
1. 捕获异常日志记录 - ERROR</br>
2. 参数校验为空，走默认策略且需记录的情况 - WARN</br></br>
3. 信息记录 - INFO</br>

