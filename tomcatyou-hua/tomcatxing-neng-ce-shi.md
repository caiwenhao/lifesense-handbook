## 安装编译

```bash
wget http://apache.fayea.com/tomcat/tomcat-8/v8.0.44/bin/apache-tomcat-8.0.44.tar.gz
tar xf apache-tomcat-8.0.44.tar.gz
mv apache-tomcat-8.0.44 /data/
```

### 启动参数

```bash
/data/apache-tomcat-8.0.44/bin/catalina.sh
JAVA_HOME=/data/jdk-1.8.0/
CATALINA_HOME=/data/apache-tomcat-8.0.44/
CATALINA_OPTS="-Djava.security.egd=file:/dev/./urandom  -Dfile.encoding=UTF-8 -server -Xms2048m -Xmx2048m -Xmn1024m -XX:PermSize=256m -XX:MaxPermSize=512m -XX:SurvivorRatio=10 -XX:MaxTenuringThreshold=15 -XX:NewRatio=2 -XX:+DisableExp
licitGC"
CATALINA_PID=$CATALINA_HOME/catalina.pid
```

### 测试页面

```
/data/apache-tomcat-8.0.44/webapps/ROOT/time.jsp 
```

```jsp
<%@ page session="false" pageEncoding="UTF-8" contentType="text/html; charset=UTF-8" %>
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>JSPf5h/</title>
    </head>
    <body>
        <% java.text.SimpleDateFormat sdf = new java.text.SimpleDateFormat("yyyy-MM-dd HH:mm:ss"); %>
        <%=sdf.format(new java.util.Date()) %>
    </body>
</html>
```

## 优化

### 连接池

```xml
<Executor
        name="tomcatThreadPool"
        namePrefix="catalina-exec-"
        maxThreads="500"
        minSpareThreads="30"
        maxIdleTime="60000"
        prestartminSpareThreads = "true"
        maxQueueSize = "100"
/>
```

* maxThreads，最大并发数，默认设置 200，一般建议在 500 ~ 800，根据硬件设施和业务来判断
* minSpareThreads，Tomcat 初始化时创建的线程数，默认设置 25
* prestartminSpareThreads，在 Tomcat 初始化的时候就初始化 minSpareThreads 的参数值，如果不等于 true，minSpareThreads 的值就没啥效果了
* maxQueueSize，最大的等待队列数，超过则拒绝请求
* maxIdleTime，如果当前线程大于初始化线程，那空闲线程存活的时间，单位毫秒，默认60000=60秒=1分钟。

### 链接参数

```xml
<Connector 
   executor="tomcatThreadPool"
   port="8080" 
   protocol="org.apache.coyote.http11.Http11Nio2Protocol" 
   connectionTimeout="20000" 
   maxConnections="10000" 
   redirectPort="8443" 
   enableLookups="false" 
   acceptCount="100" 
   maxPostSize="10485760" 
   maxHttpHeaderSize="8192" 
   compression="on" 
   disableUploadTimeout="true" 
   compressionMinSize="2048" 
   acceptorThreadCount="2" 
   compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript" 
   URIEncoding="utf-8"
/>
```

* protocol，Tomcat 8 设置 nio2 更好：org.apache.coyote.http11.Http11Nio2Protocol（如果这个用不了，就用下面那个）
* protocol，Tomcat 6、7 设置 nio 更好：org.apache.coyote.http11.Http11NioProtocol
* enableLookups，禁用DNS查询
* acceptCount，指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理，默认设置 100
* maxPostSize，以 FORM URL 参数方式的 POST 提交方式，限制提交最大的大小，默认是 2097152\(2兆\)，它使用的单位是字节。10485760 为 10M。如果要禁用限制，则可以设置为 -1。
* acceptorThreadCount，用于接收连接的线程的数量，默认值是1。一般这个指需要改动的时候是因为该服务器是一个多核CPU，如果是多核 CPU 一般配置为 2.
* maxHttpHeaderSize，http请求头信息的最大程度，超过此长度的部分不予处理。一般8K。

### 禁用AJP

```
<!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->
```

### jvm优化

```
CATALINA_OPTS="-Dfile.encoding=UTF-8 -server -Xms2048m -Xmx2048m -Xmn1024m -XX:PermSize=256m -XX:MaxPermSize=512m -XX:SurvivorRatio=10 -XX:MaxTenuringThreshold=15 -XX:NewRatio=2 -XX:+DisableExplicitGC"
```

