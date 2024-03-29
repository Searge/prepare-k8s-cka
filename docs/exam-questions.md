# Exam Questions on Kubernetes CKA Exam

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
