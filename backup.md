Node label attract the pods with a matching label.
Node taint repel any pods (unless they have the corresponding toleration)
```bash
kubectl run myapp --image=nginx --labels="env=fronted"
kubectl run myapp --image=nginx --labels="env=prod,tier=frontend"
# add a label running pod
kubectl label pods myapp "env=demo, tier=backend"
# label a node
kubectl label node "disktype=ssd"
kubectl taint nodes vasaspoccas workload.sas.com/class=cas:NoSchedule --overwrite
kubectl label nodes vasaspoccas workload.sas.com/class=cas --overwrite
# display labels
kubectl get pods --show-labels
kubectl get pods --selector="env=production, tier=frontend"
kubectl get pods --selector="tier!=production"
kubectl get pods -l "env in (production)"
kubectl get pods -l "tier notin (frontend)"
#test
kubectl run k8s-web-2 --image=nginx
kubectl run k8s-web-3 --image=nginx
kubectl get pod
kubectl describe pod k8s-web-2
#observe Labels: run=k8s-web-2
kubectl get pod --show-labels
kubectl run k8s-web-4 --image=nginx --labels="env=prod"
kubectl get pod --show-labels
```
- Following commands should be executed on Master Node only.

- Pull the packages for Kubernetes beforehand

```bash
sudo kubeadm config images pull
```