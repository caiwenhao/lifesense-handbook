### 监控分析

```
yum install dstat
dstat -tcdlnmpys --top-io --top-cpu
```

```
ss -a|grep ESTAB|grep http|wc -l 
```

### 抓包分析

tcpdump抓包分析  [http://blog.csdn.net/renrenhappy/article/details/5929702](http://blog.csdn.net/renrenhappy/article/details/5929702)

三次握手四次挥手  [http://blog.csdn.net/whuslei/article/details/6667471/](http://blog.csdn.net/whuslei/article/details/6667471/)



