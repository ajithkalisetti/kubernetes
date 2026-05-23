````md
### Kubernetes Deployment File – Complete Guide

## What is a Deployment in Kubernetes?

A Deployment in Kubernetes is a YAML manifest used to manage applications declaratively.

Deployment helps us:

- Run multiple replicas
- Perform rolling updates
- Rollback versions
- Scale applications
- Configure containers
- Add health checks
- Attach storage
- Manage secrets/configurations
- Control scheduling

Deployment ensures the desired number of Pods are always running.

---

# Deployment Architecture

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Containers
````

* Deployment manages ReplicaSets
* ReplicaSet manages Pods
* Pods run Containers

---

# Complete Deployment Template

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-deployment
  namespace: dev

  labels:
    app: nginx
    env: dev
    tier: frontend

spec:

  replicas: 3

  revisionHistoryLimit: 5

  strategy:
    type: RollingUpdate

    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1

  selector:
    matchLabels:
      app: nginx

  template:

    metadata:
      labels:
        app: nginx
        env: dev

    spec:

      containers:

      - name: nginx-container

        image: nginx:1.25

        imagePullPolicy: IfNotPresent

        ports:
        - containerPort: 80

        env:
        - name: ENVIRONMENT
          value: "DEV"

        - name: LOG_LEVEL
          value: "INFO"

        envFrom:
        - configMapRef:
            name: nginx-config

        - secretRef:
            name: nginx-secret

        resources:

          requests:
            memory: "128Mi"
            cpu: "100m"

          limits:
            memory: "512Mi"
            cpu: "500m"

        livenessProbe:
          httpGet:
            path: /
            port: 80

          initialDelaySeconds: 10
          periodSeconds: 5

        readinessProbe:
          httpGet:
            path: /
            port: 80

          initialDelaySeconds: 5
          periodSeconds: 3

        startupProbe:
          httpGet:
            path: /
            port: 80

          failureThreshold: 30
          periodSeconds: 10

        volumeMounts:
        - name: nginx-storage
          mountPath: /usr/share/nginx/html

      volumes:
      - name: nginx-storage

        persistentVolumeClaim:
          claimName: nginx-pvc

      nodeSelector:
        disktype: ssd

      tolerations:
      - key: "app"
        operator: "Equal"
        value: "frontend"
        effect: "NoSchedule"

      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: zone
                operator: In
                values:
                - ap-south-1a

      imagePullSecrets:
      - name: dockerhub-secret

      restartPolicy: Always
```

---

# Detailed Explanation

---

# 1. apiVersion

Defines Kubernetes API version.

```yaml
apiVersion: apps/v1
```

Deployment uses:

* apps/v1

---

# 2. kind

Defines resource type.

```yaml
kind: Deployment
```

Examples:

* Pod
* Service
* Deployment
* StatefulSet
* DaemonSet

---

# 3. metadata

Contains Deployment information.

```yaml
metadata:
  name: nginx-deployment
  namespace: dev
```

## Common Metadata Fields

| Field       | Purpose                 |
| ----------- | ----------------------- |
| name        | Deployment name         |
| namespace   | Namespace               |
| labels      | Resource identification |
| annotations | Additional metadata     |

---

# 4. replicas

Defines number of Pod copies.

```yaml
replicas: 3
```

Kubernetes ensures:

* 3 Pods are always running.

---

# 5. strategy

Defines update method.

```yaml
strategy:
  type: RollingUpdate
```

---

## RollingUpdate

Updates Pods gradually.

```yaml
rollingUpdate:
  maxSurge: 1
  maxUnavailable: 1
```

| Field          | Meaning                  |
| -------------- | ------------------------ |
| maxSurge       | Extra Pods during update |
| maxUnavailable | Pods allowed to go down  |

---

# 6. selector

Deployment identifies Pods using labels.

```yaml
selector:
  matchLabels:
    app: nginx
```

IMPORTANT:
Selector labels must match Pod labels.

---

# 7. template

Defines Pod configuration.

```yaml
template:
```

Everything inside template becomes Pod specification.

---

# 8. containers

Defines containers inside Pod.

```yaml
containers:
- name: nginx-container
  image: nginx:1.25
```

---

# 9. imagePullPolicy

Controls image pull behavior.

```yaml
imagePullPolicy: IfNotPresent
```

## Options

| Value        | Meaning         |
| ------------ | --------------- |
| Always       | Pull every time |
| IfNotPresent | Pull if absent  |
| Never        | Never pull      |

---

# 10. ports

Defines application listening ports.

```yaml
ports:
- containerPort: 80
```

---

# 11. Environment Variables

---

## Static Variables

```yaml
env:
- name: ENVIRONMENT
  value: "DEV"
```

---

## ConfigMap Variables

```yaml
envFrom:
- configMapRef:
    name: nginx-config
