
# Kubernetes Security Concepts

This document provides an explanation of key Kubernetes security concepts such as namespaces, service accounts, roles, cluster roles, role bindings, and service access tokens with real-world examples.

---

## **1. Namespaces**
Namespaces in Kubernetes are a way to divide a single cluster into multiple virtual clusters. They are useful for organizing and isolating resources within a cluster.

### **Real-World Example**:
Imagine a company running multiple projects in a single Kubernetes cluster. Each project can have its own namespace to isolate its resources.

- **Namespace A:** Resources for the Development Team.
- **Namespace B:** Resources for the QA Team.
- **Namespace C:** Resources for the Production Team.

This ensures:
- Resource isolation: The Development namespace cannot interfere with the Production namespace.
- Simplified resource management: Teams can manage their namespaces independently.

### **Commands**:
- Create a namespace:
  ```bash
  kubectl create namespace dev
  ```
- List namespaces:
  ```bash
  kubectl get namespaces
  ```

---

## **2. Service Account**
Service accounts are used by applications or Pods to authenticate to the Kubernetes API or other external systems. By default, every Pod is associated with a default service account in its namespace.

### **Real-World Example**:
Imagine an application running in a Pod that needs to retrieve secrets from Kubernetes. Instead of using a user’s credentials, the application uses a service account with limited permissions.

- **Default Service Account:** Automatically assigned to all Pods in the namespace unless specified otherwise.
- **Custom Service Account:** Created for specific use cases to enforce stricter security.

### **Commands**:
- Create a service account:
  ```bash
  kubectl create serviceaccount my-service-account -n dev
  ```
- Assign a service account to a Pod:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
    namespace: dev
  spec:
    serviceAccountName: my-service-account
    containers:
    - name: my-container
      image: nginx
  ```

---

## **3. Role**
A Role is used to define permissions within a specific namespace. It grants fine-grained access control over Kubernetes resources such as Pods, ConfigMaps, and Secrets.

### **Real-World Example**:
A DevOps engineer wants to allow developers to view Pods and logs in the `dev` namespace but prevent them from deleting any resources.

### **Commands**:
- Define a Role:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: dev
    name: pod-reader
  rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
  ```

---

## **4. ClusterRole**
ClusterRoles provide permissions at the cluster level or across all namespaces. They are useful for granting permissions that span multiple namespaces or managing cluster-wide resources.

### **Real-World Example**:
A system administrator needs the ability to monitor all resources across the cluster, including nodes, storage, and namespaces.

### **Commands**:
- Define a ClusterRole:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: cluster-admin-monitor
  rules:
  - apiGroups: [""]
    resources: ["nodes", "pods", "namespaces"]
    verbs: ["get", "list", "watch"]
  ```

---

## **5. RoleBinding and ClusterRoleBinding**
RoleBindings and ClusterRoleBindings are used to associate Roles and ClusterRoles with users, groups, or service accounts. 

- **RoleBinding:** Grants permissions defined in a Role within a specific namespace.
- **ClusterRoleBinding:** Grants permissions defined in a ClusterRole across the cluster.

### **Real-World Example**:
- **RoleBinding:** A developer in the `dev` namespace is granted permissions to view and list Pods but not delete them.
- **ClusterRoleBinding:** A monitoring service is granted permissions to monitor all namespaces in the cluster.

### **Commands**:
- Create a RoleBinding:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: pod-reader-binding
    namespace: dev
  subjects:
  - kind: User
    name: developer
    apiGroup: ""
  roleRef:
    kind: Role
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
  ```

- Create a ClusterRoleBinding:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: cluster-monitor-binding
  subjects:
  - kind: ServiceAccount
    name: monitoring-service-account
    namespace: monitoring
  roleRef:
    kind: ClusterRole
    name: cluster-admin-monitor
    apiGroup: rbac.authorization.k8s.io
  ```

---

## **6. Service Access Tokens**
Service Access Tokens are credentials issued to service accounts to allow authentication with the Kubernetes API or external systems. These tokens are mounted into Pods by default and can be used by applications running within the Pods to authenticate securely.

### **Real-World Example**:
An application running in a Pod needs to call the Kubernetes API to fetch configuration details or update resources. The application uses a service access token issued to its associated service account.

- **Default Token:** Automatically mounted in Pods with the default service account.
- **Custom Token:** Issued for a specific service account to provide scoped access.

### **Commands**:
- Retrieve the token for a service account:
  ```bash
  kubectl get secret $(kubectl get serviceaccount my-service-account -n dev -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 --decode
  ```
- Disable automatic mounting of the token (for enhanced security):
  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: my-secure-service-account
    namespace: dev
  automountServiceAccountToken: false
  ```

---

## **Summary Table**
| Concept               | Scope            | Purpose                                         | Example Use Case                              |
|-----------------------|------------------|------------------------------------------------|----------------------------------------------|
| **Namespace**         | Cluster-wide     | Isolate resources within a cluster             | Separate environments for dev, QA, prod      |
| **Service Account**   | Namespace        | Authenticate Pods or applications              | Pod accessing Kubernetes secrets securely     |
| **Role**              | Namespace        | Define permissions for a namespace             | Allow developers to view Pods in `dev`       |
| **ClusterRole**       | Cluster-wide     | Define permissions across the cluster          | Allow monitoring of all resources            |
| **RoleBinding**       | Namespace        | Bind a Role to a user/group/service account    | Grant dev team view-only access to Pods      |
| **ClusterRoleBinding**| Cluster-wide     | Bind a ClusterRole to a user/group/service account | Grant monitoring service access across all namespaces |
| **Service Access Tokens**| Namespace or Cluster | Authenticate using service accounts          | Application calling Kubernetes API securely  |

---

By combining these security components, Kubernetes provides a flexible and powerful mechanism to enforce resource isolation, access control, and secure communication between applications.
