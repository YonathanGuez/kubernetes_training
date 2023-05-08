# Persistent Volume

## Setup Pod resources & limits

### Creating a CPU Limit:
```
apiVersion: v1
kind: Pod
metadata:
  name: test-deploy
  namespace: demo
spec:
  containers:
    - name: my-container
      image: localhost:5000/test
      resources:
        limits:
          cpu: "0.5"
```
Deployment:
```
kubectl apply -f .\pod-concepts\3-volume\deployment-limite-cpu.yaml.yaml
```

Check CPU :
```
kubectl describe deployment test-deploy
```

### Creating a Memory Limit

The container will be limited to 512Mi of RAM. Kubernetes will still allow it to access more if the node it’s scheduled on has excess capacity. Otherwise, exceeding the limit will result in the container being marked as a candidate for termination.

```
apiVersion: v1
kind: Pod
metadata:
  name: test-deploy
  namespace: demo
spec:
  containers:
    - name: my-container
      image: localhost:5000/test
      resources:
        limits:
          memory: "512Mi"
```
### Storage Limits

All Kubernetes nodes have an amount of ephemeral storage available. 
This container would now be limited to using 1Gi of the available ephemeral storage. Pods that try to use more storage will be evicted. When there are multiple containers in a pod, the pod is evicted if the sum of the storage usage from all of the containers exceeds the overall storage limit.

```
apiVersion: v1
kind: Pod
metadata:
  name: test-deploy
  namespace: demo
spec:
  containers:
    - name: my-container
      image: localhost:5000/test
      resources:
        limits:
          ephemeral-storage: "1Gi"
```

## Setup Pod with startup, liveness, and readiness probes.


## Add Persistent Volume to the pod.