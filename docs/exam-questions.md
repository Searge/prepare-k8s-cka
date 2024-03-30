# Exam Questions on Kubernetes CKA Exam

## Tips and Tricks

### Usefull aliases

```bash
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kcuc='kubectl config use-context'
```

### Usefull variables

```bash
export KUBE_EDITOR="vim"
export do="--dry-run=client -o yaml"
```

## Questions

### Question 1

**Context:**

You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace.

**Tasks:**

- Create a new ClusterRole named deployment-clusterrole, which only allows to create the following resource types:
  - Deployment
  - Stateful Set
  - DaemonSet
- Create a new ServiceAccount named cicd-token in the existing namespace app-team1.
- Bind the new ClusterRole deployment-clusterrole to the new ServiceAccount cicd-token, limited to the namespace app-team1.

**Task solution:**

```bash
k create clusterrole deployment-clusterrole --verb=create \
  --resource=deployment,statefulset,daemonset
k create sa cicd-token -n app-team1
k create clusterrolebinding cicd-token-binding \
  --clusterrole=deployment-clusterrole \
  --serviceaccount=app-team1:cicd-token
```

### Question 2

**Context:**

```bash
k config use-context ek8s
```

**Tasks:**

Set the node named ek8s-node-0 as unavailable and reschedule all the pods running on it.

**Task solution:**

```bash
k get no
k cordon ek8s-node-0
k drain ek8s-node-0 --ignore-daemonsets --delete-emptydir-data
```

### Question 3

**Context:**

```bash
k config use-context mk8s
```

**Tasks:**

Given an existing Kubernetes cluster running version 1.22.1,
upgrade all of the Kubernetes control plane and node components on the master node only to version 1.22.2.
Be sure to drain the master node before upgrading it and uncordon it after the upgrade.

You can ssh into the master node using the following command:

```bash
ssh mk8s-master-0
```

You can assume elevated privileges with the following command:

```bash
sudo -i
```

You are also expected to upgrade the kubelet and kubectl binaries on the master node.

> **Warning:** Do not upgrade the worker nodes, etcd, the container manager, the CNI plugin,
> the DNS service, or any other components.

**Task solution:**

```bash
k get no
NAME             STATUS   ROLES                  AGE   VERSION
mk8s-master-0    Ready    control-plane,master   10d   v1.22.1
mk8s-node-0      Ready    <none>                 10d   v1.22.1

k cordon mk8s-master-0
k drain mk8s-master-0 --ignore-daemonsets
k get no
NAME             STATUS                     ROLES                  AGE   VERSION
mk8s-master-0    Ready,SchedulingDisabled   control-plane,master   10d   v1.22.1
mk8s-node-0      Ready                      <none>                 10d   v1.22.1

ssh mk8s-master-0
sudo -i
apt-get update
apt-get install -y kubelet=1.22.2-00 kubectl=1.22.2-00 kubeadm=1.22.2-00
kubeadm upgrade node
kubeadm upgrade apply v1.22.2
systemctl restart kubelet
exit

k uncordon mk8s-master-0
k get no
NAME             STATUS   ROLES                  AGE   VERSION
mk8s-master-0    Ready    control-plane,master   10d   v1.22.2
mk8s-node-0      Ready    <none>                 10d   v1.22.1
```

### Question 4

**Context:**

No configuration context change is required.
Ensure, however, that you are have returned to the base node before proceeding.

**Tasks:**

First, create a snapshot of the existing etcd instance running at <https://127.0.0.1:2379>,
saving the snapshot to /var/lib/backup/etcd-snapshot.db.

The following TLS certificates/key are supplied for connecting to the server with etcdctl:

- CA certificate: /opt/KUIN00601/ca.crt
- Client certificate: /opt/KUIN00601/etcd-client.crt
- Client key: /opt/KUIN00601/etcd-client.key

Creating a snapshot of the given instance is expected to complete in seconds.
If the operation seems to hang , somthing's likely wrong with your command.
Use `ctrl`+`c` to cancel the operation and try again.

Next, restore an existing, previous snapshot located at /var/lib/backup/etcd-snapshot-previous.db.

**Task solution:**

```bash
export ETCDCTL_API=3
etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/opt/KUIN00601/ca.crt \
  --cert=/opt/KUIN00601/etcd-client.crt \
  --key=/opt/KUIN00601/etcd-client.key \
  snapshot save /var/lib/backup/etcd-snapshot.db

etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/opt/KUIN00601/ca.crt \
  --cert=/opt/KUIN00601/etcd-client.crt \
  --key=/opt/KUIN00601/etcd-client.key \
  snapshot status /var/lib/backup/etcd-snapshot-previous.db

sudo systemctl stop etcd.service

etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/opt/KUIN00601/ca.crt \
  --cert=/opt/KUIN00601/etcd-client.crt \
  --key=/opt/KUIN00601/etcd-client.key \
  snapshot restore /var/lib/backup/etcd-snapshot-previous.db

sudo systemctl start etcd.service
```

