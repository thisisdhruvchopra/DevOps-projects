In this tutorial, we will explore Kubernetes storage solutions, which are essential for managing persistent data in containerized applications. Kubernetes provides a powerful and flexible framework for handling storage, ensuring that your applications can store and access data even if pods are rescheduled or containers are restarted. We’ll cover the key storage concepts in Kubernetes, such as Persistent Volumes (PVs), Persistent Volume Claims (PVCs), and Storage Classes, as well as how to configure and use them effectively. By the end of this tutorial, you will have a solid understanding of how to manage and utilize storage in your Kubernetes clusters to support both stateless and stateful applications.

There are two types of K8s Storage

- Ephimeral
- Persistent (most used)

## Ephimeral Storage

- Life Cycle of this kind of storage is the life cycle of a pod.
- As pod gets deleted, so does this storage.
- UseCase- storing temp files in a shared volume betweeen container of the same pod.

## Persistent Storage

This Storage does not get deleted even if the pods are deleted.

Key points to note in the YAML:

- Access Modes : RWO (read write once, one pod RW at once), RO (read only), RWM (read write many, Many can do that together)
- PersistantVolumeReclaimPolicy : retain (means the PVC must retain even after being deleted)
- StorageClassName - It is like the driver name, make it 'manual' for local or Cloud storage name like EBS.

---

Let's discuss about these in greater detail from the next tutorials!
