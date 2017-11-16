## 编译安装

```
wget http://tengine.taobao.org/download/tengine-2.2.0.tar.gz
tar xf tengine-2.2.0.tar.gz
yum -y install pcre-devel openssl openssl-devel
./configure
make
make install
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -s reload

```

```
curl 10.9.89.160
<!DOCTYPE html>
<html>
<head>
<title>Welcome to tengine!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to tengine!</h1>
<p>If you see this page, the tengine web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://tengine.taobao.org/">tengine.taobao.org</a>.</p>

<p><em>Thank you for using tengine.</em></p>
</body>
</html>
```

## 基础配置

```
worker_processes  2;
worker_connections  102400;
```

* 修改为服务器cpu核数
* 如果worker连接满了,nginx会启动保护机制,形成排队.

**客户端长连接**  
`keepalive_timeout 65; keepalive_requests 100;`

**优化keepalive\_requests**

```
keepalive_requests 8192;
```

> 这是一个客户端可以通过一个keepalive连接的请求次数。缺省值是100，但是也可以调得很高，而且这对于测试负载生成工具从哪里使用一个客户端发送这么多请求非常有用。  
> \* 设置通过“一个存活长连接”送达的最大请求数（默认是100，建议根据客户端在“keepalive”存活时间内的总请求数来设置）  
> \* 当送达的请求数超过该值后，该连接就会被关闭。

## 压测对比 {#nginx基准-压测对比}

nginx centos7.2 2核4G  
ab centos7.2 4核8G

| nginx版本 | 并发数 | 请求数 | 配置变更 | cpu idl | 内存 | 网络 | io | estab | timewait | 吞吐率 | 90% | 99% | Failed | 额外请求时间 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Tengine/2.2.0 | 100 | 5000000 | 非长连接 | 40% | 322M | 17M | 12M | 97 | 14013 | 9190.56 | 11ms | 13ms | 0 | 0.013s |
| Tengine/2.2.0 | 100 | 5000000 | 长连接 keepalive\_requests 100 | 1% | 323M | 50M | 27M | 101 | 12041 | 40225.05 | 3ms | 16ms | 0 | 0.018s |
| Tengine/2.2.0 | 100 | 5000000 | 长连接keepalive\_requests 8192 | 1% | 319M | 53M | 27M | 101 | 247 | 42131.36 | 3ms | 14ms | 0 | 0.009s |

## 结论

* 前端长连接,减少了tcp建立的消耗,提高了吞吐率,以及cpu利用率, 理想情况有400%+的提升
* 通过增加keepalive\_requests 在单节点发起高并发请求的场景下,有效控制了timewait数量.



