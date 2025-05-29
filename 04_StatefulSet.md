# StatefulSets


StorageClass and PersistentVolume definitions for StatefulSet
This file defines a StorageClass and two PersistentVolumes for use with StatefulSets in Kubernetes.
The StorageClass is set to use the 'kubernetes.io/no-provisioner' provisioner, which means it does not support dynamic provisioning.
The PersistentVolumes are configured to use host paths, which is suitable for single-node clusters or testing purposes. 
The volumes are set to be bound to the StorageClass 'local-storage' and are configured with a capacity of 1Gi each.
The access mode is set to 'ReadWriteOnce', allowing the volume to be mounted as read-write by a single node.
```yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
- Persistent Volumes for StatefulSet
```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-0
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data/pv-0"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data/pv-1"

```
- Following file is Headless Service and StatefulSet from nginx for test purpose
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.24
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

```bash
# Test the StatefulSet
kubectl get pod -l app=nginx
kubectl exec -it web-0 -- sh
echo "Hello from web-0" > /usr/share/nginx/html/index.html
kubectl exec -it web-0 -- sh
echo "Hello from web-1" > /usr/share/nginx/html/index.html 
kubectl exec -it web-1 -- sh
curl web-0.nginx
Hello from web-0
kubectl run testclient --image=busybox --restart=Never -it -- sh
wget -qO- web-0.nginx
wget -qO- web-1.nginx
```

- Another StatefulSet from busybox image
```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-0
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data/pv-0"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data/pv-1"
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  serviceName: "myapp"
  replicas: 2
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: busybox
          image: busybox
          command: ["sh", "-c", "echo Hello from $(hostname) > /data/hello.txt && sleep 3600"]
          volumeMounts:
            - name: data
              mountPath: /data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: local-storage
        resources:
          requests:
            storage: 1Gi
---
```
# PostgreSQL StatefulSet
PersistentVolume configuration for PostgreSQL StatefulSet
This configuration includes a StorageClass, a Headless Service, a StatefulSet for PostgreSQL, 
Note: Ensure that the PersistentVolumes (pv-0, pv-1) are created in advance or adjust the storageClassName to match your environment.
- Persistent Volumes for StatefulSet
```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-0
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data/pv-0"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data/pv-1"

```
- Storage Class
```yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

- PostgreSQL Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None  # Headless service!
  selector:
    app: postgres
  ports:
    - port: 5432
      name: postgres
```
- PostgreSQL StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15
          ports:
            - containerPort: 5432
              name: postgres
          env:
            - name: POSTGRES_DB
              value: mydb
            - name: POSTGRES_USER
              value: myuser
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: pg-secret
                  key: password
          volumeMounts:
            - name: pgdata
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: pgdata
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: local-storage
        resources:
          requests:
            storage: 1Gi
```

- PostgreSQL Secret for storing sensitive data
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pg-secret
type: Opaque
data:
  password: cG9zdGdyZXNxbHBhc3M=  # base64(postgresqlpass)
```

```bash
# Note: The password is base64 encoded. You can generate it using:
echo -n "postgresqlpass" | base64

# To apply this configuration, save it to a file named `statefulset.yaml` and run:
kubectl apply -f statefulset.yaml

# To check the status of the StatefulSet and Pods, you can use:
kubectl get statefulsets
kubectl get pods

# To access the PostgreSQL database, you can use a PostgreSQL client:
kubectl run -it --rm --image=postgres:15 --restart=Never psql-client -- psql -h postgres -U myuser -d mydb
# paste pasword when prompted (the password is "postgresqlpass").
\l  # List databases
\c mydb  # Connect to the database
\dt  # List tables in the current database

# To connect to the PostgreSQL database, you can use the following command:
kubectl exec -it <pod-name> -- psql -U myuser -d mydb

# To delete the StatefulSet and its associated resources, you can run:
kubectl delete statefulset postgres
kubectl delete service postgres 
kubectl delete secret pg-secret
kubectl delete pvc -l app=postgres
# To clean up the PersistentVolumes, you may need to delete them manually if they are not automatically deleted:
kubectl delete pvc pgdata-postgres-0 pgdata-postgres-1
kubectl delete pv pv-0 pv-1
kubectl delete storageclass local-storage
kubectl delete all --all