```

---

## Secret Variables

```yaml
- secretRef:
    name: nginx-secret
```

Used for:

* Passwords
* Tokens
* API keys

---

# 12. Resources

Controls CPU and Memory.

```yaml
resources:
```

---

## Requests

Minimum guaranteed resources.

```yaml
requests:
  memory: "128Mi"
  cpu: "100m"
```

---

## Limits

Maximum usable resources.

```yaml
limits:
  memory: "512Mi"
  cpu: "500m"
```

---

# CPU Units

| Value | Meaning  |
| ----- | -------- |
| 1000m | 1 CPU    |
| 500m  | Half CPU |

---

# Memory Units

| Value | Meaning   |
| ----- | --------- |
| Mi    | Mebibytes |
| Gi    | Gibibytes |

---

# 13. Health Checks

Very important in production.

---

# Liveness Probe

Checks whether app is alive.

```yaml
livenessProbe:
```

If failed:

* Pod restarts.

---

# Readiness Probe

Checks whether app can receive traffic.

```yaml
readinessProbe:
```

If failed:

* Removed from Service endpoints.

---

# Startup Probe

Used for slow-starting applications.

```yaml
startupProbe:
```

Useful for:

* Java applications
* Spring Boot apps

---

# Probe Types

| Type      | Purpose        |
| --------- | -------------- |
| httpGet   | HTTP endpoint  |
| tcpSocket | TCP port check |
| exec      | Run command    |

---

# 14. Volumes

Used for persistent storage.

```yaml
volumes:
```

---

# Mounting Volumes

```yaml
volumeMounts:
```

Examples:

* Database storage
* Shared files
* Logs

---

# 15. nodeSelector

Schedules Pods on specific nodes.

```yaml
nodeSelector:
  disktype: ssd
```

---

# 16. tolerations

Allows Pods on tainted nodes.

```yaml
tolerations:
```

Used for:

* Dedicated nodes
* GPU nodes

---

# 17. affinity

Advanced scheduling rules.

```yaml
affinity:
```

Types:

* Node Affinity
* Pod Affinity
* Pod Anti-Affinity

---

# 18. imagePullSecrets

Used for private registries.

```yaml
imagePullSecrets:
- name: dockerhub-secret
```

Examples:

* DockerHub
* AWS ECR
* Azure ACR
* Google GCR

---

# 19. restartPolicy

Controls container restart behavior.

```yaml
restartPolicy: Always
```

Deployment supports:

* Always

---

# Common Deployment Features

| Feature         | Purpose            |
| --------------- | ------------------ |
| Replicas        | High availability  |
| Rolling updates | Zero downtime      |
| ConfigMaps      | External configs   |
| Secrets         | Sensitive data     |
| Probes          | Health monitoring  |
| Resources       | CPU/Memory control |
| Volumes         | Persistent storage |
| Affinity        | Smart scheduling   |

---

# Real-Time Production Commands

---

# Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

---

# Get Deployments

```bash
kubectl get deployment
```

---

# Describe Deployment

```bash
kubectl describe deployment nginx-deployment
```

---

# Get Pods

```bash
kubectl get pods
```

---

# View Logs

```bash
kubectl logs <pod-name>
```

---

# Delete Deployment

```bash
kubectl delete -f deployment.yaml
```

---

# Rolling Update

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.26
```

Updates Pods gradually without downtime.

---

# Rollback Deployment

```bash
kubectl rollout undo deployment nginx-deployment
```

Rollback to previous version.

---

# Scale Deployment

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

---

# Check Rollout Status

```bash
kubectl rollout status deployment nginx-deployment
```

---

# Deployment History

```bash
kubectl rollout history deployment nginx-deployment
```

---

# Best Practices

| Best Practice          | Reason                          |
| ---------------------- | ------------------------------- |
| Use labels properly    | Easy management                 |
| Add resource limits    | Prevent node exhaustion         |
| Use probes             | Improve reliability             |
| Use ConfigMaps/Secrets | Better configuration management |
| Use namespaces         | Environment separation          |
| Use replicas > 1       | High availability               |
| Use proper image tags  | Avoid unexpected updates        |

---

# Beginner Deployment Example

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: myapp

spec:
  replicas: 2

  selector:
    matchLabels:
      app: myapp

  template:

    metadata:
      labels:
        app: myapp

    spec:
      containers:
      - name: myapp-container
        image: nginx

        ports:
        - containerPort: 80
```

---

# Important Interview Topics

1. Deployment vs Pod
2. RollingUpdate vs Recreate
3. Liveness vs Readiness Probe
4. Requests vs Limits
5. ConfigMap vs Secret
6. Affinity vs NodeSelector
7. Deployment vs StatefulSet
8. Rollback mechanism
9. Scaling behavior
10. Desired state management

---

```
```
