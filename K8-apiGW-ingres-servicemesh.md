# Kubernetes Networking, Services, Ingress & API Gateway Concepts

# Kubernetes Basics

Kubernetes orchestrates containers.

Provides:
- Deployment
- Scaling
- Self-healing
- Load balancing
- Service discovery

---

# Pods

Smallest deployable unit in Kubernetes.

Usually:
```text
1 Pod = 1 Container
```

Pods are temporary.

---

# Deployments

Deployments manage pods.

Features:
- Scaling
- Rolling updates
- Rollbacks
- Self-healing

---

# Services

Services provide:
- Stable IP
- Stable DNS
- Load balancing

---

# 2. Kubernetes Services

# Why Services Exist

Pods are temporary:
- IPs change
- Pods restart
- Pods reschedule

Services provide stable communication.

---

# Types of Services

## ClusterIP

Internal communication only.

Example:
```text
Frontend → Backend API
```

---

## NodePort

Exposes app using:

```text
<NodeIP>:<Port>
```

Example:
```text
192.168.1.10:30080
```

---

## LoadBalancer

Creates cloud load balancer.

Used in:
- AWS
- Azure
- GCP

---

## Headless Service

No cluster IP.
No load balancing.

Provides direct pod communication.

Used for:
- Databases
- Kafka
- Elasticsearch

---

# 3. Headless Service

# What is Headless Service?

A special service with:

```yaml
clusterIP: None
```

Meaning:
- No virtual IP
- No load balancing
- Direct pod DNS resolution

---

# Normal Service

```text
Client
   ↓
Service IP
   ↓
Random Pod
```

---

# Headless Service

```text
Client
   ↓
Specific Pod
```

---

# Why Needed?

Stateful applications require:
- Stable identity
- Direct pod communication

Examples:
- Kafka
- MySQL
- Redis Cluster

---

# Example DNS

```text
mysql-0.mysql-headless
mysql-1.mysql-headless
mysql-2.mysql-headless
```

---

# Key Point

```text
Headless Service gives direct pod access.
```

---

# 4. StatefulSet

# What is StatefulSet?

Used for stateful applications.

Provides:
- Stable pod names
- Stable DNS
- Stable storage
- Ordered deployment

---

# Example Pod Names

```text
mysql-0
mysql-1
mysql-2
```

Names never change.

---

# Why StatefulSet?

Databases require:
- Persistent storage
- Stable identity
- Ordered startup

---

# StatefulSet Features

## Stable Identity

```text
mysql-1 always remains mysql-1
```

---

## Stable Storage

Same volume reattached after restart.

---

## Ordered Deployment

```text
mysql-0 starts first
mysql-1 starts second
```

---

# StatefulSet + Headless Service

Common architecture:

```text
Headless Service
        ↓
StatefulSet
        ↓
Pods
```

---

# What Happens During Pod Restart?

Even if pod IP changes:
- DNS remains same
- Storage remains same

Example:

```text
mysql-1.mysql-headless
```

always works.

---

# 5. VM NGINX vs Kubernetes Ingress

# Traditional VM Architecture

```text
Internet
   ↓
NGINX
   ↓
Backend Services
```

Example NGINX upstream:

```nginx
upstream userservice {
    server 10.0.0.10:8080;
}
```

---

# Kubernetes Equivalent

```text
Internet
   ↓
Ingress Controller
   ↓
Ingress Rules
   ↓
Services
   ↓
Pods
```

---

# Mapping

| VM World | Kubernetes |
|---|---|
| NGINX upstream | Service |
| location block | Ingress Rule |
| Reverse proxy | Ingress Controller |

---

# 6. Ingress in Kubernetes

# What is Ingress?

Ingress provides routing rules.

Ingress Controller handles traffic.

---

# Traffic Flow

```text
Internet
   ↓
Ingress Controller
   ↓
Services
   ↓
Pods
```

---

# Path-Based Routing

```text
/users     → user-service
/payments  → payment-service
```

---

# Host-Based Routing

```text
api.myapp.com   → api-service
admin.myapp.com → admin-service
```

---

# TLS/HTTPS Ingress

