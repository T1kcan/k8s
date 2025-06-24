

# Static Volume Provisioning
Create Persistent Volume on Minikube Cluster
## pv-hostpath.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:                      # single-node cluster-test purpose
    path: "/mnt/data"
```

```bash
$ kubectl apply -f pv-hostpath.yaml 
```

Create Persistent Volume Claim to use Persistent Volume that we created
## pvc-hostpath.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostpath-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ""
```
```txt
  # storageClassName: "" is used to bind to a PersistentVolume without a StorageClass
  If you want to use a specific StorageClass, replace "" with the name of the StorageClass
  storageClassName: my-storage-class
  If you want to use dynamic provisioning, specify a StorageClass that supports it
  storageClassName: my-dynamic-storage-class
  If you want to use a specific PersistentVolume, specify its name in the selector
  selector:
    matchLabels:
      type: hostpath
```
- Now apply `kubectl` to these files.
```bash
kubectl apply -f pv-hostpath.yaml
#persistentvolume/hostpath-pv created
kubectl apply -f pvc-hostpath.yaml
persistentvolumeclaim/hostpath-pvc created
```
- Now let's create pods to test them out.
## pod-pvc1.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-pvc1
# This pod uses a PersistentVolumeClaim to access storage
spec:
  containers:
    - name: mycontainer
      image: busybox
      command: ["sh", "-c", "echo Hello from storage > /data/hello.txt && sleep 3600"]
      volumeMounts:
        - mountPath: "/data"
          name: hostpath-storage
  volumes:
    - name: hostpath-storage
      persistentVolumeClaim:
        claimName: hostpath-pvc
  restartPolicy: Never
```
## pod-pvc2.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-pvc2
# This pod uses a PersistentVolumeClaim to access storage
spec:
  containers:
    - name: mycontainer
      image: busybox
      command: ["sh", "-c", "echo Hello from second pod >> /data/hello.txt && sleep 3600"]
      volumeMounts:
        - mountPath: "/data"
          name: hostpath-storage
  volumes:
    - name: hostpath-storage
      persistentVolumeClaim:
        claimName: hostpath-pvc
  restartPolicy: Never
```
- Now apply `kubectl` to these files.
```bash 
$ kubectl apply -f pod-pvc1.yaml 
#pod/pod-pvc1 created
$ kubectl exec -it pod-pvc1 -- sh
/ # ls /data
hello.txt
/ # cat /data/hello.txt
Hello from storage
touch /data/test.txt
ls /data
exit
kubectl apply -f pod-pvc2.yaml
$ kubectl exec -it pod-pvc2 -- sh
ls /data
hello.txt  test.txt
cat /data/hello.txt
Hello from storage
Hello from second pod
exit
```
# Dynamic Volume
```bash
$ kubectl apply -f pvc-standard.yaml 
#persistentvolumeclaim/dynamic-pvc created
$ kubectl get pvc
$ kubectl describe pvc dynamic-pvc
$ kubectl apply -f pod-pvc.yaml 
#pod/dynamic-pvc-pod created
$ kubectl get pod
#NAME              READY   STATUS    RESTARTS   AGE
#dynamic-pvc-pod   1/1     Running   0          6s
$ kubectl exec -it dynamic-pvc-pod -- sh
ls /data
#hello.txt
echo test >/data/test.txt
ls /data
# hello.txt  test.txt
$ exit
$ kubectl apply -f pod2-pvc.yaml 
pod/dynamic-pvc-pod2 created
$ kubectl get pod
#NAME              READY   STATUS    RESTARTS   AGE
#dynamic-pvc-pod   1/1     Running   0          2m
#dynamic-pvc-pod2   1/1     Running   0          6s
$ kubectl exec -it dynamic-pvc-pod2 -- sh
ls /data    
#hello.txt  test.txt
cat /data/test.txt 
#test  
cat /data/hello.txt
#hello from dynamic PV
#hello from second pod
exit
$ kubectl delete pod dynamic-pvc-pod dynamic-pvc-pod2
pod "dynamic-pvc-pod" deleted
pod "dynamic-pvc-pod2" deleted
kubectl delete pvc dynamic-pvc
#persistentvolumeclaim "dynamic-pvc" deleted

```
# Real Life Scenario
```bash
# Install required linux components to use NFS drive
apt -y install nfs-common

# Add nfs storage class provider: NFS Subdirectory External Provisioner Helm Repository
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

# Install NFS Provisioner
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=10.99.0.103 --set nfs.path=/nfs --set storageClass.defaultClass=true -n nfs --create-namespace

# Check whether NFS is mounted as FS:
df- h
xxx.xxx.gov.tr:/xfs/XFS 100T /DATA
```
```bash
$ kubectl apply -f pv-hostpath.yaml 
#persistentvolume/hostpath-pv created

$ kubectl get pv
#NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
#hostpath-pv   1Gi        RWO            Retain           Available                          <unset>                          9s

kubectl apply -f pvc-hostpath.yaml 
#persistentvolumeclaim/hostpath-pvc created

$ kubectl get pv,pvc
#NAME                           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
#persistentvolume/hostpath-pv   1Gi        RWO            Retain           Bound    default/hostpath-pvc                  <unset>                          100s

#NAME                                 STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
#persistentvolumeclaim/hostpath-pvc   Bound    hostpath-pv   1Gi        RWO                           <unset>                 22s
```
