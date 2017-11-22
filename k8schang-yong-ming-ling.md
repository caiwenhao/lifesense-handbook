## k8s常用命令

```
kubectl patch storageclass gluster-heketi -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'     
```



