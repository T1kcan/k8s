
NFS Server Installation on CentOS
--------------
```bash
yum install nfs-utils
```
--------------
Create Serving Directory:
```bash
mkdir /nfs
--------------
chmod -R 755 /nfs
chown nfsnobody:nfsnobody /nfs
```
Start NFS Server Processes:
```bash
--------------
systemctl enable rpcbind
systemctl enable nfs-server
systemctl enable nfs-lock
systemctl enable nfs-idmap
systemctl start rpcbind
systemctl start nfs-server
systemctl start nfs-lock
systemctl start nfs-idmap
--------------
nano /etc/exports
/nfs *(rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)
--------------
systemctl restart nfs-server
--------------
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --permanent --zone=public --add-service=mountd
firewall-cmd --permanent --zone=public --add-service=rpc-bind
firewall-cmd --reload
```
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
   name: postgres-pv
   labels:
     app: postgres
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  mountOptions:
   - nfsvers=3
  nfs:
    server: 192.168.99.124 # Replace with your own NFS server address
    path: /nfs
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-claim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: ""
  selector:
    matchLabels:
      app: postgres
---
apiVersion: v1
kind: Pod
metadata:
  name: postgres-pod
  labels:
    app: postgres
spec:
  containers:
  - name: postgres-cntr
    image: postgres:latest
    env:
    - name: POSTGRES_USER
      value: postgresadmin
    - name: POSTGRES_DB
      value: postgresdb
    - name: POSTGRES_PASSWORD
      value: onetwothree123
    ports:
    - containerPort: 5432
    volumeMounts:
    - name: postgres-volume
      mountPath: "/var/lib/postgresql/data"
  volumes:
  - name: postgres-volume
    persistentVolumeClaim:
      claimName: postgres-claim
```
```bash
kubectl exec -it postgres-pod -- bash
psql -h localhost -U postgresadmin --password postgresdb
# paste Password: onetwothree123
CREATE DATABASE myprojectdatabase;
\list
exit
exit
```
Delete pv,pvc,pod
Delete Minikube Cluster
Recreate pv,pvc,pod
Observe that your database persist
kubectl port-forward postgres-pod 8080:5432
Use pgAdmin to connect db from localhost port 8080
