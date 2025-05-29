# Managing Secrets and ConfigMaps

## Outline

- Part 1 - Setting up the Kubernetes Cluster

- Part 2 - Kubernetes Secrets

- Part 3 - ConfigMaps in Kubernetes

## Part 1 - Setting up the Kubernetes Cluster

- Check if Kubernetes is running and nodes are ready.

```bash
kubectl cluster-info
kubectl get no
```

## Part 2 - Kubernetes Secrets

## Creating your own Secrets 

### Creating a Secret Using kubectl

- Secrets can contain user credentials required by Pods to access a database. For example, a database connection string consists of a username and password. You can store the username in a file ./username.txt and the password in a file ./password.txt on your local machine.

```bash
# Create files needed for the rest of the example.
echo -n 'admin' > ./username.txt
echo -n '1f2d1e2e67df' > ./password.txt
echo -n 'Password123' > ./password.txt
```

- The kubectl create secret command packages these files into a Secret and creates the object on the API server. The name of a Secret object must be a valid DNS subdomain name. Show types of secrets with opening : (Kubetnetes Secret Types)[https://kubernetes.io/docs/concepts/configuration/secret/]

```bash
$ kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
```

- The output is similar to:

```bash
secret "db-user-pass" created
```

- Default key name is the filename. You may optionally set the key name using `[--from-file=[key=]source]`.

```bash
$ kubectl create secret generic db-user-pass-key --from-file=username=./username.txt --from-file=password=./password.txt
```

>Note:
>Special characters such as `$`, `\`, `*`, `=`, and `!` will be interpreted by your shell and require escaping. In most shells, the easiest way to escape the password is to surround it with single quotes (`'`). For example, if your actual password is S!B\*d$zDsb=, you should execute the command this way:
>
>```bash
>kubectl create secret generic dev-db-secret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb='
>```
>You do not need to escape special characters in passwords from files (--from-file).

- You can check that the secret was created:

```bash
$ kubectl get secrets
```

- The output is similar to:

```bash
NAME                  TYPE                                  DATA      AGE
db-user-pass          Opaque                                2         51s
```

You can view a description of the secret:

```bash
$ kubectl describe secrets/db-user-pass
```

Note: The commands kubectl get and kubectl describe avoid showing the contents of a secret by default. This is to protect the secret from being exposed accidentally to an onlooker, or from being stored in a terminal log.

The output is similar to:
```bash
Name:            db-user-pass
Namespace:       default
Labels:          <none>
Annotations:     <none>

Type:            Opaque

Data
====
password.txt:    12 bytes
username.txt:    5 bytes
```

### Creating a Secret manually 

- You can also create a Secret in a file first, in JSON or YAML format, and then create that object. The name of a Secret object must be a valid DNS subdomain name. The Secret contains two maps: data and stringData. The data field is used to store arbitrary data, encoded using base64. The stringData field is provided for convenience, and allows you to provide secret data as unencoded strings.

- For example, to store two strings in a Secret using the data field, convert the strings to base64 as follows:

```bash
$ echo -n 'admin' | base64
```

- The output is similar to:

```bash
YWRtaW4=
```

```bash
$ echo -n '1f2d1e2e67df' | base64
```

- The output is similar to:

```bash
MWYyZDFlMmU2N2Rm
```

- Write a Secret that looks like this named secret.yaml:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

- Now create the Secret using `kubectl apply`:

```bash
$ kubectl apply -f ./secret.yaml
```

- The output is similar to:

```bash
secret "mysecret" created
```

### Decoding a Secret

- Secrets can be retrieved by running kubectl get secret. For example, you can view the Secret created in the previous section by running the following command:

```bash
kubectl get secret mysecret -o yaml
```

- The output is similar to:

```yaml
apiVersion: v1
data:
  password: MWYyZDFlMmU2N2Rm
  username: YWRtaW4=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"password":"MWYyZDFlMmU2N2Rm","username":"YWRtaW4="},"kind":"Secret","metadata":{"annotations":{},"name":"mysecret","namespace":"default"},"type":"Opaque"}
  creationTimestamp: "2021-10-06T12:51:08Z"
  name: mysecret
  namespace: default
  resourceVersion: "38986"
  uid: fb55a84e-24f5-461d-a000-e7dab7c34200
type: Opaque
```

- Decode the password field:

```bash
$ echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
```

- The output is similar to:

```bash
1f2d1e2e67df
```

### Using Secrets 

- This is an example of a Pod that uses secrets from environment variables:

- mysecret-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```

- Create the pod.

```bash
$ kubectl apply -f mysecret-pod.yaml
```

### Consuming Secret Values from environment variables

- Inside a container that consumes a secret in an environment variables, the secret keys appear as normal environment variables containing the base64 decoded values of the secret data. This is the result of commands executed inside the container from the example above:

- Enter into pod and type following command.

```bash
kubectl exec -it secret-env-pod -- bash
root@secret-env-pod:/data# echo $SECRET_USERNAME
admin
root@secret-env-pod:/data# echo $SECRET_PASSWORD
1f2d1e2e67df
```

## Part 3 - ConfigMaps in Kubernetes

- A ConfigMap is a dictionary of configuration settings. This dictionary consists of key-value pairs of strings. Kubernetes provides these values to your containers. Like with other dictionaries (maps, hashes, ...) the key lets you get and set the configuration value.

- A ConfigMap stores configuration settings for your code. Store connection strings, public credentials, hostnames, environment variables, container command line arguments and URLs in your ConfigMap.

- ConfigMaps bind configuration files, command-line arguments, environment variables, port numbers, and other configuration artifacts to your Pods' containers and system components at runtime.

- ConfigMaps allow you to separate your configurations from your Pods and components. 

- ConfigMap helps to makes configurations easier to change and manage, and prevents hardcoding configuration data to Pod specifications.

- ConfigMaps are useful for storing and sharing non-sensitive, unencrypted configuration information.

- For the show case we will select a simple application that displays a message like this.

```text
Hello!
```

- We will parametrized the "Hello" portion in some languages.

```bash
$ mkdir k8s
$ cd k8s/
```

- Create 3 files.

- configmap1.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-message
data:
  MESSAGE: "Merhaba Kubernetes Dünyası! 🎉"
```
- deployment1.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: env-web-app
  template:
    metadata:
      labels:
        app: env-web-app
    spec:
      containers:
        - name: app
          image: tbincan/webappgo
          ports:
            - containerPort: 8080
          env:
            - name: MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: env-message
                  key: MESSAGE
```

- service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: env-web-service
spec:
  selector:
    app: env-web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: NodePort

```

- See the files and go upper folder.

```bash
$ ls
configmap1.yaml  deployment1.yaml  service1.yaml
$ cd .. 
```

- Now apply `kubectl` to these files.

```bash
$ kubectl apply -f k8s
configmap/env-message unchanged
deployment.apps/env-web-app created
service/env-web-service created
```

Let's see the message.

```bash
$ kubectl get service/env-web-service -o wide
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE   SELECTOR
env-web-service   NodePort   10.99.109.188   <none>        80:31241/TCP   60s   app=env-web-app
#if you are running on linux machine:
$ curl < worker-ip >:31241
# if you are running on Minikube:
kubectl run testclient --rm --image=busybox --restart=Never -it -- sh
# wget -qO- 10.99.109.188
```html
                <!DOCTYPE html>
                <html lang="tr">
                <head>
                        <meta charset="UTF-8">
                        <title>Environment Variable Viewer</title>
                </head>
                <body>
                        <h1>Merhaba Kubernetes Dünyası! 🎉</h1>
                </body>
                </html>
```
```
This is the default container behaviour.

Now delete what we have created.

```bash
$ kubectl delete -f k8s
configmap "env-message" deleted
deployment.apps "env-web-app" deleted
service "env-web-service" deleted
```

## Create and use ConfigMaps with `kubectl create configmap` command

There are three ways to create ConfigMaps using the `kubectl create configmap` command. Here are the options.

1. Use the contents of an entire directory with `kubectl create configmap my-config --from-file=./my/dir/path/`
   
2. Use the contents of a file or specific set of files with `kubectl create configmap my-config --from-file=./my/file.txt`
   
3. Use literal key-value pairs defined on the command line with `kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2`

### Literal key-value pairs

We will start with the third option. We have just one parameter. Greet with "Halo" in Spanish.

```bash
$ kubectl create configmap env-message --from-literal=MESSAGE=Hola
configmap/env-message created
```

- Explain the important parts in `ConfigMap` file contents.

```yaml
$ kubectl get configmap/env-message -o yaml
apiVersion: v1
data:
  MESSAGE: Hola
kind: ConfigMap
metadata:
  creationTimestamp: "2025-05-25T18:03:32Z"
  name: env-message
  namespace: default
  resourceVersion: "715"
  uid: 010c2499-c78c-4458-aef8-8f635946baf5
```

- Apply `kubectl` to these files.

```bash
$ kubectl apply -f deployment.yaml  
deployment.apps/demo created
$ kubectl apply -f  service.yaml
service/demo-service created
```

- List the services.

```bash
$ kubectl get service/env-web-service -o wide
NAME              TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTOR
env-web-service   NodePort   10.105.137.232   <none>        80:31223/TCP   2m25s   app=env-web-app
```

- See the message.

```bash

#if you are running on linux machine:
$ curl < worker-ip >:31223
Halo!
# if you are running on Minikube:
$ kubectl run testclient --rm --image=busybox --restart=Never -it -- wget -qO- 10.105.137.232
#minikube service list
#minikube service env-web-service


                <!DOCTYPE html>
                <html lang="tr">
                <head>
                        <meta charset="UTF-8">
                        <title>Environment Variable Viewer</title>
                </head>
                <body>
                        <h1>Hola</h1>
                </body>
                </html>
        pod "testclient" deleted
```

- Reset what we have created.

```bash
$ kubectl get cm
NAME          DATA   AGE
env-message        1      116s

$ kubectl delete cm env-message
configmap "env-message" deleted

$ kubectl delete -f service.yaml
service "env-web-service" deleted

$ kubectl delete -f deployment.yaml
deployment.apps "env-web-app" deleted
```

### From a config file

- We will write the greeting key-value pair in a file in Norvegian and create the ConfigMap from this file.

```bash
echo "MESSAGE: Hei" > config
```

Note that, the comman notation used in key-value pairs is to use `key= value` notation, but this is not an obligatory. The notation actualy depends on the applicaton implementation that will parse and use these files.

- Look at the other example files that look like below

```bash
$ ls 
game.properties
ui.properties

$ cat game.properties
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30

$ cat ui.properties
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice
```

- Let's create our configmap from `config` file.

```bash
$ kubectl create configmap env-message --from-file=./config
configmap/env-message created
```

- Check the content of the `configmap/env-message`.

```json
$ kubectl get  configmap/env-message -o json
{
    "apiVersion": "v1",
    "data": {
        "config": "MESSAGE: Hei\n"
    },
    "kind": "ConfigMap",
    "metadata": {
        "creationTimestamp": "2025-05-25T18:08:38Z",
        "name": "env-message",
        "namespace": "default",
        "resourceVersion": "1011",
        "uid": "b2901874-561f-4169-9535-2d3e3d836e2c"
    }
}
```

We have modifed our application to read parameters from the file. So create another `deployment` file as follows:

# deployment2.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: env-web-app
  template:
    metadata:
      labels:
        app: env-web-app
    spec:
      containers:
        - name: app
          image: tbincan/webappgofile
          ports:
            - containerPort: 8080
          volumeMounts:
          - mountPath: /config/
            name: config-volume
            readOnly: true
      volumes:
      - name: config-volume
        configMap:
          name: env-message
          items:
          - key: config
            path: config.yaml
```

- Volume and volume mounting are common ways to place config files inside a container. We are selecting `config` key from `env-message` ConfigMap and put it inside the container at path `/config/` with the name `config.yaml`.

- Apply and run all the configurations as follow:

```bash
$ kubectl apply -f deployment2.yaml 
deployment.apps/env-web-app created

$ kubectl get pod 
NAME                          READY   STATUS    RESTARTS   AGE
env-web-app-865479867-2k9vh   1/1     Running   0          7s

$ kubectl apply -f service1.yaml 
service/env-web-service created

$ kubectl get svc
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
env-web-service   NodePort    10.102.142.247   <none>        80:32600/TCP   5s
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        52m

#if you are running on linux machine:
$ curl < worker-ip >:32600
Hei
# if you are running on Minikube:
$ kubectl run testclient --rm --image=busybox --restart=Never -it -- wget -qO- 10.102.142.247
#minikube service list
#minikube service env-web-service
```

- Reset what we have created.

```bash
$ kubectl get cm
NAME               DATA   AGE
env-message        1      52m
$ kubectl delete cm env-message
configmap "env-message" deleted
$ kubectl delete svc env-web-service
service "env-web-service" deleted
$ kubectl delete -f deployment2.yaml 
deployment.apps "env-web-app" deleted
```
