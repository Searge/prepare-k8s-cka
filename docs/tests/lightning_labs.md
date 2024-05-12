# Lightning labs

## Task #1: Create PV and PVC

Create a Persistent Volume called log-volume. It should make use of a storage class name manual. It should use RWX as the access mode and have a size of 1Gi. The volume should use the hostPath /opt/volume/nginx

Next, create a PVC called log-claim requesting a minimum of 200Mi of storage. This PVC should bind to log-volume.

Mount this in a pod called logger at the location /var/www/nginx. This pod should use the image nginx:alpine.

```yaml:pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  storageClassName: "manual"
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /opt/volume/nginx
```

```yaml:pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
  storageClassName: manual
```

Create a pod called logger with the following specifications:

- `k run logger --image=nginx:alpine --dry-run=client -o yaml > pod.yaml`

```yaml:pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: logger
  name: logger
spec:
  volumes:
    - name: log
      persistentVolumeClaim:
        claimName: log-claim
  containers:
  - image: nginx:alpine
    name: logger
    resources: {}
    volumeMounts:
      - name: log
        mountPath: /var/www/nginx
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Task #2: Troubleshoot the network

We have deployed a new pod called secure-pod and a service called secure-service. Incoming or Outgoing connections to this pod are not working.
Troubleshoot why this is happening.

Make sure that incoming connection from the pod webapp-color are successful.

Important: Don't delete any current objects deployed.

```bash
controlplane ~ ➜  k get po
NAME           READY   STATUS    RESTARTS   AGE
logger         1/1     Running   0          66s
secure-pod     1/1     Running   0          9s
webapp-color   1/1     Running   0          31m

controlplane ~ ➜  k get svc
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP   111m
secure-service   ClusterIP   10.108.77.186   <none>        80/TCP    95s

controlplane ~ ➜  k exec -it webapp-color -- sh
/opt # nc -v -z -w 2 secure-service 80
nc: secure-service (10.108.77.186:80): Operation timed out

controlplane ~ ✖ k get netpol
NAME           POD-SELECTOR   AGE
default-deny   <none>         4m56s

controlplane ~ ➜  k describe netpol
Name:         default-deny
Namespace:    default
Created on:   2024-05-11 08:16:07 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    <none> (Selected pods are isolated for ingress connectivity)
  Not affecting egress traffic
  Policy Types: Ingress

controlplane ~ ➜  k get po --show-labels
NAME           READY   STATUS    RESTARTS   AGE     LABELS
logger         1/1     Running   0          7m10s   run=logger
secure-pod     1/1     Running   0          6m13s   run=secure-pod
webapp-color   1/1     Running   0          37m     name=webapp-color

k get netpol default-deny -o yaml > netpol-allow.yaml
```

Create a Network Policy to allow incoming connections to the secure-pod from the webapp-color pod.

```yaml:netpol-allow.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  creationTimestamp: "2024-05-11T08:16:07Z"
  generation: 1
  name: network-policy
  namespace: default
  resourceVersion: "9017"
  uid: 9e15dee1-4454-4d95-b25c-304d7f6481ed
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            name: webapp-color
      ports:
        - protocol: TCP
          port: 80
```

```bash
controlplane ~ ➜  k exec -it webapp-color -- sh
/opt # nc -v -z -w 2 secure-service 80
secure-service (10.108.77.186:80) open
```

## Task #3: Create a pod with a config map

Create a pod called `time-check` in the `dvl1987` namespace. This pod should run a container called `time-check` that uses the busybox image.

1. Create a config map called time-config with the data `TIME_FREQ=10` in the same namespace.
2. The `time-check` container should run the command: `while true; do date; sleep $TIME_FREQ;done` and write the result to the location `/opt/time/time-check.log`.
3. The path `/opt/time` on the pod should mount a volume that lasts the lifetime of this pod.

Pod `time-check` configured correctly?

```bash
k create ns dvl1987
k create cm time-config --from-literal=TIME_FREQ=10 -n dvl1987

k run time-check --image=busybox -n dvl1987 --dry-run=client -o yaml --command -- \
   /bin/sh -c 'while true; do date; sleep $TIME_FREQ;done' > time-check.yaml
