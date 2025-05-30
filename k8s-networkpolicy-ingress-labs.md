# NetworkPolicy & Ingress

---

## NetworkPolicy - Restrict Pod Communication

### Goal:
Only Pods with label `role=frontend` should access Pods with label `role=db`.

---

### Step 1: Create Namespace and Pods

```bash
kubectl create ns netpol-lab

kubectl run frontend \
  --image=nginx \
  -n netpol-lab \
  --labels="role=frontend" \
  --restart=Never

kubectl run db \
  --image=nginx \
  -n netpol-lab \
  --labels="role=db" \
  --restart=Never

kubectl run intruder \
  --image=nginx \
  -n netpol-lab \
  --labels="role=intruder" \
  --restart=Never
```
List nodes:
```bash
kubectl get pod -o wide -n netpol-lab
NAME       READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
db         1/1     Running   0          2m2s    192.168.1.5   node01   <none>           <none>
frontend   1/1     Running   0          2m12s   192.168.1.4   node01   <none>           <none>
intruder   1/1     Running   0          107s    192.168.1.6   node01   <none>           <none>
```
---

### Step 2: Test connectivity before applying policy

```bash
kubectl exec -it frontend -n netpol-lab -- curl 192.168.1.5
kubectl exec -it intruder -n netpol-lab -- curl 192.168.1.5
```

✅ Both should succeed since no network policy is enforced.

---

### Step 3: Apply NetworkPolicy

Save as `networkpolicy-restrict-pod.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-only
  namespace: netpol-lab
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
```

Apply the policy:

```bash

kubectl apply -f networkpolicy-restrict-pod.yaml
```
### Step 4: Test again

```bash
kubectl exec -it frontend -n netpol-lab -- curl 192.168.1.5   # ✅ Should succeed
kubectl exec -it intruder -n netpol-lab -- curl 192.168.1.5   # ❌ Should fail

```

---

## NetworkPolicy - Restrict Pod Communication

### Goal:
Allow Access Only from Same Namespace.

---
Save as `networkpolicy-ns.yaml`:

```yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
  namespace: netpol-lab
spec:
  podSelector:
    matchLabels:
      role: db
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: netpol-lab
    - podSelector:
        matchLabels:
          role: intruder
  policyTypes:
    - Ingress
```

Apply the policy:

```bash
kubectl run intruder2 \
  --image=nginx \
  -n netpol-lab \
  --labels="role=intruder" \
  --restart=Never
kubectl apply -f networkpolicy-ns.yaml
```
---

### Step 4: Test again

```bash
kubectl exec -it frontend -n netpol-lab -- curl 192.168.1.5   # ✅ Should succeed
kubectl exec -it intruder -n netpol-lab -- curl 192.168.1.5   # ✅ Should succeed
kubectl exec -it intruder2 -n netpol-lab -- curl 192.168.1.5   # ✅ Should succeed
kubectl delete -f networkpolicy-ns.yaml
kubectl exec -it intruder2 -n netpol-lab -- curl 192.168.1.5   # ❌ Should fail
```
# service/networking/network-policy-allow-all-ingress.yaml
If you want to allow all incoming connections to all pods in a namespace, you can create a policy that explicitly allows that.

