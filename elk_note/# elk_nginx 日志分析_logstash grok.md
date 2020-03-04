# elk_nginx 日志分析_logstash grok
## nginx log
nginx log config
```
/sdata/etc/nginx/nginx.conf

    access_log /sdata/var/log/nginx/access.log;
    error_log /sdata/var/log/nginx/error.log;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent $request_time "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```
Sample Data
```
117.136.76.110 - - [04/Mar/2020:17:36:21 +0800] "GET /service/sensors.min.js HTTP/1.1" 304 0 0.042 "https://domainname.com/other/wchat/chat.html?path=chatWindow&chatType=24&TimePopUp=&evaluationPopUp=1&userId=x®Type=x®Id=abcdes&_timestamp=1583314579343" "Mozilla/5.0 (Linux; Android 9; COL-AL10 Build/HUAWEICOL-AL10; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/66.0.3359.126 MQQBrowser/6.2 TBS/045018 Mobile Safari/537.36 MMWEBID/4794 MicroMessenger/7.0.10.1580(0x27000AFE) Process/tools NetType/4G Language/zh_CN ABI/arm64" "117.136.76.110"
```

## logstash grok
Grok Pattern
```
%{IPORHOST:client_ip} - (%{USERNAME:user}|-) \[%{HTTPDATE:timestamp}\] "(?:%{WORD:request_method} %{NOTSPACE:request}(?: HTTP/%{NUMBER:http_version})?|%{DATA:raw_request})" %{NUMBER:status} %{NUMBER:response_size} %{NUMBER:request_time} "%{NOTSPACE:url_referer}" "%{GREEDYDATA:user_agent}"
```
Structured Data
```
{
  "request": "/service/sensors.min.js",
  "url_referer": "https://domainname.com/other/wchat/chat.html?path=chatWindow&chatType=24&TimePopUp=&evaluationPopUp=1&userId=x®Type=x®Id=abcdes&_timestamp=1583314579343",
  "http_version": "1.1",
  "request_method": "GET",
  "response_size": "0",
  "request_time": "0.042",
  "client_ip": "117.136.76.110",
  "user": "-",
  "user_agent": "Mozilla/5.0 (Linux; Android 9; COL-AL10 Build/HUAWEICOL-AL10; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/66.0.3359.126 MQQBrowser/6.2 TBS/045018 Mobile Safari/537.36 MMWEBID/4794 MicroMessenger/7.0.10.1580(0x27000AFE) Process/tools NetType/4G Language/zh_CN ABI/arm64\" \"117.136.76.110",
  "timestamp": "04/Mar/2020:17:36:21 +0800",
  "status": "304"
}
```


## elasticsearch output
```
{
  "_index": "logstash-prd-nginx-2020.03.04",
  "_type": "_doc",
  "_id": "xHzopHABiZWrcNp2HMOV",
  "_version": 1,
  "_score": null,
  "_source": {
    "type": "prd-nginx",
    "tags": [
      "alihn1",
      "prd",
      "nginx",
      "access",
      "log"
    ],
    "@timestamp": "2020-03-04T09:36:22.205Z",
    "host": {
      "name": "alihn1-prd-nginx-01.snail"
    },
    "agent": {
      "hostname": "alihn1-prd-nginx-01.snail",
      "type": "filebeat",
      "id": "f8063e8e-8b0a-4850-9d7c-8d259b36141b",
      "ephemeral_id": "0d1b43a5-85c0-4649-8b95-4ae76dc322b0",
      "version": "7.1.0"
    },
    "log": {
      "file": {
        "path": "/sdata/var/log/nginx/domainname.com_access.log"
      },
      "offset": 227326061
    },
    "input": {
      "type": "log"
    },
    "request": "/service/sensors.min.js",
    "request_method": "GET",
    "request_time": 0.042,
    "ecs": {
      "version": "1.0.0"
    },
    "http_version": "1.1",
    "client_ip": "117.136.76.110",
    "timestamp": "04/Mar/2020:17:36:21 +0800",
    "user_agent": "Mozilla/5.0 (Linux; Android 9; COL-AL10 Build/HUAWEICOL-AL10; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/66.0.3359.126 MQQBrowser/6.2 TBS/045018 Mobile Safari/537.36 MMWEBID/4794 MicroMessenger/7.0.10.1580(0x27000AFE) Process/tools NetType/4G Language/zh_CN ABI/arm64\" \"117.136.76.110",
    "url_referer": "https://domainname.com/other/wchat/chat.html?path=chatWindow&chatType=24&yuyueTimePopUp=&evaluationPopUp=1&userId=x®Type=x®Id=abcdes&_timestamp=1583314579343",
    "message": "117.136.76.110 - - [04/Mar/2020:17:36:21 +0800] \"GET /service/sensors.min.js HTTP/1.1\" 304 0 0.042 \"https://domainname.com/other/wchat/chat.html?path=chatWindow&chatType=24&yuyueTimePopUp=&evaluationPopUp=1&userId=x®Type=x®Id=abcdes&_timestamp=1583314579343\" \"Mozilla/5.0 (Linux; Android 9; COL-AL10 Build/HUAWEICOL-AL10; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/66.0.3359.126 MQQBrowser/6.2 TBS/045018 Mobile Safari/537.36 MMWEBID/4794 MicroMessenger/7.0.10.1580(0x27000AFE) Process/tools NetType/4G Language/zh_CN ABI/arm64\" \"117.136.76.110\"",
    "status": "304",
    "fields": {
      "log_source": "prd-nginx-01"
    },
    "response_size": "0",
    "geoip": {
      "timezone": "Asia/Shanghai",
      "latitude": 34.7725,
      "country_code2": "CN",
      "country_code3": "CN",
      "longitude": 113.7266,
      "coordinates": [
        113.7266,
        34.7725
      ],
      "country_name": "China",
      "ip": "117.136.76.110",
      "continent_code": "AS",
      "location": {
        "lon": 113.7266,
        "lat": 34.7725
      }
    },
    "@version": "1",
    "user": "-"
  },
  "fields": {
    "@timestamp": [
      "2020-03-04T09:36:22.205Z"
    ]
  },
  "highlight": {
    "log.file.path": [
      "/@kibana-highlighted-field@sdata@/kibana-highlighted-field@/@kibana-highlighted-field@var@/kibana-highlighted-field@/@kibana-highlighted-field@log@/kibana-highlighted-field@/@kibana-highlighted-field@nginx@/kibana-highlighted-field@/@kibana-highlighted-field@domainname.com_access.log@/kibana-highlighted-field@"
    ]
  },
  "sort": [
    1583314582205
  ]
}
```
