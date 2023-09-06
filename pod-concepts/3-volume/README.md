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

- It is used to indicate if the application inside the Container has started.
- If a startup probe is provided, all other probes are disabled.
- If the request fails, it will restart the container.
- Once the startup probe has succeeded once, the liveness probe takes over to provide a fast response to container deadlocks.
- If not provided the default state is Success.

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
### DIFFRENTE CONFIG FOR PROBES

Startup probes support the four basic Kubernetes probing mechanisms:

- Exec: Executes a command within the container. The probe succeeds if the command exits with a 0 code.

- HTTP: Makes an HTTP call to a URL within the container. The probe succeeds if the container issues an HTTP response in the 200-399 range.

- TCP: The probe succeeds if a specific container port is accepting traffic.

- gRPC: Makes a gRPC health checking request to a port inside the container and uses its result to determine whether the probe succeeded.

### Summary

Startup Probes: Used to check if the application inside the Container has started

Liveness Probes: Used to check if the container is available and alive.

Readiness Probes: Used to check if the application is ready to use and serve the traffic

## Add Persistent Volume to the pod.

Build a percistance volume is useful for save Configuration and Data of your DB 
For that you need to tools : Persistent Volume ant Persistence Volume Claime

### Persistent Volume:
That will tell you Pod where to stock all you data and what is the size 

In this example we use storageClassName: hostpath (it s our local node storage)
You can check that with : 
```kubectl get storageclass
```

So we put local node storage + 1Go + call "/run/desktop/mnt/host/c/tmp/" for c:/tmp in windows:
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-volume
  labels:
    type: local
spec:
  storageClassName: hostpath
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    # deployment volume on local machine windows : c:/tmp (youn need to have tmp available and empty in your local)
    path: "/run/desktop/mnt/host/c/tmp/"
```
### Persistence Volume Claim:

PersistentVolumeClaim will be referenced in a pod. 
A PersistentVolumeClaim is created by specifying the minimum size and the access mode they require from the persistentVolume.
Is same like in this example how to get 50mi from 1Gi:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-claim
spec:
  storageClassName: hostpath
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```
### Test on Postgres with Statefulset 

A StatefulSet does three big things differently from a Deployment:
1- It creates a new PersistentVolumeClaim for each replica;
2- It gives the pods sequential names, starting with statefulsetname-0.
3- It starts the pods in a specific order (ascending numerically).
This is useful when the database itself knows how to replicate data between different copies of itself.
StatefulSet it will make sure to terminate the existing Pod before creating a new, so that there are never more than 1 (when you have 1 as desired number of replicas).

Run Test :
```
kubectl apply -f .\ex-pg-volume-local-windows.yaml
```

Check deployments :
```
kubectl get all
```

Result :
```
NAME             READY   STATUS    RESTARTS        AGE
pod/nginx        1/1     Running   12 (113m ago)   135d
pod/postgres-0   1/1     Running   0               8s
pod/utils        1/1     Running   29 (113m ago)   184d

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    231d
service/postgres     ClusterIP   10.108.123.30   <none>        5432/TCP   8s

NAME                        READY   AGE
statefulset.apps/postgres   1/1     8s
```

Check logs for postgres-0:
```
kubectl logs postgres-0
```

Return :
```
2023-09-06 14:07:58.502 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2023-09-06 14:07:58.502 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2023-09-06 14:07:58.516 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-09-06 14:07:58.586 UTC [24] LOG:  database system was shut down at 2023-09-06 13:42:51 UTC
2023-09-06 14:07:58.617 UTC [1] LOG:  database system is ready to accept connections
```