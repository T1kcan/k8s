## Label-Selector

```bash
# Clusterdaki nodeları Label’ları ile birlikte listeler
kubectl get node --show-labels
# Default namespacedeki podları Label’ları ile birlikte listeler
kubectl get po --show-labels
# Nginx image’ını kullanan my-web isimli podu label ataması yaparak oluştur/başlat
kubectl run my-web --image=nginx --labels="env=prod,tier=frontend"
# Çalışan my-web isimli pod üzerine “tier=backend” label etiketi ekle
kubectl label pods my-web tier=backend
# Çalışan my-web isimli pod üzerinde tanımlı “tier” etiketini değiştirme
kubectl label pods my-web tier=frontend --overwrite
# Default namespace üzerindeki tüm podlar üzerine “status=healthy” etiketini ekle
kubectl label pods --all status=healthy
# emea.internal isimli node üzerine “disktype=ssd” etiketini ekle
kubectl label nodes emea.internal disktype=ssd
# Etiketi “env=prod” olan podları listele
kubectl get po --selector="env=prod"
# Etiketi “env=prod” olmayan podları listele
kubectl get po --selector="env!=prod"
# Etiketi “env=prod,tier=backend” olan podları listele
kubectl get po -l "env=prod,tier=backend"
# Etiketi “env=prod” olan podları listele
kubectl get po -l "env in (prod)"
# Etiketi “env=prod,tier=backend” olan podları listele
kubectl get po -l "env in (prod),tier in (backend)"
# Etiketi “env=prod,env=demo” olan podları listele
kubectl get po -l "env in (prod,demo)"
# Etiketi “env=demo” olan podları sil
kubectl delete pods -l "env=demo"
# Etiketi "env=prod,tier=backend"olan podları sil
kubectl delete pods -l "env in (prod),tier in (backend)"
```
# Equality-based requirement
```
environment = production
tier != frontend
```
# Set-based requirement
```
environment in (production, qa)
tier notin (frontend, backend)
```
```bash
# Pod'a Label atama
kubectl label pod tomcat-labelpod-2 app.kubernetes.io/version="9.0" 
# Node'a Label atama
kubectl label nodes node1 disktype=ssd
# Bir namespace’deki tüm objelere toplu halde label atama
kubectl label pods --all foo=bar
# Node üzerinde ki Label bilgisini kaldırma
kubectl label node minikube kubernetes.io/say-
# Pod üzerindeki label bilgisini kaldırma
kubectl label pod tomcat-labelpod-2 app.kubernetes.io/version-
# Label olan Pod üzerinde değişiklik yapma
kubectl label pod tomcat-labelpod-2 app.kubernetes.io/version="10.0" --overwrite
# Podlara atanan labelları listeleme
kubectl get po --show-labels
# Pod üzerinde bulunan labelları görüntüleme
kubectl describe pod tomcat-labelpod-1
# Selector kullanarak Label üzerinden POD listeleme 
kubectl get po -l app.kubernetes.io/version="8.0" -o wide
# Selector kullanarak Label üzerinden POD Silme
kubectl delete po -l app.kubernetes.io/name=tomcat
# Selector kullanarak Label üzerinden POD Listeleme
kubectl get pods -l "app=prod,tier=frontend" --show-labels
# Selector kullanarak Label üzerinden POD Listeleme
kubectl get pods -l 'app in (prodapp)' --show-labels
# Selector kullanarak Label üzerinden POD Listeleme
kubectl get pods -l 'app in (prodapp,demoapp)' --show-labels
# Selector kullanarak Label üzerinden POD Listeleme
kubectl get pods -l 'app in (prodapp),tier notin (frontend)' --show-labels
# Selector kullanarak Label üzerinden POD Listeleme
kubectl get pods -l 'app notin (prodapp)' --show-labels
```
# label-selector.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    env: production
    tier: frontend
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    env: production
    tier: frontend
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod3
  labels:
    env: production
    tier: frontend
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod4
  labels:
    env: production
    tier: backend
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod5
  labels:
    env: production
    tier: backend
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod6
  labels:
    app: firstapp
    tier: frontend
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod7
  labels:
    app: firstapp
    tier: frontend
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod8
  labels:
    app: secondapp
    tier: backend
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod9
spec:
  containers:
  - name: app-container
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod10
spec:
  containers:
  - name: app-container
    image: nginx
  nodeSelector:
    hddtype: ssd
```
```bash
kubectl apply -f label-selector.yaml
kubectl apply -f tomcat.yaml
kubectl get pod
kubectl get pod -w
kubectl get pod --show-labels
kubectl get pods --selector="env=production" --show-labels
kubectl get pods --selector="tier" --show-labels
kubectl get pod -l "tier in (frontend)" --show-labels
kubectl get pod -l "tier notin (frontend)" --show-labels
kubectl get pod -l "tier notin (fistapp,secondapp)" --show-labels
kubectl get pod -l "!env" --show-labels
kubectl get pod --show-labels
kubectl label pods pod9 env=production
kubectl label pods --all "ver=1"
kubectl get pod --show-labels
kubectl get nodes --show-labels
#pod10 has nodeSelector, thence needs a node with labeled hddtype:ssd
kubectl label nodes minikube hddtype=ssd
#:->=
```
# Annotations

```bash
kubectl annotate --help
kubectl annotate pod tomcat-1 Description='Version:1'
kubectl describe pod tomcat-1
kubectl annotate pod tomcat-1 Description='Version:2'
kubectl annotate pod tomcat-1 Description='Version:2' --overwrite
kubectl describe pod tomcat-1
```

# Pod Environment Variables
# mysql.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "Password123"   
```
```bash
kubectl apply -f mysql.yaml
kubectl exec -it mysql-b7dd75968-xlxd8 -- env
kubectl describe pod mysql-b7dd75968-xlxd8
```
# Node Selector
```bash
minikube delete
minikube start --nodes 3
kubectl describe nodes
```
nodeselector.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  nodeSelector:
    #kubernetes.io/hostname: your-node-name
    disktype: ssd
  containers:
    - name: busybox
      image: busybox:1.36
      command: ['sh', '-c', 'while true; do echo Hello from BusyBox; sleep 3600; done']
      imagePullPolicy: IfNotPresent
  restartPolicy: Always