```

```yaml:time-check.yaml
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: dvl1987
spec:
  containers:
  - name: time-check
    image: busybox
    env:
    - name: TIME_FREQ
      valueFrom:
        configMapKeyRef:
          name: time-config
          key: TIME_FREQ
    command:
    - /bin/sh
    - -c
    - while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log
    volumeMounts:
    - name: log-volume
      mountPath: /opt/time
  volumes:
  - name: log-volume
    configMap:
      name: time-config
```

### Task #4: Create a deployment

Create a new deployment called `nginx-deploy`, with one single container called `nginx`, image `nginx:1.16` and `4` replicas.
The deployment should use `RollingUpdate` strategy with `maxSurge=1`, and `maxUnavailable=2`.

Next upgrade the deployment to version `1.17`.

Finally, once all pods are updated, undo the update and go back to the previous version.

```bash
k create deploy nginx-deploy --image=nginx:1.16 --replicas=4 --dry-run=client -o yaml > deploy.yaml
```

```yaml:deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
```

```bash
k apply -f deploy.yaml
k set image deploy nginx-deploy nginx=nginx:1.17
k rollout undo deploy nginx-deploy
```

### Task #5: Create a redis deployment

Create a `redis` deployment with the following parameters:

Name of the deployment should be `redis` using the `redis:alpine` image. It should have exactly `1` replica.

The container should request for `.2` CPU. It should use the label `app=redis`.

It should mount exactly 2 volumes.

a. An Empty directory volume called data at path `/redis-master-data`.
b. A configmap volume called `redis-config` at path`/redis-master`.
c. The container should expose the port `6379`.

The configmap has already been created.

```bash
k create deploy redis --image=redis:alpine --replicas=1 --dry-run=client -o yaml > redis-deploy.yaml
```

```yaml:redis-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:alpine
        resources:
          requests:
            cpu: ".2"
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: redis-config
```

```bash
k apply -f redis-deploy.yaml
```

### Task #6: Create a pod with a security context

We have deployed a few pods in this cluster in various namespaces. Inspect them and identify the pod which is not in a Ready state. Troubleshoot and fix the issue.

Next, add a check to restart the container on the same pod if the command `ls /var/www/html/file_check` fails. This check should start after a delay of `10` seconds and run every `60` seconds.

You may delete and recreate the object. Ignore the warnings from the probe.

```bash
k get po -A
k describe po nginx1401 -n dev1401
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  8m13s                 default-scheduler  Successfully assigned dev1401/nginx1401 to node01
  Normal   Pulling    8m10s                 kubelet            Pulling image "kodekloud/nginx"
  Normal   Pulled     8m5s                  kubelet            Successfully pulled image "kodekloud/nginx" in 288ms (5.362s including waiting)
  Normal   Created    8m5s                  kubelet            Created container nginx
  Normal   Started    8m5s                  kubelet            Started container nginx
  Warning  Unhealthy  3m3s (x36 over 8m4s)  kubelet            Readiness probe failed: Get "http://10.244.192.1:8080/": dial tcp 10.244.192.1:8080: connect: connection refused

k get po nginx1401 -n dev1401 -o yaml > nginx1401.yaml
```

```yaml:nginx1401.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"run":"nginx"},"name":"nginx1401","namespace":"dev1401"},"spec":{"containers":[{"image":"kodekloud/nginx","imagePullPolicy":"IfNotPresent","name":"nginx","ports":[{"containerPort":9080}],"readinessProbe":{"httpGet":{"path":"/","port":8080}},"resources":{}}],"dnsPolicy":"ClusterFirst","restartPolicy":"OnFailure"},"status":{}}
  creationTimestamp: "2024-05-11T18:08:34Z"
  labels:
    run: nginx
  name: nginx1401
  namespace: dev1401
  resourceVersion: "2853"
  uid: 23171315-ce42-47d6-846e-b60a1d397c26
