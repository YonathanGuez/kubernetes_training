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