```
```bash
kubectl describe pod busybox-pod
Warning  FailedScheduling  14s   default-scheduler  0/1 nodes are available: 1 node(s) didn't match Pod's node affinity/selector. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.'
```
```bash
kubectl label node minikube-m03 disktype=ssd
kubectl describe node minikube-m03
```
# Node Selector
# affinity1.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - your-node-name
  containers:
    - name: busybox
      image: busybox:1.36
      command: ['sh', '-c', 'while true; do echo Hello from BusyBox with nodeAffinity; sleep 3600; done']
      imagePullPolicy: IfNotPresent
  restartPolicy: Never
```
operator: In, NotIn, Exists, DoesNotExist

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-preferred
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 20
          preference:
            matchExpressions:
            #   - key: disktype
            #     operator: In
            #     values:
            #       - ssd
            - key: kubernetes.io/hostname
              operator: In
              values:
              - minikube
        - weight: 10
          preference:
            matchExpressions:
            #   - key: disktype
            #     operator: In
            #     values:
            #       - ssd
            - key: kubernetes.io/hostname
              operator: In
              values:
              - minikube-m03
  containers:
    - name: busybox
      image: busybox:1.36
      command: ['sh', '-c', 'while true; do echo Hello from preferred nodeAffinity; sleep 3600; done']
      imagePullPolicy: IfNotPresent
  restartPolicy: Always

```
# Taints and Tolerations



#### Effect types
```
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:PreferNoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
```
***
#### Node Taint & Remove Taint
```
kubectl taint nodes minikube-m03 disktype=ssd:NoSchedule
kubectl taint nodes minikube-m03 disktype-
```
***
#### POD tolerations
```
tolerations:
- key: "hardware"
  operator: "Equal"
  value: "special"
  effect: "NoSchedule"
```
***
#### POD tolerations - "memorysize" is enough for taint for the node
```
tolerations:
- key: "memorysize"
  operator: "Exists"
  effect: "NoSchedule"
```


```bash
kubectl describe node | grep Taints
kubectl taint node minikube dedicated=web:NoSchedule
kubectl taint node minikube dedicated-
kubectl taint node minikube dedicated=web:NoSchedule
kubectl describe node | grep Taints -C3
#Taints:             dedicated=web:NoSchedule
kubectl run taintpod --image=nginx --restart=Never
kubectl get pod -w
# NAME       READY   STATUS    RESTARTS   AGE
# taintpod   0/1     Pending   0          0s
kubectl describe pod taintpod
#   Warning  FailedScheduling  68s   default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {dedicated: web}. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.

kubectl delete pod --all

```
# tainted.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: taintedpod1
  labels:
    name: taintedpod
spec:
  containers:
  - image: nginx
    name: taintpod
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "web"
    effect: "NoSchedule"
```

# tainted2.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: taintedpod2
  labels:
    name: taintedpod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
  tolerations:
  - key: "dedicated"
    operator: "Exists"
    effect: "NoSchedule"
```
```bash
kubectl describe node | grep Taints
kubectl taint node minikube dedicated=web:NoSchedule
kubectl apply -f tainted.yaml 
kubectl apply -f tainted2.yaml 
```
# Cordon Node
Disable scheduling pods to nodes
```bash
kubectl cordon --help
kubectl cordon minikube --dry-run=client
node/minikube cordoned (dry run)
kubectl cordon minikube
#node/minikube cordoned
kubectl get nodes
#NAME       STATUS                     ROLES           AGE     VERSION
#minikube   Ready,SchedulingDisabled   control-plane   4h46m   v1.32.0
kubectl get pods
#NAME          READY   STATUS    RESTARTS   AGE
#taintedpod1   1/1     Running   0          8m31s
#taintedpod2   1/1     Running   0          8m26s
kubectl delete pod taintedpod1
kubectl apply -f tainted.yaml 
kubectl get pod
#NAME          READY   STATUS    RESTARTS   AGE
#taintedpod2   1/1     Running   0          8m40s
#taintedpod1   0/1     Pending   0          0s
kubectl uncordon minikube
#node/minikube uncordoned
kubectl get pod
#NAME          READY   STATUS    RESTARTS   AGE
#taintedpod1   1/1     Running   0          113s
#taintedpod2   1/1     Running   0          10m
kubectl get nodes
#NAME       STATUS   ROLES           AGE     VERSION
#minikube   Ready    control-plane   4h50m   v1.32.0

```

# Kubectl Drain
Turns node into maintenance mode, kernel upgrade, hardware upgrade or reboot purposes. Removes node from cluster. Evicts pods from node to other nodes. To make it schedulable again ` uncordon` node.
```bash
kubectl drain node --force # including ReplicationController, Jobs and DaemonSets
kubectl drain node --force --ignore-daemonsets # skips daemonsets
```