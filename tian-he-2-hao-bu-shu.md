## 天河2号部署

### disconf

```
helm install disconf/ --set zookeeper.storageClass=gluster-heketi --set mysql.persistence.storageClass=gluster-heketi --namespace=disconf
mysql -h 10.233.70.13 -uroot -pVCOZZZlwbeK3EMah disconf  < /tmp/disconf_20171113.sql
```

### 业务集群

```
helm install lifesense --namespace=lifesense-th2 --set configmap.DISCONF_HOST=knobby-umbrellabird-disconf.disconf
helm install kafka/ --namespace=lifesense-th2 --set storageClass=gluster-heketi --set zookeeper.storageClass=gluster-heketi
```

### 配置端口映射

```

```



