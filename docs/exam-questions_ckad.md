# Exam Questions on Kubernetes CKAD Exam

## Questions

### Question 1

**Task:**

1. Update the Propertunel scaling configuration of the Deployment web1 in the ckad00015 namespace setting maxSurge to 2 and maxUnavailable to 59
2. Update the web1 Deployment to use version tag 1.13.7 for the Ifconf/nginx container image.
3. Perform a rollback of the web1 Deployment to its previous version

**Solution:**

```bash
k edit deploy web1 -n ckad00015
k set image deploy web1 Ifconf/nginx=Ifconf/nginx:1.13.7 -n ckad00015
k rollout undo deploy web1 -n ckad00015
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web1
  namespace: ckad00015
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web1
  template:
    metadata:
      labels:
        app: web1
    spec:
      containers:
      - name: nginx
        image: Ifconf/nginx:1.13.7
        ports:
        - containerPort: 80
  strategy:
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 59
    type: RollingUpdate
```

```bash
k create secret generic app-secret -n default --from-literal=key3=value1
k run nginx-secret -n default --image=nginx:stable --dry-run=client -o yaml > nginx-secret.yaml
vi nginx-secret.yaml
k create -f nginx-secret.yaml
k config use-context k8s
k edit deploy web1 -n ckad00015
k rollout status deploy web1 -n ckad00015
k rollout undo deploy web1 -n ckad00015
k rollout history deploy web1 -n ckad00015
k get rs -n ckad00015
```

### Question 2

Context
You sometimes need to observe a pod's logs, and write those logs to a file for further analysis. Task
Please complete the following;

- Deploy the counter pod to the cluster using the provided YAMLspec file at `/opt/KDOB00201/counter.yaml`
- Retrieve all currently available application logs from the running pod and store them in the file `/opt/KDOB0020l/log_Output.txt`, which has already been created

**Solution:**

```bash
k create -f /opt/KDOB00201/counter.yaml
k logs counter > /opt/KDOB00201/log_Output.txt
```

### Question 3

Context

Anytime a team needs to run a container on Kubernetes they will need to define a pod within which to run the container.

**Task:**

Please complete the following:

- Create a YAML formatted pod manifest `/opt/KDPD00101/podl.yml` to create a pod named app1 that runs a container named app1cont using image `Ifccncf/arg-output` with these command line arguments: `-lines 56 -F`
- Create the pod with the kubectl command using the YAML file created in the previous step
- When the pod is running display summary data about the pod in JSON format using the kubect1 command and redirect the output to a file named `/opt/KDPD00101/out1.json`
- All of the files you need to work with have been created, empty, for your convenience

**Solution:**

```bash
k run app1 --image=Ifccncf/arg-output --dry-run=client -o yaml --command -- /bin/sh -c '-lines 56 -F' > /opt/KDPD00101/pod1.yml
sed -i 's/command:/args:/' /opt/KDPD00101/pod1.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
  - name: app1cont
    image: Ifccncf/arg-output
    args: ["-lines", "56", "-F"]
```

```bash
k create -f /opt/KDPD00101/pod1.yml
k get pod app1 -o json > /opt/KDPD00101/out1.json
```

### Question 4

Context

You are tasked to create a ConfigMap and consume the ConfigMap in a pod using a volume mount. Task
Please complete the following:

- Create a ConfigMap named `another-config` containing the key/value pair: `key4/value3`
- start a pod named nginx-configmap containing a single container using the nginx image, and mount the key you just created into the pod under directory `/also/a/path`

**Solution:**

```bash
k create configmap another-config --from-literal=key4=value3
k run nginx-configmap --image=nginx --dry-run=client -o yaml > nginx-configmap.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /also/a/path
  volumes:
  - name: config-volume
    configMap:
      name: another-config
```

```bash
k create -f nginx-configmap.yaml
```

### Question 5

Context

You have been tasked with scaling an existing deployment for availability, and creating a service to expose the deployment within your infrastructure.

