# Kubernetes Training Session

# Kubernetes Usage
    kubectl     command     type    object-name     flag​

                get         pods    xxxx            –o wide​
                describe    deployment​
                delete      node​
                logs        service
    kubectl     run         myweb –image=nginx:latest​
                get​
                run​
                describe​
                delete​
                exec​
                logs​
                apply​
                explain

po pods​
namespace -> ns​
replicaset -> rs​
pod/myweb​

# Sample Kubernetes YAML File
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-first
  labels:
    app: frontend
spec:
  containers:
  - name: web
    image: nginx:latest
    ports:
    - containerPort: 80
```
# Imperative Way Of Using Kubernetes
```bash
kubectl run NAME --image=image [--env="key=value"] [--port=port] [--dry-run=server|client] [--overrides=inline-json] [--command] -- [COMMAND] [args...]
kubectl run my-web --image=nginx
kubectl run apache --image=httpd --port=8080
kubectl run nginxpod --image=nginx --port=80 --restart=Never
kubectl run my-web --image=nginx --dry-run=client
kubectl run my-web --image=nginx --dry-run=client -o yaml
kubectl run my-web --image=nginx --dry-run=client -o yaml >> mypod.yaml
kubectl run my-web --image=nginx --restart=Never
kubectl delete pods my-web
kubectl delete pod,service baz foo
kubectl delete -f pod.yaml
kubectl delete pods --all
kubectl delete all --all
kubectl delete -n my-ns pod,svc --all
kubectl delete pod foo –now
kubectl delete pod foo --force
kubectl edit po my-web
kubectl edit deployment my-deployment
```
# Declerative Way Of Using Kubernetes
```bash
kubectl apply -f ./sample.yaml
kubectl apply -f ./directory_name
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml       
kubectl create -f ./sample.yaml
kubectl run testpod --image=httpd --dry-run=client -o yaml > testpod.yaml
kubectl run pod1 --image=alpine:3.9 --restart=Never --dry-run=client --command -o yaml -- sleep 3600 > pod1.yaml
kubectl run pod2 --image=busybox --restart=Never --dry-run=client -o yaml --command -- printenv HOSTNAME KUBERNETES_PORT > pod2.yaml
kubectl edit 
kubectl delete -f ./sample.yaml 

```
```bash
alias k="kubectl"​
kubectl cluter-info​
kubectl get –help​
kubectl get all​
kubectl get po –o name
kubectl describe pod​
kubectl describe node minikube​
kubectl explain pod​
kubectl explain pod.metadata ​
kubectl explain pod.metadata --recursive​
kubectl run k8s-pod-1 –image=hello-world​
kubectl run k8s-pod-2 –image=hello-world –restart=Never​
minikube image list​
kubectl delete pod k8s-pod-1​
kubectl delete pod –all​
kubectl delete all --all
```         
# Jsonpath
--output=jsonpath​
kubectl get pod deneme –o=jsonpath='{.metadata.name}'​
--output=jsonpath​
kubectl get pod deneme –o=jsonpath='{.metadata.namespace}'​
--output=jsonpath​
kubectl get pod deneme –o=jsonpath='{.spec.containers[*].name}'​
--output=jsonpath​
kubectl get pod deneme –o=jsonpath='{.spec.containers.[*].volumeMounts[*].name}'​
--output=jsonpath​
kubectl get pod deneme –o=jsonpath='{.status.posIP}'​
--output=jsonpath
```bash
kubectl get pods -o=jsonpath="{.items[*]['metadata.name']}"​
kubectl get pods –output=json | grep "podIP"​
kubectl get pod –sort-by=.metadata.name​
```
--dry-run=client –o yaml:​
```bash
kubectl run test –image=nginx –dry-run=client –o yaml​
kubectl run test –image=nginx –dry-run=client –o yaml >>nginx.yaml
# kubectl get pods --field-selector metadata.name=myApp
# kubectl get pods --field-selector metadata.namespace=production
# kubectl get pods --field-selector metadata.namespace!=Project
# kubectl get services  --all-namespaces --field-selector metadata.namespace!=default
# kubectl get pods --field-selector status.phase=Running
# kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always
```
# Pod Interaction
```bash
kubectl exec mypod – bash​
kubectl exec –it mypod – bash​
kubectl exec –it mypod –c containername – bash​
kubectl exec –it mypod – date​
minikube image load pyton​
minikube list​
kubectl run k8s-python-1 –it –rm --image=pyton​
kubectl run k8s-mariadb-1 –image=mariadb –restart=Never –env=MYSQL_ROOT_PASSWORD='Password123'​
kubectl exec –it k8s-mariadb-1 – mariadb –uroot –p​
```
```yaml
Create database test;​
Show databases;
```
```bash
kubectl cp –help​
kubectl cp file podname:/folder​
kubectl cp namespace/podname:/folder/file .​
minikube cp​
minikube ssh​
sudo passwd docker​
minikube ip​
```
Winscp​
Winscp use docker user

# Kubernetes Port-Forwarding
  ```bash
