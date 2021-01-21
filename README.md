# Static Persistent Volume 

**Get the current storage class**
```
[root@localhost ~]# kubectl get storageclass
NAME                          PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
pacific-gold-storage-policy   csi.vsphere.vmware.com   Delete          Immediate           false                  97m
[root@localhost ~]# 
```

**Create the persitent volume YAML and apply**
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

**Create the persistent volume claim YAML and apply**
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

**Sample nginx YAML (note image path needs to be updated!!!)**
```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: nginx
  name: nginx
spec:
  ports:
    - port: 80
  selector:
    app: nginx
  type: LoadBalancer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: gcr.io/my-repo-here/nginx
        ports:
        - containerPort: 80
        volumeMounts:
          - name: demo-vol
            mountPath: /mnt/
      volumes:
        - name: demo-vol
          persistentVolumeClaim:
            claimName: task-pv-claim
```

**Test with nginx yaml**
```
[root@localhost ~]# kubectl apply -f ./pvc-google-nginx-lbsvcGA.yaml 
service/nginx created
deployment.apps/nginx created

[root@localhost ~]# kubectl get pods 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-75b5b44df9-dmhfw   1/1     Running   0          20m

[root@localhost ~]# kubectl exec -it nginx-75b5b44df9-dmhfw bash
root@nginx-75b5b44df9-dmhfw:/# cd /mnt
root@nginx-75b5b44df9-dmhfw:/mnt# ls -ltr
total 4
-rw-r--r-- 1 root root  0 Jan 21 17:43 abc
-rw-r--r-- 1 root root 58 Jan 21 17:55 def
root@nginx-75b5b44df9-dmhfw:/mnt# cat def
I was here
I was here again 
Thu Jan 21 17:55:05 UTC 2021
root@nginx-75b5b44df9-dmhfw:/mnt# echo "I was here again and again " >> def
root@nginx-75b5b44df9-dmhfw:/mnt# date >> def
root@nginx-75b5b44df9-dmhfw:/mnt# exit
exit
[root@localhost ~]# kubectl delete -f ./pvc-google-nginx-lbsvcGA.yaml 
service "nginx" deleted
deployment.apps "nginx" deleted
[root@localhost ~]# kubectl get pods
NAME                     READY   STATUS        RESTARTS   AGE
nginx-75b5b44df9-dmhfw   0/1     Terminating   0          23m
[root@localhost ~]# kubectl get pods
NAME                     READY   STATUS        RESTARTS   AGE
nginx-75b5b44df9-dmhfw   0/1     Terminating   0          23m
[root@localhost ~]# kubectl get pods
NAME                     READY   STATUS        RESTARTS   AGE
nginx-75b5b44df9-dmhfw   0/1     Terminating   0          23m
[root@localhost ~]# kubectl get pods
No resources found in default namespace.
[root@localhost ~]# kubectl apply -f ./pvc-google-nginx-lbsvcGA.yaml 
service/nginx created
deployment.apps/nginx created
[root@localhost ~]# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-75b5b44df9-mzntx   1/1     Running   0          7s
[root@localhost ~]# kubectl exec -it nginx-75b5b44df9-mzntx bash
root@nginx-75b5b44df9-mzntx:/# cd /mnt
root@nginx-75b5b44df9-mzntx:/mnt# cat def
I was here
I was here again 
Thu Jan 21 17:55:05 UTC 2021
I was here again and again 
Thu Jan 21 18:20:03 UTC 2021
root@nginx-75b5b44df9-mzntx:/mnt# 

```
