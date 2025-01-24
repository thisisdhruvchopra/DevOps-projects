# A Typical YAML file looks like this : 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25% # Up to 25% of Pods can be unavailable during an update
      maxSurge: 50%       # Up to 50% more Pods can be created during the rollout
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80

```
# Explanation of Kubernetes Deployment YAML

This document explains each line of the given Kubernetes YAML file in detail.

---

## **1. Top-Level Fields**
```yaml
apiVersion: apps/v1
```
- **Definition**: Specifies the API version of the Kubernetes resource you're defining.
- **Purpose**: 
  - `apps/v1` indicates that this YAML defines resources under the `apps` API group, version `v1`. 
  - Different API versions correspond to specific resource capabilities and functionalities in Kubernetes.
- **Why Important**: The API version must match the Kubernetes cluster version. If it’s outdated or incorrect, the cluster won’t recognize the resource.

```yaml
kind: Deployment
```
- **Definition**: Specifies the type of Kubernetes resource being created.
- **Purpose**: 
  - `Deployment` defines a resource that manages **Pods**, handles updates to them, and ensures the desired state of the application.
  - It is commonly used for applications requiring rolling updates, scaling, and self-healing.

```yaml
metadata:
  name: nginx-deployment
```
- **Definition**: Contains metadata about the Deployment.
- **`name`**: The unique identifier for the Deployment within its namespace.
- **Purpose**: 
  - The Deployment will be referenced as `nginx-deployment` in the cluster. 
  - This name helps differentiate it from other resources.

```yaml
  labels:
    app: nginx
```
- **Definition**: Labels are key-value pairs used for identifying and organizing Kubernetes resources.
- **Purpose**: 
  - The label `app: nginx` serves as a tag that can be used by other resources (like Services) to locate this Deployment.
  - Labels are used for grouping, filtering, and selecting resources.

---

## **2. Specifying the Deployment Specification**
```yaml
spec:
  replicas: 3
```
- **Definition**: Describes the desired state of the Deployment.
- **`replicas`**: Specifies the number of Pod replicas Kubernetes should maintain at all times.
- **Purpose**: 
  - In this case, 3 replicas of the `nginx` pod will always run, ensuring high availability and load distribution.
  - Kubernetes ensures that 3 Pods are always running by creating or deleting Pods as needed.

```yaml
  selector:
    matchLabels:
      app: nginx
```
- **Definition**: Specifies the label(s) that the Deployment uses to identify the Pods it manages.
- **Purpose**: 
  - The selector ensures that only Pods with the label `app: nginx` are managed by this Deployment.
  - If the labels on Pods don’t match the selector, the Deployment won’t recognize or control them.

---

## **3. Pod Template**
```yaml
  template:
    metadata:
      labels:
        app: nginx
```
- **Definition**: Defines the template for the Pods that the Deployment creates.
- **Purpose**: 
  - **`metadata`**: Assigns labels to the Pods created by this Deployment.
  - The `app: nginx` label ensures that these Pods match the Deployment’s selector and are recognized as part of the Deployment.

```yaml
    spec:
      containers:
```
- **Definition**: Specifies the configuration of containers within each Pod.
- **Purpose**: 
  - A Pod can run one or more containers, and this section defines their behavior and configuration.
 

## Rolling Update 
It is the default deployment strategy in Kubernetes. It replaces old Pods with new ones incrementally to ensure high availability and zero downtime.

**Key Concepts**

1. **Gradual Pod Replacement**: Old Pods are replaced incrementally, ensuring some Pods always remain available.
2. **Rollback Support**: Easily rollback to a previous version if issues occur.

## Configuration

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 25% # Maximum Pods unavailable during update
    maxSurge: 50%       # Maximum additional Pods created during update
```

- **`maxUnavailable`**: Maximum Pods unavailable at a time (absolute number or percentage).
- **`maxSurge`**: Maximum Pods created above desired replicas during the update.

## Benefits

- High availability during updates.
- Controlled, incremental rollout.
- Supports rollbacks for stability.

## Example Command

Rollback if needed:
```bash
kubectl rollout undo deployment/<deployment-name>
```

Rolling Update ensures your applications are updated with minimal downtime and maximum reliability.


---

## **4. Container Configuration**
```yaml
        - name: nginx
```
- **Definition**: The name of the container in the Pod.
- **Purpose**: 
  - Helps identify this container within the Pod.
  - Useful when multiple containers are running in the same Pod.

```yaml
          image: nginx:1.14.2
```
- **Definition**: Specifies the Docker image for the container.
- **Purpose**: 
  - This tells Kubernetes to pull the `nginx` image version `1.14.2` from a container registry (e.g., Docker Hub) and run it inside the Pod.
  - Docker images contain the application code and its dependencies.

```yaml
          ports:
            - containerPort: 80
```
- **Definition**: Specifies the ports that the container exposes.
- **Purpose**: 
  - **`containerPort: 80`**: Exposes port 80 on the container for HTTP traffic.
  - This port is where the application listens for incoming requests.
  - This configuration is crucial for connecting Services or exposing the application externally.

---

## **Summary of the YAML**
This YAML defines a **Deployment** named `nginx-deployment` that:
1. Creates **3 replicas** of Pods.
2. Ensures Pods are labeled `app: nginx` and uses this label for management and filtering.
3. Each Pod runs a single **nginx** container using the `nginx:1.14.2` Docker image.
4. Exposes **port 80** on the container for HTTP communication.

---

### **How This YAML Works in Kubernetes**
1. The Deployment ensures the desired number of Pods (`replicas`) is always running.
2. Kubernetes uses the `selector` to manage Pods with matching labels.
3. Each Pod created by the Deployment uses the specified template, including the `nginx` container.
4. The exposed port allows other Kubernetes resources (like Services) to route traffic to the Pods.