Ingress can terminate SSL/TLS.

---

# Ingress Controllers

Popular ingress controllers:
- NGINX Ingress
- Traefik
- APISIX
- HAProxy
- Istio Gateway

---

# 7. APISIX and API Gateway

# What is APISIX?

APISIX is:

```text
Ingress Controller
        +
API Gateway
```

---

# What is API Gateway?

Centralized traffic manager for APIs.

---

# Traffic Flow

```text
Users
   ↓
API Gateway
   ↓
Microservices
```

---

# Why API Gateway?

Without gateway:
- Every service handles auth
- Every service handles rate limiting
- Difficult monitoring

Gateway centralizes everything.

---

# API Gateway Responsibilities

- Authentication
- Authorization
- Rate limiting
- Traffic management
- Monitoring
- Logging
- SSL termination

---

# 8. API Gateway Features

# Authentication

Validates:
- JWT
- OAuth
- API keys

---

# Rate Limiting

Restricts requests.

Example:
```text
100 requests/minute
```

---

# Traffic Splitting

Example:

```text
90% → v1
10% → v2
```

Used in canary deployments.

---

# Canary Deployment

Gradual rollout strategy.

---

# Observability

Collects:
- Metrics
- Logs
- Latency
- Errors

Integrated with:
- Prometheus
- Grafana
- Jaeger

---

# Plugin Architecture

Plugins provide extra features.

Examples:
- JWT auth
- Rate limiting
- CORS
- IP restriction

---

# 9. APISIX vs Kong vs Istio vs NGINX Ingress

# NGINX Ingress

Purpose:
- External traffic routing

Best for:
- Simple ingress
- Web applications

---

# Kong

Purpose:
- API management

Best for:
- Public APIs
- Enterprise APIs

---

# APISIX

Purpose:
- Cloud-native API Gateway

Best for:
- Kubernetes-native traffic management
- API gateway features

---

# Istio

Purpose:
- Service Mesh

Controls:
- Internal service-to-service communication

Provides:
- mTLS
- Circuit breaking
- Advanced observability

---

# Feature Comparison

| Feature | NGINX Ingress | Kong | APISIX | Istio |
|---|---|---|---|---|
| Ingress Routing | Yes | Yes | Yes | Yes |
| API Gateway | Limited | Yes | Yes | Partial |
| Service Mesh | No | No | No | Yes |
| Authentication | Basic | Advanced | Advanced | Advanced |
| Rate Limiting | Basic | Advanced | Advanced | Advanced |
| Traffic Splitting | Limited | Yes | Yes | Yes |
| mTLS | No | No | No | Yes |
| Complexity | Low | Medium | Medium | High |

---

# 10. Important Commands

# Services

```bash
kubectl get svc
kubectl describe svc
```

---

# Ingress

```bash
kubectl get ingress -A
kubectl describe ingress
```

---

# APISIX Routes

```bash
kubectl get apisixroute -A
kubectl describe apisixroute
```

---

# API Resources

```bash
kubectl api-resources
```

---

# Check Ingress Controllers

```bash
kubectl get pods -A | grep -i ingress
kubectl get pods -A | grep -i apisix
```

---

# 11. Real-World Kubernetes Traffic Flow

# Basic Ingress Architecture

```text
Internet
   ↓
LoadBalancer
   ↓
Ingress Controller
   ↓
Service
   ↓
Pods
```

---

# API Gateway Architecture

```text
Internet
   ↓
API Gateway (APISIX/Kong)
   ↓
Authentication
Rate limiting
Traffic control
Observability
   ↓
Services
   ↓
Pods
```

---

# Service Mesh Architecture

```text
Service A
   ↓
Envoy Proxy
   ↓
Service B
```

Controlled by Istio.

---

# Important Final Understanding

## Ingress

Routes external traffic.

---

## API Gateway

Manages API traffic intelligently.

---

## Service Mesh

Controls internal service-to-service traffic.

---

# Simplified Memory Trick

```text
Ingress
→ Routes traffic

API Gateway
→ Manages traffic

Service Mesh
→ Controls internal traffic
```

---