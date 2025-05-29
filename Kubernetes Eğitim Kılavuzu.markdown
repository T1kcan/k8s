# Kubernetes Eğitim Kılavuzu

Bu doküman, Kubernetes'e giriş, temel bileşenler, ileri düzey konular, ağ, güvenlik, izleme, loglama ve CI/CD entegrasyonu hakkında kapsamlı bir genel bakış sunar.

## 1. Giriş ve Temel Kavramlar

### Kubernetes'e Giriş
Kubernetes (K8s), konteynerize uygulamaların dağıtımını, ölçeklendirilmesini ve yönetimini otomatikleştiren açık kaynaklı bir platformdur. Google tarafından geliştirilmiş ve Cloud Native Computing Foundation (CNCF) tarafından yönetilmektedir. Altyapıyı soyutlaştırarak geliştiricilerin uygulama mantığına odaklanmasını sağlar.

**Temel Özellikler:**
- **Konteyner Orkestrasyonu**: Birden fazla ana bilgisayarda konteyner yaşam döngüsünü yönetir.
- **Ölçeklenebilirlik**: Talebe göre uygulamaları otomatik olarak ölçeklendirir.
- **Kendi Kendini Onarma**: Başarısız konteynerleri yeniden başlatır ve sağlıklı düğümlere yeniden planlar.
- **Servis Keşfi ve Yük Dengeleme**: Trafiği sağlıklı pod'lara yönlendirir.

#### Neden Kubernetes Kullanmalıyız?
- Monolitik uygulamaların zorluklarını (ölçeklenme, dağıtım, bakım) ele alır.
- Mikroservis mimarilerinin orkestrasyon ihtiyacını karşılar.
- **Faydalar**:
  - Trafiğe göre otomatik ölçeklendirme.
  - Otomatik yeniden başlatmalarla yüksek erişilebilirlik.
  - Trafiğin eşit dağıtımı için servis keşfi ve yük dengeleme.
  - Çöken uygulamalar için kendi kendini onarma.
  - Kalıcı depolama yönetimi.
  - Kesintisiz güncellemeler ve geri almalar.
  - CPU ve bellek gibi kaynakların etkin yönetimi.

### Temel Kavramlar: Pod, Node, Cluster

#### Pod
- Kubernetes'teki en küçük çalıştırılabilir birim.
- Aynı ağ ve depolama alanını paylaşan bir veya daha fazla konteyner içerir.
- Geçici (ephemeral) nesnelerdir, genellikle Deployment gibi nesneler tarafından kontrol edilir.
- **Örnek**: Bir web uygulaması ve log toplama aracı aynı Pod içinde çalışabilir.
- **Pod Tanımı (pod.yaml)**:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
    - name: nginx-container
      image: nginx:latest
      ports:
      - containerPort: 80
  ```
- **Komutlar**:
  ```bash
  kubectl apply -f pod.yaml         # Pod oluştur
  kubectl get pods                 # Pod'ları listele
  kubectl describe pod nginx-pod   # Pod detaylarını görüntüle
  ```

#### Node
- Kubernetes kümesinde Pod'ları çalıştıran fiziksel veya sanal bir işçi makinesi.
- Bir ajan (kubelet) ve konteyner çalışma zamanı (ör. Docker) içerir.
- **Türler**:
  - **İşçi Node**: Uygulamaları çalıştırır.
  - **Ana Node**: Kümenin yönetiminden sorumludur.
- **Komut**:
  ```bash
  kubectl get nodes   # Node'ları listele
  ```

#### Cluster
- Ana (Master) ve İşçi (Worker) Node'ların birleşimi ile API server, scheduler ve controller manager gibi bileşenlerden oluşan yapı.
- **Komut**HAND:
  ```bash
  kubectl version --short # Versiyon bilgisi
  kubectl cluster-info   # Küme (Cluster) bilgisini görüntüle
  kubectl get nodes # Kümedeki nodeları listele
  kubectl get nodes -o wide # Kümedeki nodeları detaylı olarak listele
  kubectl describe node node-name # Node hakkında detaylı olarak bilgileri getir
  ```

### Kubernetes Kurulumu (Minikube, Kubectl)
**Windows Hyper-V Aktive Etme (PowerShell)**:
```
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
  #or
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  #or
  Turn Windows Feature on or Off / Click Hyper-V
