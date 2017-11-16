## 安装与命令

> ab是Apache超文本传输协议\(HTTP\)的性能测试工具。其设计意图是描绘当前所安装的Apache的执行性能，主要是显示你安装的Apache每秒可以处理多少个请求。

```
yum install httpd-tools
ab -r -n 10000000 -c 100 http://10.9.89.160
ab -r -k -n 2 -c 1 http://10.9.89.160/
ab -H "Connection：Keep-Alive" -k -r -n 2 -c 1 http://10.9.89.160/

```

> -k 是保持长连接, 从nginx日志看虽然依旧走http1.0 但抓抓包分析,连接是有复用的

## 结果解析

```
This is ApacheBench, Version 2.3 
<
$Revision: 1430300 $
>

Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.9.89.160 (be patient)
Completed 500000 requests
Completed 1000000 requests
Completed 1500000 requests
Completed 2000000 requests
Completed 2500000 requests
Completed 3000000 requests
Completed 3500000 requests
Completed 4000000 requests
Completed 4500000 requests
Completed 5000000 requests
Finished 5000000 requests


Server Software:        Tengine/2.2.0
Server Hostname:        10.9.89.160
Server Port:            80

Document Path:          /
Document Length:        555 bytes

Concurrency Level:      100
Time taken for tests:   94.273 seconds
Complete requests:      5000000
Failed requests:        0
Write errors:           0
Keep-Alive requests:    4950051
Total transferred:      3969750255 bytes
HTML transferred:       2775000000 bytes
Requests per second:    53037.18 [#/sec] (mean)
Time per request:       1.885 [ms] (mean)
Time per request:       0.019 [ms] (mean, across all concurrent requests)
Transfer rate:          41121.95 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0      24
Processing:     0    2   3.2      1     166
Waiting:        0    2   3.2      1     166
Total:          0    2   3.2      1     167

Percentage of the requests served within a certain time (ms)
  50%      1
  66%      1
  75%      2
  80%      2
  90%      3
  95%      5
  98%     11
  99%     17
 100%    167 (longest request)

```

### 服务器信息

```
Server Software:        Tengine/2.2.0
Server Hostname:        10.9.89.160
Server Port:            80

```

### 请求文档信息

```
Document Path:          /
Document Length:        555 bytes

```

### 一般指标

```
Concurrency Level:      100
Time taken for tests:   94.273 seconds
Complete requests:      5000000
Failed requests:        0
Write errors:           0
Keep-Alive requests:    4950051
Total transferred:      3969750255 bytes
HTML transferred:       2775000000 bytes

```

* Concurrency Level 请求并发数
* Time taken for tests 本次压测所花费的总时间
* Complete requests 完成的请求次数
* Failed requests 失败请求次数
* Write errors 写入错误数
* Keep-Alive requests 长连接请求数
* Total transferred 整个压测场景的网络传输量
* HTML transferred 整个压测场景的html传输量

### 核心指标

```
Requests per second:    53037.18 [#/sec] (mean)
Time per request:       1.885 [ms] (mean)
Time per request:       0.019 [ms] (mean, across all concurrent requests)
Transfer rate:          41121.95 [Kbytes/sec] received

```

* Requests per second 吞吐率/每秒事务数
* Time per request 用户平均请求等待时间/平均事务响应时间
* Time per request\(across all concurrent requests\) 服务器平均请求处理时间
* Transfer rate 平均每秒网络上的流量

### 响应时间分布

```
Percentage of the requests served within a certain time (ms)
  50%      1
  66%      1
  75%      2
  80%      2
  90%      3
  95%      5
  98%     11
  99%     17
 100%    167 (longest request)

```

* 每个请求处理时间的分布情况,80%的处理时间在2ms内
* 重要的是看90%的处理时间



