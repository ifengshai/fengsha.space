---
title: test_ApacheAb
date: 2023-01-28 05:25:48
permalink: /pages/d80ead/
categories:
  - zh
  - study
tags:
  - 
author: 
  name: fengsha
  link: https://github.com/ifengshai/
---
# PHP性能优化

### php测试工具Apache自带的ab测试工具
> ab.exe -n100 -c77 http://www.baidu.com/index.html
-n表示请求数，-c并发数，url目标测压地址
###测试结果
~~~

Server Software:		BWS/1.1	(服务器软件名称及版本信息)
Server Hostname:        www.baidu.com	(服务器主机名)
Server Port:            80	(目标端口)

Document Path:          /index.html	(供测试的URL路径)
Document Length:        0 bytes	(供测试的URL返回的文档大小)

Concurrency Level:      77	(并发数)
Time taken for tests:   2.167 seconds	(压力测试消耗的总时间)
Complete requests:      100	(压力测试的总次数)
Failed requests:        39	(失败的请求数)
   (Connect: 0, Receive: 0, Length: 39, Exceptions: 0)
Total transferred:      9921054 bytes	(传输的总数据量)
HTML transferred:       9869850 bytes	(HTML文档的总数据量)
Requests per second:    46.15 [#/sec] (mean)	(平均每秒的请求数)
Time per request:       1668.536 [ms] (mean)	(所有并发用户都请求一次的平均时间)
Time per request:       21.669 [ms] (mean, across all concurrent requests)	(单个用户请求一次的平均时间)
Transfer rate:          4471.09 [Kbytes/sec] received	(传输速率，单位：KB/s)

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       16   20   2.5     20      27
Processing:    27  967 534.0   1034    2065
Waiting:       23  644 459.4    543    1538
Total:         47  988 533.9   1054    2085

Percentage of the requests served within a certain time (ms)
  50%   1054
  66%   1335
  75%   1413
  80%   1449
  90%   1573
  95%   1870
  98%   2022
  99%   2085
 100%   2085 (longest request)

~~~