```
#### Minikube
- Yerel makinede test amaçlı tek nod'lu bir Kubernetes kümesi çalıştırır.
- **Linux/Mac Kurulumu**:
  ```bash
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  sudo install minikube-linux-amd64 /usr/local/bin/minikube
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  sudo install kubectl /usr/local/bin/kubectl
  minikube start --driver=docker
  ```
- **Windows Kurulumu (PowerShell)**
  ```bash
  choco install minikube kubernetes-cli
  or
  Download minikube-installer https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download
  minikube start --driver=hyperv
  minikube config set driver virtualbox
  # Add to Path
  $oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}

  #Powershell ekranını kapatıp yönetici modunda tekrar aç.

- **Minikube Komutları**:
  ```bash
  minikube version         # Versiyon bilgisi
  minikube status          # Durumu kontrol et
  minikube stop            # Minikube'yi durdur
  minikube delete          # Kümeyi sil
  minikube ssh             # sanal makine içinde oturum aç
  minikube start --cpus 2 --memory 2400 # 2 cpu ve 4 gb ram ile yeni bir kubernetes cluster başlat
  minikube start p mycluster
  minikube dashboard       # Kontrol panelini aç
  minikube dashboard --url      # Kontrol panelini aç
  minikube node list       # nodeları listele
  minikube node add        # yeni node ekleme
  minikube node delete minikube-m02 # eklenen nodu silme
  minikube image list      # minikube tarafından kullanılan docker imajlarını listeleme
  ```

#### Kubectl
- Kubernetes kümelerini yönetmek için komut satırı aracı.
- **Örnek Komutlar**:
  ```bash
  kubectl get pods
  kubectl get pods -A                # Tüm Pod'ları listele
  kubectl get node                   # Tüm Node'ları listele
  kubectl get nodes -o wide          # Node'ları detaylı listele
  kubectl run --help
  kubectl run nginx --image=nginx --dry-run=client
  kubectl run web --image=nginx:1.10
  kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
  kubectl expose deployment hello-minikube --type=NodePort --port=8080
  minikube service hello-minikube   # Servise eriş
  kubectl get pod -o wide
  curl 10.244.0.6
  minikube ssh
  curl 10.244.0.6:8080
  
  ```
### Hello Minikube Web Sitesi

https://kubernetes.io/docs/tutorials/hello-minikube/

### İlk Uygulamanın Kubernetes Üzerinde Çalıştırılması
- Basit bir Nginx uygulaması dağıtma:
  ```bash
  kubectl create deployment nginx --image=nginx
  kubectl expose deployment nginx --type=NodePort --port=80
  minikube service nginx --url
  ```
- Doğrulama:
  ```bash
  kubectl get pods,services
  ```
## Docker Desktop

## Play With Kubernetes
https://training.play-with-kubernetes.com/
https://killercoda.com/playgrounds/scenario/kubernetes

## 2. Kubernetes Temelleri

### Deployment ve ReplicaSet
- **Deployment**: Durumsuz uygulamaları yönetir, belirli sayıda Pod kopyasının çalışmasını sağlar. Kesintisiz güncellemeler ve geri almaları destekler.
  ```bash
  kubectl create deployment my-app --image=my-app:1.0 --replicas=3
  ```
- **ReplicaSet**: Belirtilen sayıda Pod kopyasının her zaman çalışmasını sağlar. Genellikle Deployment'lar tarafından yönetilir.
  ```bash
  kubectl get replicasets
  ```

### Servisler (Services)
- Pod'lar için sabit ağ uç noktaları sağlar, yük dengeleme ve servis keşfi sunar.
- **Türler**:
  - **ClusterIP**: Varsayılan, yalnızca dahili IP.
  - **NodePort**: Servisi bir nod'un IP'si ve sabit bir portta açar.
  - **LoadBalancer**: Harici bir yük dengeleyici sağlar.
- **Örnek**:
  ```bash
  kubectl create service clusterip my-app --tcp=80:8080
  ```

### ConfigMap ve Secret
- **ConfigMap**: Hassas olmayan yapılandırma verilerini depolar.
  ```bash
  kubectl create configmap my-config --from-literal=key1=value1
  ```
- **Secret**: Hassas verileri (base64 kodlu) depolar.
  ```bash
  kubectl create secret generic my-secret --from-literal=password=secret123
  ```
- **Kullanım**: Pod'larda hacim olarak bağlama veya ortam değişkeni olarak enjekte etme.

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