**Task:**

Start with the deployment named `kdsn00101-deployment` which has already been deployed to the namespace `kdsn00101`.
Edit it to:

- Add the `func=webFrontEnd` key/value label to the pod template metadata to identify the pod for the service definition
- Have 4 replicas

Next, create ana deploy in namespace kdsn00l01 a service that accomplishes the following:

- Exposes the service on `TCP` port `8080`
- is mapped to me pods defined by the specification of `kdsn00l01-deployment`
- Is of type `NodePort`
- Has a name of cherry

**Solution:**

```bash
k edit deploy kdsn00101-deployment -n kdsn00101
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kdsn00101-deployment
  namespace: kdsn00101
spec:
  replicas: 4
  selector:
    matchLabels:
      app: kdsn00101
  template:
    metadata:
      labels:
        app: kdsn00101
        func: webFrontEnd
    spec:
      containers:
      - name: kdsn00101
        image: Ifccncf/arg-output
        ports:
        - containerPort: 80
```

```bash
k expose deploy kdsn00101-deployment --port=8080 --target-port=80 --type=NodePort --name=cherry -n kdsn00101
```

### Question 6

**Task:**

Update the Deployment `app-1` in the frontend namespace to use the existing ServiceAccount app.

**Solution:**

```bash
k edit deploy app-1 -n frontend
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-1
  namespace: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-1
  template:
    metadata:
      labels:
        app: app-1
    spec:
      serviceAccountName: app
      containers:
      - name: app-1
        image: Ifccncf/arg-output
        ports:
        - containerPort: 80
```

```bash
k set serviceaccount deploy app-1 app -n frontend
```

### Question 7

**Task:**

- Create a Deployment named expose in the existing `ckad00014` namespace running 6 replicas of a Pod.
- Specify a single container using the `ifccncf/nginx: 1.13.7` image
- Add an environment variable named `NGINX_PORT` with the value `8001` to the container then expose port `8001`

**Solution:**

```bash
k create deploy expose --image=ifccncf/nginx:1.13.7 --replicas=6 -n ckad00014 --dry-run=client -o yaml > expose.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: expose
  namespace: ckad00014
spec:
  replicas: 6
  selector:
    matchLabels:
      app: expose
  template:
    metadata:
      labels:
        app: expose
    spec:
      containers:
      - name: expose
        image: ifccncf/nginx:1.13.7
        env:
        - name: NGINX_PORT
          value: "8001"
        ports:
        - containerPort: 8001
```

```bash
k create -f expose.yaml
```

### Question 8

Context
You are asked to prepare a Canary deployment for testing a new application release.

**Task:**

A Service named krill-Service in the goshark namespace points to 5 pod created by the Deployment named current-krill-deployment

> [!INFO] The Service is exposed on NodePort 30000, to test it's load balancing run `curl http:k8s.master-0:30000`

**Solution:**

```bash
k scale deploy canary-krill-deployment --replicas=4 -n goshark
```

Text Description automatically generated

```bash
wget https://k8s.io/examples/admin/resource/quota-pod.yaml
vim quota-pod.yaml
```

```bash
k create -f quota-pod.yaml

curl http:k8s.master-0:30000
```

### Question 9

**Task:**

The pod for the Deployment named nosql in the craytisn namespace fails to start because its container runs out of resources.
Update the nosol Deployment so that the Pod:

- The nosol Deployment can be found in `~/chief-ardianl/nosql.yaml`

**Solution:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nosql
  namespace: crayfish
  labels:
    app.kubernetes.io/name: nosql
    app.kubernetes.io/component: backend
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: nosql
      app.kubernetes.io/component: backend
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nosql
        app.kubernetes.io/component: backend
    spec:
      containers:
      - name: mongo
        image: mongo:4.2
        args:
          - --bind_ip
          - 0.0.0.0
        ports:
          - containerPort: 27017
        resources:
          requests:
            memory: "160Mi"
          limits:
            memory: "320Mi"
```

```bash
k describe netpol

