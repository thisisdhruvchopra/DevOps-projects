# Kubernetes Storage : Manual Provisioning
---



#  ***Step 1 : Create PersistentVolume.yaml (PV)***
 ## PV
 ```yaml
 apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  hostPath:
    path: /mnt/data
 ```
 
# ***Step 2 : Create PersistentVolumeClain.yaml (PVC)***
 ## PVC
 ```yaml
 apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: local-storage
 ```

**Things to note** : 
- Match the access mode exactly
- Match the Storage Class exactly
- Storage of PVC <= Storage of PV. Only then we will be able to claim it.

# ***Step 3 : Deployment with NGINX Container Mounting the PV***
 ## PVC
 ```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: my-pv-storage
          persistentVolumeClaim:
            claimName: my-pvc
      containers:
        - name: nginx
          image: nginx
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: my-pv-storage
```

**Things to note** : 
- here, persistentVolumeClaim, ClaimName should match with the name of your PVC.

---

## Summary :
- The PV YAML defines a PersistentVolume named "local-pv" with 5Gi storage capacity, using a local directory /mnt/data as storage.
- The PVC YAML creates a PersistentVolumeClaim named "my-pvc" requesting 2Gi of storage and using the "local-storage" StorageClass.
- The Deployment YAML creates a NGINX Deployment with one replica, which mounts the PVC claimed by "my-pvc" to the NGINX container's /usr/share/nginx/html directory.




Now, your Mannual Provisioning should work correctly ! 

---