kubectl port-forward k8s-web 8080:80​
kubectl port-forward pods/mariadb 3306:3306​
kubectl port-forward deployment/mongo 28015:27017​
kubectl port-forward replicaset/mongo-7868483f :27017​
kubectl cp . K8s-web-1:/tmp/web​
kubectl exec –it k8s-web-1 – bash​
cd /tmp/web​
cp –r * /usr/share/nginx/html​
kubectl port-forward –help​
kubectl port-forward k8s-web-1 8080:80​
kubectl run k8s-mariadb-2 –image=mariadb –restart=Never –env=MYSQL_ROOT_PASSWORD="Password123"
kubectl port-forward k8s-mariadb-2 3306:3306​
#Use db application to connect database
  ```
# Label and Selector
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
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-crashloopback
spec:
  restartPolicy: Always
  containers:
  - name: c1
    image: busybox:1.28
    env:
    - name: MESSAGE
      value: "hello world"
    command: ["/bin/echo"]
    args: ["$(MESSAGE)"]

```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      nodeSelector:
        kubernetes.io/hostname: viya-monitor
      tolerations:
      - key: "monitor"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
      containers:
      - name: my-container
        image: nginx
```

### Uygulama Konfigürasyonlarının Yönetimi
- Tekrarlanabilirlik için YAML dosyalarında yapılandırmalar tanımlayın.
- **Örnek ConfigMap**:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    db_host: "mysql-service"
    log_level: "debug"
  ```
- Uygulama ve referans:
  ```bash
  kubectl apply -f configmap.yaml
  ```
  ```yaml
  envFrom:
  - configMapRef:
      name: app-config
  ```

## 3. İleri Düzey Konular

### Volume ve Persistent Volume
- **Volume**: Pod'lara bağlı geçici veya kalıcı depolama sağlar.
  - Türler: `emptyDir`, `hostPath`, `nfs`, vb.
- **Persistent Volume (PV)**: Küme genelinde bağımsız depolama kaynağı.
- **Persistent Volume Claim (PVC)**: PV'den depolama talep eder.
- **Örnek**: 10Gi NFS PV'den 5Gi talep etme.
  ```bash
  kubectl create -f pvc.yaml
  ```

### StatefulSets
- Sabit ağ kimlikleri ve kalıcı depolama gerektiren durumlu uygulamaları yönetir.
- **Özellikler**: Sıralı Pod oluşturma, sabit ana bilgisayar adları (ör. `app-0`, `app-1`).
- **Örnek**: MySQL StatefulSet:
  ```yaml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: mysql
  spec:
    serviceName: mysql
    replicas: 3
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
          env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: password
          volumeMounts:
          - name: data
            mountPath: /var/lib/mysql
    volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
  ```
  ```bash
  kubectl apply -f mysql-statefulset.yaml
  ```

### Dağıtık Sistemlerin Kubernetes ile Yönetimi
- Dağıtık veritabanları (ör. MongoDB, Cassandra) için StatefulSet kullanın.
- Güvenilirlik için sağlık kontrolleri ve hazır olma sondaları uygulayın.

## 4. Ağ ve Güvenlik

### Kubernetes Ağ Yapısı
- **Pod Ağı**: Pod'lar, bir CNI eklentisi (ör. Calico, Flannel) ile benzersiz IP'ler alır.
- **Servis Ağı**: Servisler, Pod IP'lerini soyutlayan sanal IP'ler sağlar.
- **Ağ Politikaları**: Etiketlerle trafik kontrolü.