k label pod cakd00018-newpod -n ckad00018 web-access=true
k label pod cakd00018-newpod -n ckad00018 db-access=true

vi ~/chief-ardianl/nosql.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nosql
  namespace: crayfish
  labels:
    app.kubernetes.io/name: nosql
    app.kubernetes.io/component: backend
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: nosql
      app.kubernetes.io/component: backend
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nosql
        app.kubernetes.io/component: backend
    spec:
      containers:
      - name: mongo
        image: mongo:4.2
        args:
          - --bind_ip
          - 0.0.0.0
        ports:
          - containerPort: 27017
        resources:
          requests:
            memory: "160Mi"
          limits:
            memory: "320Mi"
```

```bash
k apply -f ~/chief-ardianl/nosql.yaml
```

```bash
k get deploy -n crayfish
```

```bash
k get pod -n crayfish
```

```bash
k describe pod nosql-7b4b7b7b4b-7b4b7b7b4b -n crayfish
```

### Question 10

Context

A user has reported an aopticauon is unteachable due to a failing livenessProbe.

**Task:**

Perform the following tasks:

- Find the broken pod and store its name and namespace to /opt/KDOB00401/broken.txt in the format: `<podname>/<namespace>`

The output file has already been created

- Store the associated error events to a file /opt/KDOB00401/error.txt, The output file has already been created. You will need to use the -o wide output specifier with your command
- Fix the issue.

> [!INFO] The associated deployment could be running in any of the following namespaces:
>
> - qa
> - test
> - production
> - staging

Solution:
Create the Pod:

```bash
kubectl create -f http://k8s.io/docs/tasks/configure-pod-container/exec-liveness.yaml
```

Within 30 seconds, view the Pod events: kubectl describe pod liveness-exec
The output indicates that no liveness probes have failed yet:

```text
FirstSeen LastSeen Count From SubobjectPath Type Reason Message
--------- -------- ----- ---- ------------- -------- ------ ------
24s 24s 1 {default-scheduler } Normal Scheduled Successfully assigned liveness-exec to worker0
23s 23s 1 {kubelet worker0} spec.containers{liveness} Normal Pulling pulling image "gcr.io/google_containers/busybox"
23s 23s 1 {kubelet worker0} spec.containers{liveness} Normal Pulled Successfully pulled image "gcr.io/google_containers/busybox"
23s 23s 1 {kubelet worker0} spec.containers{liveness} Normal Created Created container with docker id 86849c15382e; Security:[seccomp=unconfined]
23s 23s 1 {kubelet worker0} spec.containers{liveness} Normal Started Started container with docker id 86849c15382e
```

After 35 seconds, view the Pod events again: kubectl describe pod liveness-exec
At the bottom of the output, there are messages indicating that the liveness probes have failed, and the containers have been killed and recreated.

```text
FirstSeen LastSeen Count From SubobjectPath Type Reason Message
--------- -------- ----- ---- ------------- -------- ------ ------
37s 37s 1 {default-scheduler } Normal Scheduled Successfully assigned liveness-exec to worker0
36s 36s 1 {kubelet worker0} spec.containers{liveness} Normal Pulling pulling image "gcr.io/google_containers/busybox"
36s 36s 1 {kubelet worker0} spec.containers{liveness} Normal Pulled Successfully pulled image "gcr.io/google_containers/busybox"
36s 36s 1 {kubelet worker0} spec.containers{liveness} Normal Created Created container with docker id 86849c15382e; Security:[seccomp=unconfined]
36s 36s 1 {kubelet worker0} spec.containers{liveness} Normal Started Started container with docker id 86849c15382e
2s 2s 1 {kubelet worker0} spec.containers{liveness} Warning Unhealthy Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

Wait another 30 seconds, and verify that the Container has been restarted: kubectl get pod liveness-exec
The output shows that RESTARTS has been incremented:

```text
NAME READY STATUS RESTARTS AGE
liveness-exec 1/1 Running 1 m
```
