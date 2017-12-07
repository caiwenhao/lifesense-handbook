## 天河2号部署

### disconf

```
helm install disconf/ --set zookeeper.storageClass=gluster-heketi --set mysql.persistence.storageClass=gluster-heketi \
 --namespace=disconf
```

```bash
#配置mysql权限
CREATE USER 'disconf'@'%' IDENTIFIED BY 'CCmRPz4Eeergp7Ni';
GRANT ALL ON *.* TO 'disconf'@'%';
#导入数据
mysql -h 10.233.69.21 -u disconf -p disconf < /tmp/disconf_20171207.sql
```

### 业务集群

```
helm install lifesense --namespace=lifesense-th2 --set configmap.DISCONF_HOST=knobby-umbrellabird-disconf.disconf
helm install kafka/ --namespace=lifesense-th2 --set storageClass=gluster-heketi \
 --set zookeeper.storageClass=gluster-heketi
```

### 配置端口映射

```
/etc/rinetd.conf
0.0.0.0 80 10.181.7.37 80
```