> -Dfile.encoding：默认文件编码  
> -server：表示这是应用于服务器的配置，JVM 内部会有特殊处理的  
> -Xmx1024m：设置JVM最大可用内存为1024MB  
> -Xms1024m：设置JVM最小内存为1024m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。  
> -Xmn1024m：设置JVM新生代大小（JDK1.4之后版本）。一般-Xmn的大小是-Xms的1/2左右，不要设置的过大或过小，过大导致老年代变小，频繁Full GC，过小导致minor GC频繁。如果不设置-Xmn，可以采用-XX:NewRatio=2来设置，也是一样的效果  
> -XX:NewSize：设置新生代大小  
> -XX:MaxNewSize：设置最大的新生代大小  
> -XX:PermSize：设置永久代大小  
> -XX:MaxPermSize：设置最大永久代大小  
> -XX:NewRatio=4：设置年轻代（包括 Eden 和两个 Survivor 区）与终身代的比值（除去永久代）。设置为 4，则年轻代与终身代所占比值为 1：4，年轻代占整个堆栈的 1/5  
> -XX:MaxTenuringThreshold=10：设置垃圾最大年龄，默认为：15。如果设置为 0 的话，则年轻代对象不经过 Survivor 区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在 Survivor 区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论。需要注意的是，设置了 -XX:MaxTenuringThreshold，并不代表着，对象一定在年轻代存活15次才被晋升进入老年代，它只是一个最大值，事实上，存在一个动态计算机制，计算每次晋入老年代的阈值，取阈值和MaxTenuringThreshold中较小的一个为准。  
> -XX:+DisableExplicitGC：这个将会忽略手动调用 GC 的代码使得 System.gc\(\) 的调用就会变成一个空调用，完全不会触发任何 GC
>
> 参考  
> [https://github.com/judasn/Linux-Tutorial/blob/master/Tomcat-Install-And-Settings.md](https://github.com/judasn/Linux-Tutorial/blob/master/Tomcat-Install-And-Settings.md)

## 压测

### nginx代理tomcat

```
upstream http_backend {
        server 10.9.147.96:8080;
        keepalive 96;
    }
server {
    listen          80 ;
    server_name     10.9.89.160;
    charset         utf-8;
    index index.html index.htm index.jsp;
    location / {
        proxy_set_header Connection "";
        proxy_http_version 1.1;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass  http://http_backend;
    }
}
```

### 对齐nginx与tomcat的长连接超时

```
       keepAliveTimeout="650000"
       maxKeepAliveRequests="5000"
```

> 如果nginx 与tomcat的 keepAliveTimeout 时间不等,会导致过多timewait.

### 压测环境

nginx centos7.2 2核4G  
tomcat centos7.2 2核4G  
ab centos7.2 4核8G

### 压测结果

| tomcat版本 | 并发数 | 请求数 | 配置变更 | cpu idl | 内存 | 网络 | io | estab | timewait | 吞吐率 | 90% | 99% | Failed | 额外请求时间 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| tomcat8.0 | 100 | 5000000 | 默认配置长连接 | 8% | 1374M | 9250k | 9537k | 101 | 14017 | 7847.25 | 6ms | 8ms | 0 | 0.018s |
| tomcat8.0 | 100 | 5000000 | 优化配置Nio2 短连接 | 4% | 1369M | 8002k | 4932k | 101 | 14030 | 4000.17 | 12ms | 21ms | 0 | 0.024s |
| tomcat8.0 | 100 | 5000000 | 优化配置Nio2 长连接 | 14% | 1379M | 8126k | 8349k | 101 | 14016 | 6823.92 | 7ms | 15ms | 0 | 0.026s |
| tomcat8.0 | 100 | 5000000 | 优化配置apr 长连接 | 9% | 1314M | 11M | 11M | 101 | 14016 | 8381.62 | 5ms | 9ms | 0 | 0.021s |

  


| 版本 | 并发数 | 请求数 | 配置变更 | cpu idl | 内存 | 网络 | io | estab | timewait | 吞吐率 | 90% | 99% | Failed | 额外请求时间 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Tengine/2.2.0 | 100 | 5000000 | 短连接 | 54% | 328M | 8349k | 5376k | 187 | 16384 | 3340.96 | 14ms | 35ms | 0 | 0.014s |
|  |  |  |  |  |  |  |  | tomcat 8.0  | 1% | 1674M | 6076k | 7918k | 101 | 0 |

  


| 版本 | 并发数 | 请求数 | 配置变更 | cpu idl | 内存 | 网络 | io | estab | timewait | 吞吐率 | 90% | 99% | Failed | 额外请求时间 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Tengine/2.2.0 | 100 | 5000000 | 长连接keepalive 96 | 25% | 325M | 13M | 5376k | 199 | 16384 |  7418.06 | 6ms | 8ms | 0 | 0.016s |
|  |  |  |  |  |  |  |  | tomcat 8.0  | 12% | 1654M | 8132k | 8288k | 101 | 21 |

  


| 版本 | 并发数 | 请求数 | 配置变更 | cpu idl | 内存 | 网络 | io | estab | timewait | 吞吐率 | 90% | 99% | Failed | 额外请求时间 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Tengine/2.2.0 | 100 | 5000000 | 长连接keepalive 96keepAliveTimeout="650000"maxKeepAliveRequests="5000" | 25% | 322M | 14M | 7530k | 204 | 592 |  7780.64 | 6ms | 9ms | 0 | 0.016s |
|  |  |  |  |  |  |  |  | tomcat 8.0  | 12% | 1298M | 9913k | 10M | 104 | 0 |



> 1. nginx 与tomcat 长连接 取决于 upstream中的keepalive 参数
> 2. nginx 与tomcat 长连接相较短链接, 在服务器性能没达到瓶颈的情况下, 最高可以提高1倍的性能
> 3. nginx 与tomcat 对齐 长连接超时与及加大 maxKeepAliveRequests后, 能够有效解决 高并发下出现 大量timewait的情况