### Güvenlik Uygulamaları
- **RBAC**: Kullanıcı ve servisler için izinleri tanımlar.
  ```bash
  kubectl create role pod-reader --verb=get,list --resource=pods
  ```
- **Pod Güvenlik Standartları**: K8s 1.21+ için Pod Security Policies yerine kullanılır.

### Ingress Kontrol
- HTTP/HTTPS servislerini URL tabanlı yönlendirme ile dışarıya açar.
- **Örnek**:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: my-ingress
  spec:
    rules:
    - host: example.com
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: my-app
              port:
                number: 80
  ```
  ```bash
  kubectl apply -f ingress.yaml
  ```

### Güvenlik ve Ağ Yönetimi
- Trafik kontrolü için Ağ Politikaları kullanın.
- Ingress için TLS etkinleştirin.
- Secret'ları düzenli olarak yenileyin ve şifreleme için Sealed Secrets gibi araçlar kullanın.

## 5. İzleme ve Loglama

### Kubernetes İzleme (Prometheus, Grafana)
- **Prometheus**: Metrik toplar.
  ```bash
  helm install prometheus prometheus-community/kube-prometheus-stack
  ```
- **Grafana**: Metrikleri görselleştirir.
  ```bash
  kubectl port-forward svc/prometheus-grafana 3000:80
  ```

### Loglama (Elasticsearch, Fluentd, Kibana)
- **EFK Yığını**:
  - Fluentd: Logları toplar.
  - Elasticsearch: Logları depolar.
  - Kibana: Logları görselleştirir.
  ```bash
  helm install efk elastic/eck-operator
  ```

### İzleme ve Loglama Yapılandırması
- Özel metrikler için Prometheus kuralları tanımlayın:
  ```yaml
  apiVersion: monitoring.coreos.com/v1
  kind: PrometheusRule
  metadata:
    name: example-alert
  spec:
    groups:
    - name: example
      rules:
      - alert: HighPodCPU
        expr: rate(container_cpu_usage_seconds_total[5m]) > 0.8
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Yüksek CPU kullanımı tespit edildi"
  ```

### Uyarı ve Bildirim
- Bildirimler için Alertmanager yapılandırın (ör. Slack).
  ```yaml
  global:
    slack_api_url: '<your-slack-webhook>'
  route:
    receiver: 'slack-notifications'
  receivers:
  - name: 'slack-notifications'
    slack_configs:
    - channel: '#alerts'
      text: 'Uyarı: {{ .CommonAnnotations.summary }}'
  ```

## 6. CI/CD ve Kubernetes

### CI/CD Kavramları
- **Sürekli Entegrasyon (CI)**: Kod oluşturma ve test etme süreçlerini otomatikleştirir.
- **Sürekli Dağıtım (CD)**: Üretim veya hazırlık ortamlarına dağıtımı otomatikleştirir.

### Jenkins ile CI/CD Entegrasyonu
- Jenkins'i dağıt:
  ```bash
  helm install jenkins jenkins/jenkins
  ```
- **Örnek Jenkinsfile**:
  ```groovy
  pipeline {
    agent any
    stages {
      stage('Oluştur') {
        steps {
          sh 'docker build -t my-app:latest .'
        }
      }
      stage('Dağıt') {
        steps {
          sh 'kubectl apply -f deployment.yaml'
        }
      }
    }
  }
  ```

### GitOps (ArgoCD)
- Altyapı ve uygulama yapılandırmaları için Git'i tek doğruluk kaynağı olarak kullanır.
- **ArgoCD Kurulumu**:
  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```
- Uygulama oluştur:
  ```bash
  argocd app create my-app --repo <repo-url> --path k8s --dest-namespace default --dest-server https://kubernetes.default.svc
  ```

### CI/CD Pipeline Oluşturma ve Yönetimi
- Konteyner görüntülerini bir kayıt defterine oluştur ve gönder.
- Git deposunda Kubernetes manifestlerini güncelle.
- ArgoCD ile değişiklikleri kümeye senkronize et.
- Pipeline durumunu izle ve gerekirse geri al.