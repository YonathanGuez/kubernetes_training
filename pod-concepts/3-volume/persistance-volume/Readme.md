# persitance volume

Use PVC

```
kubectl get storageclass
NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
hostpath (default)   docker.io/hostpath   Delete          Immediate
    false                  313d

kubectl create namespace postgres
kubectl apply -f .\pv.yaml

kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLexample-volume   1Gi        RWO            Retain           Available           hostpath
               37s

kubectl  -n postgres apply -f pvc.yaml
persistentvolumeclaim/example-claim created


kubectl  -n postgres get pvc
NAME            STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
example-claim   Bound    example-volume   1Gi        RWO            hostpath       48s

kubectl  -n postgres apply -f pg-with-pv.yaml
configmap/postgres-config created
statefulset.apps/postgres created
service/postgres created

kubectl  -n postgres get pods
NAME         READY   STATUS              RESTARTS   AGE
postgres-0   0/1     ContainerCreating   0          30s

kubectl  -n postgres get pods
NAME         READY   STATUS    RESTARTS   AGE
postgres-0   1/1     Running   0          77s


kubectl  -n postgres describe pod postgres-0
Name:             postgres-0
Namespace:        postgres
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.4
Start Time:       Mon, 04 Sep 2023 21:52:37 +0300
Labels:           app=postgres
                  controller-revision-hash=postgres-75bb665555
                  statefulset.kubernetes.io/pod-name=postgres-0
Annotations:      <none>
Status:           Running
IP:               10.1.0.54
IPs:
  IP:           10.1.0.54
Controlled By:  StatefulSet/postgres
Containers:
  postgres:
    Container ID:   docker://c40deb91ca9835317abfda385c14cef8c1689ec192999ba3a78e582d3e670338
    Image:          postgres:10.4
    Image ID:       docker-pullable://postgres@sha256:9625c2fb34986a49cbf2f5aa225d8eb07346f89f7312f7c0ea19d82c3829fdaa
    Port:           5432/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 04 Sep 2023 21:53:18 +0300
    Ready:          True
    Restart Count:  0
    Environment Variables from:
      postgres-config  ConfigMap  Optional: false
    Environment:       <none>
    Mounts:
      /var/lib/postgresql/data from data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j4hk8 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  example-claim
    ReadOnly:   false
  kube-api-access-j4hk8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m21s  default-scheduler  Successfully assigned postgres/postgres-0
to docker-desktop
  Normal  Pulling    2m21s  kubelet            Pulling image "postgres:10.4"
  Normal  Pulled     105s   kubelet            Successfully pulled image "postgres:10.4"
in 35.3387459s
  Normal  Created    102s   kubelet            Created container postgres
  Normal  Started    101s   kubelet            Started container postgres
PS C:\Users\YONATHAN\Desktop\projects\kubernetes\kubernetes_training\pod-concepts\3-volume\persistance-volume>
```
