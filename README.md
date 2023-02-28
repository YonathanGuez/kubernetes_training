# Kubernetes Training

## Build DOCKER IMAGE 

Build my image from Dockerfile:
```
docker build -t test .
```

## Build local register with Volume from Windows

### Build and run :
```
docker run -d -p 5000:5000 --name registry -v C:\Users\yaniv\Documents\registry:/var/lib/registry --restart always registry:2
```

### push image in local registry :
```
docker tag test:latest localhost:5000/test:latest
docker push localhost:5000/test:latest
```

## Deployment

### Test deployment Nginx on Docker Windows:

#### 1) We need to do pull nginx for that exist in docker windows:
```
docker pull nginx
```
#### 2) Deployment Nginx and service:
```
kubectl apply -f  .\deployment-nginx.yaml
```

get pod :
```
kubectl get pod
```

enter in the pod :
```
kubectl exec -it test-deploy-979784999-cxtrh -- /bin/sh
```
#### 3) For call nginx outside we need port-forward:
```
kubectl port-forward service/test-nginx 8080:80
```

#### 4) Now we can call the web site and get 'Welcome to nginx!':
```
http://127.0.0.1:8080/
```

## Test deployment of my API
### 0) Example useing port-forward
Deployment Service :
```
kubectl apply -f  .\0-deployment.yaml
```
Get deployment:
```
kubectl get deployments test-deploy
```
Get detail :
```
kubectl describe deployments test-deploy
```
Get Replicate set :
```
kubectl get replicasets
```
Create a Service object that exposes the deployment:
```
kubectl expose deployment test-deploy --type=LoadBalancer --name=my-service
```
```
kubectl get services my-service     
```
```
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
my-service   LoadBalancer   10.96.118.73   localhost     3000:31129/TCP   5m49s
```
Use the external IP address (LoadBalancer Ingress) to access to the app:
```
curl http://<external-ip>:<port>
```
ex : curl localhost:3000

delete all :
```
kubectl delete services my-service
kubectl delete -f  .\0-deployment.yaml
```
### 1) Example useing ClusterIp and port-forward for verification
This IP is internal and won’t work outside.
it s usefull for internal connection 

Deployment Service :
```
kubectl apply -f  .\1-deployment-service-clusterip.yaml
```

Get Service IP and PORT :
```
kubectl get services
```

Listen on port 8080 on all addresses, forwarding to 3000 in the pod:
```
kubectl port-forward --address 0.0.0.0 service/test-node 8080:3000
```
Check the link and get 'Hi there':
```
http://127.0.0.1:8080/
```
or ip of the computer:
```
http://IP_COMPUTER:8080/
```

Remove all deployment in 1-deployment-service-clusterip.yaml
```
kubectl delete -f  1-deployment-service-clusterip.yaml
```

### 2) Example with NodePort and Ingress:

When a NodePort is created, kube-proxy exposes a port in the range 30000-32767:
The problem with using a NodePort is that you still need to access each of the Nodes separately.
```
kubectl apply -f .\2-deployment-service-nodeport-ingress.yaml
```
```
kubectl get service
```
```
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP          36d
test-node-service   NodePort    10.99.122.245   <none>        3000:31751/TCP   9s
```
Check chrome :
```
http://127.0.0.1:31751/
```

Delete:
```
kubectl delete -f .\2-deployment-service-nodeport-ingress.yaml
```

### 3) Example with LoadBalancer:
A LoadBalancer is a Kubernetes service that:

Creates a service like ClusterIP
Opens a port in every node like NodePort
Uses a LoadBalancer implementation from your cloud provider (your cloud provider needs to support this for LoadBalancers to work).
The main difference with NodePort is that LoadBalancer can be accessed and will try to equally assign requests to Nodes.

```
kubectl apply -f .\3-deployment-loadbalancer.yaml
```
```
kubectl get service example-service
```
```
NAME              TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
example-service   LoadBalancer   10.100.32.92   localhost     3000:30118/TCP   36s
```

Check chrome :
```
http://localhost:3000/
```

Delete:
```
kubectl delete -f .\3-deployment-loadbalancer.yaml
```

