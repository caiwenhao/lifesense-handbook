# rinetd {#title-text}

> Linux下做地址NAT有很多种方法。比如haproxy、nginx的4层代理，linux自带的iptables等都能实现。haproxy、nginx就不说了，配置相对简单；iptables配置复杂，概念也比较多DNAT、SNAT、PREROUTING、POSTROUTING等等。其实，Linux下有一个叫rinetd的工具，安装简单，配置也不复杂。
>
> ucloud 跨域通道目前只支持同项目联通, 为解决跨项目联通问题, 这里使用rinetd做端口转发

**安装**

```
wget http://www.boutell.com/rinetd/http/rinetd.tar.gz
tar xf rinetd.tar.gz
cd rinetd
sed -i's/65536/65535/g' rinetd.c
mkdir /usr/man/make && make install
vim  /etc/rinetd.conf
rinetd -c /etc/rinetd.conf
```

**/etc/rinetd.conf**

```
0.0.0.0 80 10.181.7.37 80
```



