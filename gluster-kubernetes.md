## gluster-kubernetes

```
   18  /usr/sbin/pvcreate --metadatasize=128M --dataalignment=256K /dev/vdb -ff -y
   19  umount /dev/mapper/vg_d6c4f964062532b6b2de9037c6b30dbd-brick_cc73e39dc3bde810972848df5f112178
   20  /usr/sbin/pvcreate --metadatasize=128M --dataalignment=256K /dev/vdb -ff -y
   21  unmount LV
   22  pvremove 
   23  pvremove  /dev/vdb
   24  pvremove  /dev/mapper/vg_d6c4f964062532b6b2de9037c6b30dbd-brick_cc73e39dc3bde810972848df5f112178
   25  df -h
   26  df -h
   27  fdisk -l
   28  vgscan
   29  vgdisplay -v vg_d6c4f964062532b6b2de9037c6b30dbd
   30  umount /dev/vg_d6c4f964062532b6b2de9037c6b30dbd/brick_cc73e39dc3bde810972848df5f112178 
   31  umount /dev/vdb    
   32  umounttp_cc73e39dc3bde810972848df5f112178
   33  umount tp_cc73e39dc3bde810972848df5f112178
   34  lvremove /dev/vdb     
   35  lvremove  /dev/vg_d6c4f964062532b6b2de9037c6b30dbd/brick_cc73e39dc3bde810972848df5f112178
   36  lvremove tp_cc73e39dc3bde810972848df5f112178
   37  vgremove vg_d6c4f964062532b6b2de9037c6b30dbd
   38  pvremove /dev/vdb 
   39  pvremove /dev/vdb 
   40  history 
[root@node1 /]# /usr/sbin/pvcreate --metadatasize=128M --dataalignment=256K /dev/vdb -ff -y
  Physical volume "/dev/vdb" successfully created.
[root@node1 /]# pvremove /dev/vdb 
  Labels on physical volume "/dev/vdb" successfully wiped.



heketi-cli  topology info|grep vdb|awk '{print $1}'|awk -F: '{print $2}'|xargs -i heketi-cli device delete {}
heketi-cli  topology info|grep "Node Id"|awk '{print $3}'|xargs -i heketi-cli node delete {}



apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: glusterfs-storage
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://10.233.68.7:8080"

heketi-cli -s http://10.233.68.7:8080 --user admin --secret '<ADMIN_KEY>' cluster list  


gluster-kubernetes

```



