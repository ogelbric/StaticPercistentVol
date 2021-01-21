# Static Persistent Volume 

**Get the current storage class**
```
[root@localhost ~]# kubectl get storageclass
NAME                          PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
pacific-gold-storage-policy   csi.vsphere.vmware.com   Delete          Immediate           false                  97m
[root@localhost ~]# 
```
