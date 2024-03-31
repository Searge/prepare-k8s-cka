# Required fields in Kubernetes manifests

This document lists the required fields in Kubernetes manifests.

- [Required fields in Kubernetes manifests](#required-fields-in-kubernetes-manifests)
  - [Required fields](#required-fields)
    - [Pod fields](#pod-fields)
    - [Deployment fields](#deployment-fields)
  - [Examples](#examples)
    - [Pod](#pod)
    - [Deployment](#deployment)
    - [StatefulSet](#statefulset)
    - [DaemonSet](#daemonset)
    - [Job](#job)
    - [CronJob](#cronjob)
    - [Service](#service)
    - [Ingress](#ingress)
    - [ConfigMap](#configmap)
    - [Secret](#secret)
    - [PersistentVolume](#persistentvolume)
    - [PersistentVolumeClaim](#persistentvolumeclaim)
    - [ServiceAccount](#serviceaccount)
    - [Role](#role)
    - [RoleBinding](#rolebinding)
    - [ClusterRole](#clusterrole)
    - [ClusterRoleBinding](#clusterrolebinding)
    - [NetworkPolicy](#networkpolicy)
    - [PodSecurityPolicy](#podsecuritypolicy)
    - [CustomResourceDefinition](#customresourcedefinition)

## Required fields

In the manifest (YAML or JSON file) for the Kubernetes object you want to create, you'll need to set values for the following fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object
- `kind` - What kind of object you want to create
- `metadata` - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`
- `spec` - What state you desire for the object

### Pod fields

- `apiVersion`
- `kind`
- `metadata.name`
- `spec.containers[].name`
- `spec.containers[].image`
- `spec.containers[].command`
- `spec.volumes[].name`
- `spec.volumes[].emptyDir`
- `spec.volumes[].configMap`
- `spec.volumes[].secret`
- `spec.volumes[].persistentVolumeClaim`
- `spec.volumes[].hostPath`

### Deployment fields

- `apiVersion`
- `kind`
- `metadata.name`
- `spec.replicas`
- `spec.selector.matchLabels`
- `spec.template.metadata.labels`
- `spec.template.spec.containers[].name`
- `spec.template.spec.containers[].image`
- `spec.template.spec.containers[].command`
- `spec.template.spec.volumes[].name`
- `spec.template.spec.volumes[].emptyDir`
- `spec.template.spec.volumes[].configMap`
- `spec.template.spec.volumes[].secret`
- `spec.template.spec.volumes[].persistentVolumeClaim`

## Examples

### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  containers:
  - name: my-container
    image: busybox:1.28
    command: ["sh", "-c", "echo Hello, Kubernetes! && sleep 3600"]
    volumeMounts:
    - name: my-volume
      mountPath: /data
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  volumes:
  - name: my-volume
    emptyDir: {}
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: busybox:1.28
        command: ["sh", "-c", "echo Hello, Kubernetes! && sleep 3600"]
        volumeMounts:
        - name: my-volume
          mountPath: /data
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
      volumes:
      - name: my-volume
        emptyDir: {}
```

### StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  serviceName: my-service
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: busybox:1.28
        command: ["sh", "-c", "echo Hello, Kubernetes! && sleep 3600"]
        volumeMounts:
        - name: my-volume
          mountPath: /data
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
      volumes:
      - name: my-volume
        emptyDir: {}
```

### DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: busybox:1.28
        command: ["sh", "-c", "echo Hello, Kubernetes! && sleep 3600"]
        volumeMounts:
        - name: my-volume
          mountPath: /data
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
      volumes:
      - name: my-volume
        emptyDir: {}
```

### Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: busybox:1.28
        command: ["sh", "-c", "echo Hello, Kubernetes! && sleep 3600"]
        volumeMounts:
        - name: my-volume
          mountPath: /data
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
      volumes:
      - name: my-volume
        emptyDir: {}
```

### CronJob

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cronjob
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: my-app
        spec:
          containers:
          - name: my-container
            image: busybox:1.28
            command: ["sh", "-c", "echo Hello, Kubernetes! && sleep 3600"]
            volumeMounts:
            - name: my-volume
              mountPath: /data
            resources:
              requests:
                memory: "64Mi"
                cpu: "250m"
              limits:
                memory: "128Mi"
                cpu: "500m"
          volumes:
          - name: my-volume
            emptyDir: {}
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  rules:
  - host: my-host
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
  annotations:
    key1: value1
  labels:
    key1: value1
data:
  key1: value1
```

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  annotations:
    key1: value1
  labels:
    key1: value1
type: Opaque
data:
  key1: dmFsdWUx
```

### PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: my-storage-class
  hostPath:
    path: /data
```

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: my-storage-class
```

### ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  annotations:
    key1: value1
  labels:
    key1: value1
```

### Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-role
  annotations:
    key1: value1
  labels:
    key1: value1
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
```

### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-role-binding
  annotations:
    key1: value1
  labels:
    key1: value1
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: my-role
subjects:
- kind: ServiceAccount
  name: my-service-account
```

### ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: my-cluster-role
  annotations:
    key1: value1
  labels:
    key1: value1
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
```

### ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-cluster-role-binding
  annotations:
    key1: value1
  labels:
    key1: value1
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: my-cluster-role
subjects:
- kind: ServiceAccount
  name: my-service-account
```

### NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr:
        except:
        except:
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - ipBlock:
        cidr:
        except:
        except:
    ports:
    - protocol: TCP
      port: 80
```

### PodSecurityPolicy

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: my-psp
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
  - ALL
  volumes:
  - configMap
  - emptyDir
  - secret
  hostNetwork: false
  hostPorts:
  - min: 0
    max: 65535
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  readOnlyRootFilesystem: false
```

### CustomResourceDefinition

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: my-crd
  annotations:
    key1: value1
  labels:
    key1: value1
spec:
  group: my-group
  names:
    kind: MyKind
    listKind: MyKindList
    plural: mykinds
    singular: mykind
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              field1:
                type: string
```
