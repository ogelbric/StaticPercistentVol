# Static Persistent Volume 

**Get the current storage class**
```
[root@localhost ~]# kubectl get storageclass
NAME                          PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
pacific-gold-storage-policy   csi.vsphere.vmware.com   Delete          Immediate           false                  97m
[root@localhost ~]# 
```

**Create the persitent volume YAML**
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: pacific-gold-storage-policy
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

```
[root@localhost ~]# kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS                  REASON   AGE
task-pv-volume   4Gi        RWO            Retain           Bound    default/task-pv-claim   pacific-gold-storage-policy            36m
[root@localhost ~]# 
```

**Create the persistent volume claim YAML**
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: pacific-gold-storage-policy
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

```
[root@localhost ~]# kubectl get pvc
NAME            STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS                  AGE
task-pv-claim   Bound    task-pv-volume   4Gi        RWO            pacific-gold-storage-policy   35m
[root@localhost ~]#
```