### Question 5

**Context:**

```bash
k config use-context hk8s
```

**Tasks:**

- Create a new NetworkPolicy named `allow-port-from-namespace` in the existing namespace `fubar`.
- Ensure that the new NetworkPolicy allows Pods in namespace internal to connect to port 9000 of Pods in namespace `fubar`.
- Further ensure that the new NetworkPolicy:
  - does not allow access to Pods, which don't listen on port 9000
  - does not allow access from Pods, which are not in namespace internal

**Task solution:**

```bash
# Label the namespace for the NetworkPolicy
k label ns fubar app=fubar

# Create the NetworkPolicy
cat <<EOF | k apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: fubar
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          app: fubar
    ports:
    - protocol: TCP
      port: 9000

```

- [The NetworkPolicy resource](https://kubernetes.io/docs/concepts/services-networking/network-policies/#networkpolicy-resource)

### Question 6

**Context:**

```bash
k config use-context k8s
```

**Tasks:**

- Reconfigure the existing deployment front-end and add a port specification named http exposing port 80/tcp of the existing container nginx.
- Create a new service named front-end-svc exposing the container port http.
- Configure the new service to also expose the individual Pods via a NodePort on the nodes on which they are scheduled.

**Task solution:**

```bash
k edit deploy front-end
```

```yaml
spec:
  template:
    spec:
      containers:
      - name: nginx
        ports:
        - containerPort: 80
          name: http
```

```bash
k expose deploy front-end --port=80 --target-port=80 --name=front-end-svc --type=NodePort
```

### Question 7

**Context:**

```bash
k config use-context k8s
```

**Tasks:**

- Scale the deployment presentation to 3 pods.

**Task solution:**

```bash
k scale deploy presentation --replicas=3
```

### Question 8

**Context:**

```bash
k config use-context k8s
```

**Tasks:**

- Schedule a pod as follows:
  - Name: `nginx-kusc00401`
  - Image: `nginx`
  - Node selector: `disk=ssd`

**Task solution:**

```bash
k run nginx-kusc00401 --image=nginx \
  --overrides='{"spec": {"nodeSelector": {"disk": "ssd"}}}' \
  --dry-run=client -o yaml > nginx-kusc00401.yaml

k apply -f nginx-kusc00401.yaml

```

### Question 9

**Context:**

```bash
k config use-context k8s
```

**Tasks:**

- Check to see how many nodes are ready (not including nodes tainted NoSchedule)
  and write the number to /opt/KUSC00402/kusc00402.txt.

**Task solution:**

```bash
k get no \
  -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}' \
  | grep 'node' | grep True | wc -l > /opt/KUSC00402/kusc00402.txt
```

### Question 10

**Context:**

```bash
k config use-context k8s
```

**Tasks:**

Schedule a Pod as follows:

- Name: `kucc8`
- App Containers: 2
- Container Name/Images:
  - `nginx`
  - `consul`

**Task solution:**

```bash
k run kucc8 --image=nginx --image=consul --dry-run=client -o yaml > kucc8.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kucc8
spec:
  containers:
  - name: nginx
    image: nginx
  - name: consul
    image: consul
```

```bash
k apply -f kucc8.yaml
```

### Question 11

**Context:**

```bash
k config use-context hk8s
```

**Tasks:**

- Create a persistent volume with name `app-data`, of capacity 2Gi and access mode ReadOnlyMany.
- The type of volume is hostPath and its location is `/srv/app-data`.

**Task solution:**

```bash
cat <<EOF | k apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-data
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadOnlyMany
  hostPath:
    path: /srv/app-data
EOF
```

[Reserving a PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reserving-a-persistentvolume)

### Question 12

**Context:**

```bash
k config use-context k8s
```

**Tasks:**

Monitor the logs of pod foo and:

- Extract log lines corresponding to error `file-not-found`
- Write them to `/opt/KUTR00101/foo`

**Task solution:**

```bash
k logs foo | grep 'file-not-found' > /opt/KUTR00101/foo
```

### Question 13

**Context:**

```bash
k config use-context k8s
```

An existing Pod needs to be integrated into the Kubernetes built-in logging architecture (e.g. kubectl logs).
Adding a streaming sidecar container is a good and common way to accomplish this requirement.

**Tasks:**

Add a sidecar container named sidecar, using the busybox image, to the existing Pod big-corp-app.
The new sidecar container has to run the following command:

```bash
/bin/sh -c 'tail -n+1 -f /var/log/big-corp-app.log'
```

Use a VolumeMount, mounted at /var/log, to make the log file `big-corp-app.log` available to the sidecar container.

> **Warning:** Don't modify the specification of the existing container other than adding the reqired volume mount.

**Task solution:**

```bash
k edit pod big-corp-app
```

```yaml
spec:
  containers:
  - name: big-corp-app
    volumeMounts:
    - name: log
      mountPath: /var/log
  - name: sidecar
    image: busybox
    command: ["/bin/sh", "-c", "tail -n+1 -f /var/log/big-corp-app.log"]
    volumeMounts:
    - name: log
      mountPath: /var/log
  volumes:
  - name: log
    emptyDir: {}
```

### Question 14

**Context:**

```bash
k config use-context k8s
```

**Tasks:**

From the pod label `name=overloaded-cpu`, find pods running high CPU workloads and write the name of the pod
consuming most CPU to the file `/opt/KUTR00401/KUTR00401.txt` (which already exists).

**Task solution:**

```bash
k top pod -l name=overloaded-cpu --sort-by=cpu | head -n 2 | tail -n 1 \
  | awk '{print $1}' > /opt/KUTR00401/KUTR00401.txt
```

### Question 15

**Context:**

```bash
k config use-context wk8s
```

**Tasks:**

A Kubernetes worker node, named `wk8s-node-0` is in state `NotReady`.

Investigate why this is the case, and perform any appropriate steps to bring the node to a `Ready` state,
ensuring that any changes are made permanent.

> **Note:** You can ssh to the filed node using:
> `ssh wk8s-node-0`
>
> You can assume elevated privileges with the following command:
> `sudo -i`

**Task solution:**

```bash
k get no
NAME             STATUS   ROLES                  AGE   VERSION
wk8s-master-0    Ready    control-plane,master   10d   v1.23.1
wk8s-node-0      NotReady <none>                 10d   v1.23.1
wk8s-node-1      Ready    <none>                 10d   v1.23.1

k describe no wk8s-node-0
...
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Tue, 01 Jan 0001 00:00:00 +0000   Tue, 01 Jan 0001 00:00:00 +0000   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Tue, 01 Jan 0001 00:00:00 +0000   Tue, 01 Jan 0001 00:00:00 +0000   KubeletHasSufficientMemory   Kubelet stopped posting node status.
  DiskPressure         False   Tue, 01 Jan 0001 00:00:00 +0000   Tue, 01 Jan 0001 00:00:00 +0000   KubeletHasNoDiskPressure     Kubelet stopped posting node status.
  PIDPressure          False   Tue, 01 Jan 0001 00:00:00 +0000   Tue, 01 Jan 0001 00:00:00 +0000   KubeletHasSufficientPID      Kubelet stopped posting node status.
  Ready                False   Tue, 01 Jan 0001 00:00:00 +0000   Tue, 01 Jan 0001 00:00:00 +0000   KubeletNotReady              Kubelet stopped posting node status.
...

ssh wk8s-node-0
sudo -i
systemctl restart kubelet # Ensure it is enabled too. So change remains active after reboot
systemctl enable kubelet
exit

k get no
NAME             STATUS   ROLES                  AGE   VERSION
wk8s-master-0    Ready    control-plane,master   10d   v1.23.1
wk8s-node-0      Ready    <none>                 10d   v1.23.1
wk8s-node-1      Ready    <none>                 10d   v1.23.1
```

### Question 16

**Context:**

```bash
k config use-context ok8s
```

**Tasks:**

- Create a new `PersistentVolumeClaim`:
  - Name: `pv-volume`
  - Class: `csi-hostpath-sc`
  - Capacity: `10Mi`
- Create a new Pod which mounts the `PersistentVolumeClaim` as a volume:
  - Name: `web-server`
  - Image: `nginx`
  - Mount path: `/usr/share/nginx/html`
- Configure the new Pod to have `ReadWriteOnce` access on the volume.
- Finally, using kubectl edit or kubectl patch expand the `PersistentVolumeClaim` to a capacity of 70Mi and record that change.

**Task solution:**

```bash
cat <<EOF | k apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
  storageClassName: csi-hostpath-sc
EOF
```

```bash
cat <<EOF | k apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: pv-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: pv-volume
    persistentVolumeClaim:
      claimName: pv-volume
EOF
```

```bash
k edit pvc pv-volume
```

```yaml
spec:
  resources:
    requests:
      storage: 70Mi
```

### Question 17

**Context:**

```bash
k config use-context ok8s
```

**Tasks:**

- Create a new nginx Ingress resource as follows:
  - Name: `pong`
  - Namespace: `ing-internal`
  - Exposing service hello on path `/hello` using service port `5678`

> **Note:** The availability of service `hello` can be checked using the following command,
> which should return `hello``
>
> ```bash
> curl -kL <INTERNAL_IP>/hello
> ```
>

**Task solution:**

```bash
cat <<EOF | k apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pong
  namespace: ing-internal
spec:
  rules:
  - http:
      paths:
      - path: /hello
        pathType: ImplementationSpecific
        backend:
          service:
            name: hello
            port:
              number: 5678
EOF
```