spec:
  containers:
  - image: kodekloud/nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 9080
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        path: /
        port: 9080
        scheme: HTTP
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    livelinessProbe:
      exec:
        command:
        - ls
        - /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-tzrrd
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: node01
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: OnFailure
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-tzrrd
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-05-11T18:08:43Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2024-05-11T18:08:34Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-05-11T18:08:34Z"
    message: 'containers with unready status: [nginx]'
    reason: ContainersNotReady
    status: "False"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-05-11T18:08:34Z"
    message: 'containers with unready status: [nginx]'
    reason: ContainersNotReady
    status: "False"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-05-11T18:08:34Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://459203a3ee6bc08c38a6154f570f7626a101103768b78d94c35f82e2af5a5d91
    image: docker.io/kodekloud/nginx:latest
    imageID: docker.io/kodekloud/nginx@sha256:2862900861517dfaf9e0ed0f4fa199744a7410f4f78520866031c725c386bb5e
    lastState: {}
    name: nginx
    ready: false
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-05-11T18:08:42Z"
  hostIP: 192.26.253.9
  hostIPs:
  - ip: 192.26.253.9
  phase: Running
  podIP: 10.244.192.1
  podIPs:
  - ip: 10.244.192.1
  qosClass: BestEffort
  startTime: "2024-05-11T18:08:34Z"
```

```bash
k replace -f nginx1401.yaml --force
```

### Task #7: Create a cronjob

Create a cronjob called `dice` that runs every one minute. Use the Pod template located at `/root/throw-a-dice`. The image `throw-dice` randomly returns a value between `1` and `6`. The result of `6` is considered success and all others are failure.

The job should be non-parallel and complete the task once. Use a `backoffLimit` of `25`.

If the task is not completed within 20 seconds the job should fail and pods should be terminated.

You don't have to wait for the job completion. As long as the cronjob has been created as per the requirements.

```bash
k create cronjob dice --image=kodecloud/throw-dice --schedule="*/1 * * * *" --dry-run=client -o yaml > dice-cronjob.yaml
```

```yaml:dice-cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: dice
spec:
  schedule: '*/1 * * * *'
  jobTemplate:
    spec:
      completions: 1
      backoffLimit: 25
      activeDeadlineSeconds: 20
      template:
        spec:
          containers:
          - name: dice
            image: kodecloud/throw-dice
          restartPolicy: Never
```

```bash
k apply -f dice-cronjob.yaml
```

### Task #8: Create a pod with a secret

Create a pod called `my-busybox` in the `dev2406` namespace using the busybox image. The container should be called `secret` and should sleep for `3600` seconds.

The container should mount a read-only secret volume called `secret-volume` at the path `/etc/secret-volume`. The secret being mounted has already been created for you and is called `dotfile-secret`.

Make sure that the pod is scheduled on controlplane and no other node in the cluster.

```bash
k create ns dev2406
k run my-busybox --image=busybox -n dev2406 --dry-run=client -o yaml --command -- sleep 3600 > my-busybox.yaml
```

```yaml:my-busybox.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-busybox
  namespace: dev2406
spec:
  nodeName: controlplane
  containers:
  - name: secret
    image: busybox
    command:
    - sleep
    - "3600"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: /etc/secret-volume
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
```

```bash
k apply -f my-busybox.yaml
```

```bash
k describe po my-busybox -n dev2406
```

```bash
k get po my-busybox -n dev2406 -o jsonpath='{.spec.nodeName}'
```

```bash
k get no controlplane -o jsonpath='{.spec.taints}'
```

### Task #9: Create an Ingress resource

Create a single ingress resource called `ingress-vh-routing`. The resource should route HTTP traffic to multiple hostnames as specified below:

1. The service `video-service` should be accessible on <http://watch.ecom-store.com:30093/video>
2. The service `apparels-service` should be accessible on <http://apparels.ecom-store.com:30093/wear>

Here `30093` is the port used by the Ingress Controller

```bash
k create ingress ingress-vh-routing --rule="watch.ecom-store.com/video=video-service:80" --rule="apparels.ecom-store.com/wear=apparels-service:80" --dry-run=client -o yaml > ingress.yaml
```

```yaml:ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-vh-routing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /

spec:
  rules:
  - host: watch.ecom-store.com
    http:
      paths:
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /video
        pathType: Exact
  - host: apparels.ecom-store.com
    http:
      paths:
      - backend:
          service:
            name: apparels-service
            port:
              number: 8080
        path: /wear
        pathType: Exact
```

### Task #10: Create a pod with a security context

A pod called `dev-pod-dind-878516` has been deployed in the default namespace. Inspect the logs for the container called `log-x` and redirect the warnings to `/opt/dind-878516_logs.txt` on the controlplane node

```bash
k get po dev-pod-dind-878516 -o jsonpath='{.spec.nodeName}'
```

```bash
k logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt
```
