
### Kubernetes Setup (Minikube, Kubectl)
- Enabling hypervisor on Windows 

**Windows Hyper-V Activation (PowerShell run as admistrator**:
```
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
  #or
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  #or
  Turn Windows Feature on or Off / Click Hyper-V
  #Disable Hyper-V use: 'bcdedit /set hypervisorlaunchtype off'
```
- Reboot Windows
 
#### Minikube Installation
- Local Kubernetes test environment.
- **Linux/Mac Setup**:
  ```bash
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  sudo install minikube-linux-amd64 /usr/local/bin/minikube
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  sudo install kubectl /usr/local/bin/kubectl
  minikube start --driver=docker
  ```
- **Windows Setup (PowerShell)**
  ```bash
  choco install minikube kubernetes-cli
  or
  Download minikube-installer https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download
  minikube start --driver=hyperv
  minikube config set driver virtualbox #set default driver according to your virtualization environment
  # Add to Path
  $oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}

  #Run Powershell again in administrative mode.

- **Minikube Commands**:
  ```bash
  minikube version         # Version info
  minikube status          # Minikube cluster status
  minikube stop            # Stop Minikube
  minikube delete          # Delete Minikube Cluster
  minikube ssh             # Acces via ssh to Minikube Virtual Machine
  minikube start --cpus 2 --memory 2400 # Start Minikube Cluster with 2 cpu ve 4 gb ram 
  minikube start p mycluster # Start Minikube Cluster with the name of cluster
  minikube dashboard       # Opens up dashboard on default browser
  minikube dashboard --url      # Prints url for dashboard
  minikube node list       # Lists Minikube nodes
  minikube node add        # Add new node to cluster
  minikube node delete minikube-m02 # Delete node from cluster
  minikube image list      # List docker images used by Minikube
  ```

#### Kubectl Setup
- Kubernetes CLI Interface
https://kubernetes.io/releases/download/
https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/


- **Sample kubectl Commands**:
  ```bash
  kubectl get pods                   # Lists pods in current namespace
  kubectl get pods -A                # Lists pods in all namespaces
  kubectl get node                   # Lists nodes on the cluster
  kubectl get nodes -o wide          # Detailed list of nodes
  kubectl run --help                 
  kubectl run nginx --image=nginx --dry-run=client #Act as if running pod
  kubectl run web --image=nginx:1.10    # Runs nginx pod
  kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0  #Create deployment
  kubectl expose deployment hello-minikube --type=NodePort --port=8080 #Create service for deployment
  minikube service hello-minikube   # Access service outside of Minikube
  kubectl get pod -o wide   
  curl 10.244.0.6
  minikube ssh
  curl 10.244.0.6:8080
  
  ```
### Hello Minikube Tutorial

https://kubernetes.io/docs/tutorials/hello-minikube/

### First Application on Kubernetes
- Run simple nginx application:
  ```bash
  kubectl create deployment nginx --image=nginx
  kubectl expose deployment nginx --type=NodePort --port=80
  minikube service nginx --url
  ```
- List pod and services:
  ```bash
  kubectl get pods,services
  ```
## Docker Desktop

- Minikube can be created on Docker Desktop on Windows 

## Play With Kubernetes
https://training.play-with-kubernetes.com/
https://killercoda.com/playgrounds/scenario/kubernetes
