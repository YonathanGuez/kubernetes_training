# Create a POD simple 

## Template POD :

```
kubectl run nginx -o yaml --image nginx --port 80 --dry-run=client > pod.yaml
```

that will build a file like this :
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Run the Pod file :
```
kubectl apply -f pod.yml 
```

# Deploy pod on the specific worker node

## 1) Add a label to a node
```
kubectl label nodes <your-node-name> disktype=ssd
```
check :
```
kubectl get nodes --show-labels
```
## 2) Create a pod that gets scheduled to your chosen node
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

Verify :
```
kubectl get pods --output=wide
```

```
NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
nginx    1/1       Running   0          13s    10.200.0.4   worker0
```

## Create a pod that gets scheduled to specific node

Schedule pod to specific node: 
ex :nodeName: foo-node 
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: foo-node # schedule pod to specific node
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

### Service
[SERVICE POD](./2-examples-services/README.md)

- Expose the pod Service using ClusterIP
- Expose the pod Service using Nodeport
- Expose the pod Service using loadBalancer
- Headless
- Service without Selectors
- ExternalIP
- ExternalName

#### Ingress
[NGINX with Ingress](./1-example-nginx-ingress/README.md)

Expose the Pod Service using Ingress

### Persistent Volume
[Add Persistent Volume ](./3-volume/README.md)

- Setup Pod resources & limits:
- Setup Pod with startup, liveness, and readiness probes.
- Add Persistent Volume to the pod:
  * Persistent Volume 
  * Persistence Volume Claim
  * Test on Postgres with Statefulset 

### Configmap
Attach configmap to pod

ConfigMap:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  labels:
    app: postgres
data:
  POSTGRES_DB: postgresdb
  POSTGRES_USER: admin
  POSTGRES_PASSWORD: admin123
```

In My POD :
spec.template.spec.containers:
```
  envFrom:
        - configMapRef:
            name: postgres-config
```
So now all the data inside ConfigMap are called (POSTGRES_DB,POSTGRES_USER,POSTGRES_PASSWORD)
you can check the example of [ex-pg-volume-local-windows.yaml](./3-volume/ex-pg-volume-local-windows.yaml)

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  selector:
    matchLabels:
      app: postgres
  replicas: 1
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:10.4
        imagePullPolicy: "IfNotPresent"
        ports:
        - containerPort: 5432
        envFrom:
        - configMapRef:
            name: postgres-config
      .........
```

### Secret
Add Secret to pod