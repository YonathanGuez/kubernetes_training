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

### liveness
The kubelet uses liveness probes to know if an application running as in a healthy state and if unhealthy state then restart a container and tries to redeploy it . 
For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. 

```
kubectl apply -f ex-livesse.yaml
```

check before 30 secondes :
```
kubectl describe pod liveness-exec
```
we will see that no liveness probes have failed yet.
```
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  5s    default-scheduler  Successfully assigned default/liveness-exec to docker-desktop
  Normal  Pulling    4s    kubelet            Pulling image "localhost:5000/busybox"
  Normal  Pulled     4s    kubelet            Successfully pulled image "localhost:5000/busybox" in 72.9112ms
  Normal  Created    4s    kubelet            Created container liveness
  Normal  Started    4s    kubelet            Started container liveness
```

And after 30 seconde :
```
kubectl describe pod liveness-exec
```
we will see Reason: Unhealthy -> Killing --> Pulling --> Created --> Started : 
```
 Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  89s                default-scheduler  Successfully assigned default/liveness-exec to docker-desktop
  Normal   Pulled     88s                kubelet            Successfully pulled image "localhost:5000/busybox" in 72.9112ms
  Warning  Unhealthy  44s (x3 over 54s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    44s                kubelet            Container liveness failed liveness probe, will be restarted
  Normal   Pulling    14s (x2 over 88s)  kubelet            Pulling image "localhost:5000/busybox"
  Normal   Created    14s (x2 over 88s)  kubelet            Created container liveness
  Normal   Started    14s (x2 over 88s)  kubelet            Started container liveness
```
### readiness
The kubelet uses readiness probes to know when a container is ready to start accepting traffic.
A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

```
kubectl apply -f ex-readiness.yaml
```

check before 30 secondes :
```
kubectl describe pod readiness-exec
```
we will see that no liveness probes have failed yet.
```
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  23s   default-scheduler  Successfully assigned default/readiness-exec to docker-desktop
  Normal  Pulling    23s   kubelet            Pulling image "localhost:5000/busybox"
  Normal  Pulled     23s   kubelet            Successfully pulled image "localhost:5000/busybox" in 91.0959ms
  Normal  Created    22s   kubelet            Created container readiness
  Normal  Started    22s   kubelet            Started container readiness
```

And after 30 seconde :
```
kubectl describe pod readiness-exec
```
we will see Reason: Unhealthy -> Killing --> Pulling --> Created --> Started : 
```
 Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  52s               default-scheduler  Successfully assigned default/readiness-exec to docker-desktop
  Normal   Pulling    51s               kubelet            Pulling image "localhost:5000/busybox"
  Normal   Pulled     51s               kubelet            Successfully pulled image "localhost:5000/busybox" in 91.0959ms
  Normal   Created    50s               kubelet            Created container readiness
  Normal   Started    50s               kubelet            Started container readiness
  Warning  Unhealthy  2s (x5 over 17s)  kubelet            Readiness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

Let’s check the status directly:
```
kubectl get pods readiness-exec -o jsonpath='{.status.containerStatuses[0].state}'             
```
result ex:
{"running":{"startedAt":"2023-06-21T11:27:50Z"}}

Let’s get the ready status:
```
kubectl get pods readiness-exec -o jsonpath='{.status.containerStatuses[0].ready}'
```
result : false
so the container is not ready

delete deployment:
```
kubectl delete -f .\ex-readiness.yaml
```

### startup
The kubelet uses startup probes to know when a container application has started.
If such a probe is configured, it disables liveness and readiness checks until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running.

```
kubectl apply -f ex-startup.yaml
```

check before 300 secondes :
```
kubectl describe pod startup-exec
```
we will see that startup probes not have failed yet.
```
    Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  35s   default-scheduler  Successfully assigned default/startup-exec to docker-desktop
  Normal  Pulling    34s   kubelet            Pulling image "localhost:5000/busybox"
  Normal  Pulled     34s   kubelet            Successfully pulled image "localhost:5000/busybox" in 88.7345ms
```

And after 300 seconde :
```
kubectl describe pod startup-exec
```
we will see Reason: Unhealthy -> Killing --> Pulling --> Created --> Started : 
```
   Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  3m43s                default-scheduler  Successfully assigned default/startup-exec to docker-desktop
  Normal   Pulling    3m43s                kubelet            Pulling image "localhost:5000/busybox"
  Normal   Pulled     2m2s                 kubelet            Successfully pulled image "localhost:5000/busybox" in 1m40.6246236s
  Normal   Created    2m2s                 kubelet            Created container startup
  Normal   Started    2m2s                 kubelet            Started container startup
  Warning  Unhealthy  24s (x10 over 114s)  kubelet            Startup probe failed: cat: can't open '/etc/foobar': No such file or directory
  Normal   Killing    24s                  kubelet            Container startup failed startup probe, will be restarted

```

Let’s check the status directly:
```
kubectl get pods startup-exec -o jsonpath='{.status.containerStatuses[0].state}'             
```
result ex:
{"running":{"startedAt":"2023-06-21T11:27:50Z"}}

Let’s get the ready status:
```
kubectl get pods startup-exec -o jsonpath='{.status.containerStatuses[0].ready}'
```
result : false
so the container is not ready

delete deployment:
```
kubectl delete -f .\ex-startup.yaml
```

## Add Persistent Volume to the pod.