---
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
  namespace: netpol-lab
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress
```
```bash
kubectl exec -it intruder -n netpol-lab -- curl 192.168.1.5 # ✅ Should succeed
```
---

## Ingress Controller (NGINX) - Path Routing

### Goal:
Expose a web app through HTTP on a custom path using NGINX Ingress.

---

### 🧰 Prerequisites

Ensure NGINX Ingress Controller is installed. If not, install it (example using Helm):

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# helm install nginx-ingress ingress-nginx/ingress-nginx \
#   --namespace ingress-nginx \
#   --create-namespace

  helm install nginx ingress-nginx/ingress-nginx \
    --namespace ingress-nginx --create-namespace \
    --set controller.service.type=NodePort

```
You should be seeing an output similar to below:
```text
helm install nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.service.type=NodePort

NAME: nginx
LAST DEPLOYED: Thu May 29 19:54:01 2025
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
Get the application URL by running these commands:
  export HTTP_NODE_PORT=$(kubectl get service --namespace ingress-nginx nginx-ingress-nginx-controller --output jsonpath="{.spec.ports[0].nodePort}")
  export HTTPS_NODE_PORT=$(kubectl get service --namespace ingress-nginx nginx-ingress-nginx-controller --output jsonpath="{.spec.ports[1].nodePort}")
  export NODE_IP="$(kubectl get nodes --output jsonpath="{.items[0].status.addresses[1].address}")"

  echo "Visit http://${NODE_IP}:${HTTP_NODE_PORT} to access your application via HTTP."
  echo "Visit https://${NODE_IP}:${HTTPS_NODE_PORT} to access your application via HTTPS."

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

Wait for the controller to be ready:

```bash
kubectl get pods -n ingress-nginx
NAME                                              READY   STATUS    RESTARTS   AGE
nginx-ingress-nginx-controller-678bcf8fc8-fkqzz   1/1     Running   0          79s
```

---

### Step 1: Deploy and expose app

```bash
kubectl create ns ingress-lab

kubectl create deployment my-app --image=nginx -n ingress-lab

kubectl expose deployment my-app \
  --port=80 \
  --target-port=80 \
  --name=my-app-svc \
  -n ingress-lab
```

---

### Step 2: Create Ingress resource

Save as `my-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  namespace: ingress-lab
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: example.com
      http:
        paths:
          - path: /app
            pathType: Prefix
            backend:
              service:
                name: my-app-svc
                port:
                  number: 80
```

Apply the resource:

```bash
kubectl apply -f my-ingress.yaml
```

---

### Step 3: Test the Ingress

1. Get the **external IP or LoadBalancer address**:

```bash
kubectl get svc -n ingress-nginx
```

Note your nodes IPs and `ingress-nginx-controller` nodePort ex:
nginx-ingress-nginx-controller             NodePort    10.106.110.5    <none>        80:30653/TCP,443:32572/TCP   2m39s

2. Edit your `/etc/hosts` file (Linux/macOS):

```
<EXTERNAL-IP> example.com
```

Or use curl directly:

```bash
curl -H "Host: example.com" http://<EXTERNAL-IP>/app
```

✅ You should see the NGINX welcome page.


## Path-based Routing

### Step 1: Create Web ve API Application and Services

#### Web Deployment and Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

#### API Deployment ve Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: hashicorp/http-echo
          args:
            - "-text=Merhaba API'den"
          ports:
            - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 5678
```

### Step 2: Ingress Resource Definition

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-routing-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /web
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 8080
```

### ✅ Step 3: Access Test

1. run 
```bash
curl kubectl get ingress
```
and add `myapp.local` DNS name to `/etc/hosts` file:
    ```
10.109.141.197 myapp.local
    ```
2. Use browser or `curl` to access:
    ```bash
    curl http://myapp.local/web
    curl http://myapp.local/api
    ```

---


## Example 2: Subdomain-based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: subdomain-routing
  namespace: default
spec:
  ingressClassName: nginx
  rules:
    - host: app1.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 80
    - host: app2.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app2-service
                port:
                  number: 80
```

---

## Example 3: Default Backend (Catch-All)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-backend-example
  namespace: default
spec:
  ingressClassName: nginx
  defaultBackend:
    service:
      name: default-backend
      port:
        number: 80
```

---

## Example 4: Enforcing HTTPS (Redirect HTTP to HTTPS)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: force-https
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - secure.example.com
      secretName: tls-secret
  rules:
    - host: secure.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: secure-service
                port:
                  number: 443
```

---

## Example 5: Rate Limiting (NGINX Only)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rate-limit-example
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/limit-connections: "1"
    nginx.ingress.kubernetes.io/limit-rpm: "10"
spec:
  ingressClassName: nginx
  rules:
    - host: limited.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: slow-service
                port:
                  number: 80
```
