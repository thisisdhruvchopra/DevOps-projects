
# Ingress in Kubernetes

## Introduction

In Kubernetes, **Ingress** is a resource that manages external access to the services within a cluster, typically HTTP/HTTPS. It provides a set of rules for routing traffic from outside the cluster to services based on the incoming request’s URL, host, or other criteria. Essentially, an Ingress acts as a reverse proxy to direct incoming requests to the appropriate service within the cluster.

### Key Concepts

1. **Ingress Controller**:
   The Ingress resource itself does not handle the traffic. It is the Ingress Controller that implements the rules defined in the Ingress resource. The controller is a piece of software (like Nginx, Traefik, or HAProxy) that listens to the Ingress resource and manages routing traffic accordingly.
   
2. **Ingress Resource**:
   The Ingress resource defines how HTTP/HTTPS requests should be routed to different services within the Kubernetes cluster. It includes paths, hosts, and the backend services to which the requests should be forwarded.

3. **Backend Services**:
   These are the actual services running within the cluster that will handle the requests. A service can be any type of backend application, like a web server or API endpoint.

4. **Ingress Rules**: 
   These are the routing rules that specify how the traffic should be directed based on URL, host, and path.

5. **TLS**:
   Ingress can also be used to handle SSL/TLS termination, which means handling encrypted HTTPS traffic and forwarding unencrypted HTTP traffic to the backend services.

---

## Example

### Step 1: Deploying an Ingress Controller

Before using Ingress, you need to deploy an Ingress Controller. Here’s how you can deploy the **Nginx Ingress Controller** on Kubernetes:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

This will install the Nginx Ingress Controller, which listens to the Ingress resources in the cluster and routes traffic accordingly.

### Step 2: Creating an Ingress Resource

Now, let's assume you have two services running within your Kubernetes cluster—`service1` and `service2`. You want to route traffic from different URL paths to these services.

#### 1. Create services for your applications:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service1
spec:
  selector:
    app: app1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: service2
spec:
  selector:
    app: app2
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

#### 2. Create the Ingress resource to route traffic:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80
```

In this example:
- Traffic coming to `myapp.example.com/service1` will be routed to `service1`.
- Traffic coming to `myapp.example.com/service2` will be routed to `service2`.

### Step 3: Exposing Ingress to External Traffic

To allow external traffic to reach your Ingress Controller, you may need to expose it via a LoadBalancer or NodePort depending on your Kubernetes setup. For example, with an Nginx Ingress Controller:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  labels:
    app: ingress-nginx
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  - port: 443
    targetPort: 443
    protocol: TCP
  selector:
    app: ingress-nginx
  type: LoadBalancer
```

This exposes the Ingress Controller externally, allowing traffic to be routed to the Ingress resources based on the rules you’ve defined.

---

## Handling SSL/TLS with Ingress

You can configure TLS termination directly in your Ingress resource by specifying a secret that holds your TLS certificate and key. Here's how you can modify the previous Ingress example to support HTTPS:

#### 1. Create a TLS secret:

```bash
kubectl create secret tls my-tls-secret --cert=path/to/cert.crt --key=path/to/cert.key
```

#### 2. Update the Ingress resource to use TLS:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: my-tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80
```

Now, traffic to `myapp.example.com` will be encrypted (HTTPS) and the Ingress Controller will handle decrypting the traffic before routing it to the appropriate backend service.

---

## Why We Don't Specify Service Type in Ingress

### 1. **Ingress and ClusterIP Services**
   - **ClusterIP** is the default service type in Kubernetes, and it exposes the service only within the cluster. This is the type that the Ingress resource typically interacts with.
   - Ingress is designed to handle HTTP/HTTPS traffic coming from outside the cluster and route it to **ClusterIP services** inside the cluster, which means it doesn’t need to expose these services directly to the outside world.
   - When you create a service with type `ClusterIP`, Kubernetes ensures that the service is accessible only from within the cluster (e.g., via internal DNS), and the Ingress controller will handle the routing externally.

### 2. **Ingress Controller**
   - The Ingress controller (like Nginx, Traefik, etc.) acts as a reverse proxy, and it knows how to reach the ClusterIP services behind the scenes. It handles the external requests and then forwards them to the internal services based on the rules defined in the Ingress resource.
   - The Ingress Controller is typically exposed via a `LoadBalancer` or `NodePort` service to route external traffic into the cluster. However, this exposure does not affect how the Ingress resource works with the backend services.

### 3. **When Would You Specify Service Type?**
You **do not need** to specify service types in Ingress, but you would use other types (like `LoadBalancer` or `NodePort`) when you are dealing with services that need to be exposed directly to the outside world **without the Ingress** (i.e., outside the control of the Ingress routing mechanism). However, with Ingress in place, your backend services generally remain internal (ClusterIP).
