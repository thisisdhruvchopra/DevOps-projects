# Ephimeral Storage

We can use ephimeral storage like emptyDir to share a common storage between containers of the same pod to store and access temporary data during the lifecycle of the pod.

Let's see how can we use a shared volume in a multi container pod.


## ***pods.yaml***

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: container1
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /data1
      
    - name: container2
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /data2
      
    - name: container3
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /data3
      
  volumes:
  - name: shared-data
    emptyDir: {}
```

Here as you can see we have 3 containers in a single pod, all using different mount paths but using the same 'shared-data'.
ephimeral volume.

---
```yaml
  volumes:
  - name: shared-data
    emptyDir: {}
```

This part of the yaml tells us :
- name of the volume, name: shared-data 
- type of the volume: 'emptyDir: {}'
- Had this been any other type of volume like Persistent volume, we would have mentioned that here. We will explore this in the next tutorial.