### 4) Example Headless Service
A headless service is a service with a service IP but instead of load-balancing it will return the IPs of our associated Pods. 
This allows us to interact directly with the Pods instead of a proxy. 
It's as simple as specifying clusterIP: None and can be utilized with or without selectors.

#### Example:
we will run 4 replicats with time:
```
kubectl apply -f .\4-deployment-headless.yaml 
```

check if it's running with port-forward because i not expose IP :
```
kubectl port-forward --address 0.0.0.0 service/example-service 8080:3000
curl http://127.0.0.1:8080/
```

Now what it is doing really :
for that we will use the image dockerbogo/utils:
```
docker pull dockerbogo/utils
kubectl run -it -t utils --image dockerbogo/utils bash
```
this image help us to check what append in the pod :
inside the pod utils root@utils:/#
```
nslookup example-service
```
I will get :
```
root@utils:/# nslookup example-service
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   example-service.default.svc.cluster.local
Address: 10.1.0.132
Name:   example-service.default.svc.cluster.local
Address: 10.1.0.136
Name:   example-service.default.svc.cluster.local
Address: 10.1.0.135
Name:   example-service.default.svc.cluster.local
Address: 10.1.0.137
```
So my service can listen my for pods without to specify IPs

Delete All :
```
kubectl delete -f .\4-deployment-headless.yaml  
kubectl delete pod utils
```

Example utilisation of headless: scraping replicas for prometheus: 

https://medium.com/data-reply-it-datatech/using-a-headless-service-to-expose-replicas-for-prometheus-scraping-543194594e0

### 5) Example ExternalIP


### 6) Example with ExternalName
The ExternalName service was introduced due to the need of connecting to an element outside of the Kubernetes cluster. Think of it not as a way to connect to an item within your cluster, but as a connector to an external element of the cluster.

This serves two purposes:

It creates a single endpoint for all communications to that element.
In case that external service needs to be replaced, it’s easier to switch by just modifying the ExternalName, instead of all connections.

### Explanation

https://sysdig.com/blog/kubernetes-services-clusterip-nodeport-loadbalancer/

#### Service:
A service is an abstract mechanism for exposing pods on a network. Each service is assigned a type—either ClusterIP, NodePort, or LoadBalancer. These define how external traffic reaches the service.

ClusterIP:
ClusterIP is the default Kubernetes service. Your service will be exposed on a ClusterIP unless you manually define another type.

NodePort:
A NodePort publicly exposes a service on a fixed port number. It lets you access the service from outside your cluster. You’ll need to use the cluster’s IP address and the NodePort number—e.g. 123.123.123.123:30000.
Creating a NodePort will open that port on every node in your cluster. Kubernetes will automatically route port traffic to the service it’s linked to.

Ingress:
An Ingress is actually a completely different resource to a Service. You normally use Ingresses in front of your Services to provide HTTP routing configuration. They let you set up external URLs, domain-based virtual hosts, SSL, and load balancing.
You create Ingresses using the Ingress resource type. The kubernetes.io/ingress.class annotation lets you indicate which kind of Ingress you’re creating. This is useful if you’re running multiple cluster controllers.
You can configure SSL by setting the tls field in your Ingress’ spec

Load Balancers
A final service type is LoadBalancer. These services automatically integrate with the load balancers provided by public cloud environments. You’ll need to set up your own load balancer if you’re self-hosting your cluster.
Load balancers are used to map external IP addresses to services in your cluster. Unlike Ingresses, there’s no automatic filtering or routing. Traffic to the external IP and port will be sent straight to your service. This means that they’re suitable for all traffic types.

#### Cluster access:

To access something inside the cluster, there ae a couple of different options available to,

Above 1 and 2 approach will require to write config file:
1 - Cluster IP service with Ingress-Nginx
2 - NodePort Service to expose the pod directly to the outside world.

Dont need config file:
3 - Port Forward: We can run a command at our terminal that tells our kubernets cluster to port-forward a port off a very specific pod inside of our cluster when we use this port forwarding thing that's going to cause our cluster to essentially behaves as though it has a node port service running inside it. It's going to expose this pod or a very specific port on it to the outside world and allow us to connect to it directly from our local machine.