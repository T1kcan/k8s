# Health Check Types
Readiness Probe: do not send traffic until the application is ready.
Liveness Probe: Pod is running however is the application healty?
Startup Probe: Legay applications, slow starting apps.
200-400 http responses are ok.
Otherwise pod is restarted.
Probe Types:
ExecAction: $?=0 ok
TCPSocketAction: Is the TCP Port accessable?
HTTPGetAction: http response from address 200-400
intialDelaySeconds: Default 0
periodSeconds: Probe frequency default 10
timeoutSeconds: Default 1
successThreshold: Minimum success count to evaluate container is healty
failureThreshold: Failure count to restart

## HTTPGetAction
### liveness1.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness1
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10

```
httpGet: Nginx Pod / endpoint HTTP GET 
initialDelaySeconds: 5: First problem is done 5 seconds after Container starts.
periodSeconds: 10: Probe is made each 10 seconds.
```bash
kubectl apply -f liveness1.yaml
# Stop Nginx
kubectl exec -it liveness1 -- sh
rm -rf /usr/share/nginx/html/index.html
exit
# observe pod restarts
kubectl describe pod liveness1
kubectl exec -it livenessexec2 -- sh;
rm -rf /usr/share/nginx/html/index.html
```
## ExecAction 
### liveness2.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness2
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; while true; do echo OK; sleep 5; done
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
---
apiVersion: v1
kind: Pod
metadata:
  name: livenessexec2
spec:
  containers:
  - name: liveness-command-exec
    image: nginx
    ports:
        - containerPort: 80
    livenessProbe:
      exec:
        command:
        - cat
        - /usr/share/nginx/html/index.html
      initialDelaySeconds: 2
      periodSeconds: 2
```

```bash
kubectl exec -it liveness2 -- sh
ls /tmp
#healthy
rm -rf /tmp/healthy
kubectl describe pod liveness2
kubectl exec -it livenessexec2 -- sh
rm -rf /usr/share/nginx/html/index.html
kubectl describe pod livenessexec2
```

# Tcp Socket
### liveness3.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness3
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 2
      periodSeconds: 1
    volumeMounts:
    - name: nginx-config-volume
      mountPath: /etc/nginx/conf.d/default.conf
      subPath: default.conf
  volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
---
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcpsocket
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```



```bash
kubectl apply -f liveness3.yaml
kubectl cp liveness3:/etc/nginx/conf.d/default.conf nginx.tar
# extract default.conf from tar ball
# create configmap from default.conf
kubectl create configmap nginx-config --from-file=default.conf
# change port 80-> 8080 in configmap
kubectl edit cm nginx-config
# when configmap changes pod must be restarted
kubectl delete pod liveness3
kubectl apply -f liveness3.yaml

