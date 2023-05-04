# Kubernetes Training

 # RoadMap Kubernetes :
 ## Learn Kubernetes Architecture
 ### Control plane components
 ### Worker node components:
 ### Addon Components
 ### Cluster high availability
 ### Network Design

 ## Kubeconfig File Explained With Practical Examples:
 [link here](./kubeconfig.md) 
 ### Different Methods to Connect Kubernetes Cluster With Kubeconfig File
 ### Merging Multiple Kubeconfig Files
 ### How to Generate Kubeconfig File


## Pod concepts:
  [Deploy a pod](./pod-concepts/README.md)
  Deploy pod on the specific worker node

### Service
[SERVICE POD](./pod-concepts/2-examples-services/README.md)

[- Example ClusterIp and port-forward for verification](./pod-concepts/2-examples-services#1-example-useing-clusterip-and-port-forward-for-verification)

[- Example NodePort and Ingress](./pod-concepts/2-examples-services)

[- Example LoadBalancer](./pod-concepts/2-examples-services)

[- Example Headless Service](./pod-concepts/2-examples-services)

[- Exemple service without Selector](./pod-concepts/2-examples-services)

[- Example ExternalIP](./pod-concepts/2-examples-services)

[- Example ExternalName](./pod-concepts/2-examples-services)

#### Ingress
Expose the Pod Service using Ingress
[- Deployment Ingress with Nginx](./pod-concepts/1-example-nginx-ingress/README.md)

[- Test deployment on Docker Windows](./pod-concepts/1-example-nginx-ingress/README.md)

### Persistent Volume
- Setup Pod resources & limits
- Setup Pod with startup, liveness, and readiness probes.
- Add Persistent Volume to the pod.

### Configmap
- Attach configmap to pod

### Secret
- Add Secret to pod

## Multi container :
multi-container pods (sidecar container pattern)
Init containers
Ephemeral containers
Static Pods
Learn to troubleshoot Pods
Pod Preemption & Priority
Pod Disruption Budget
Pod Placement Using a Node Selector
Pod Affinity and Anti-affinity

## Learn Pod Dependent Objects
Replicaset
Deployment
Daemonsets
Statefulset
Jobs & Cronjobs

## Learn Ingress & Ingress Controllers
Kubernetes Ingress Explained
Setting up Nginx Ingress Controller

## Learn End to End Microservices Application Deployment on Kubernetes
Build Docker images for all the services. Ensure you optimize the Dockerfile to reduce the Docker Image size.
Create manifests for all the services. (Deployment, Statefulset, Services, Configmaps, Secrets, etc)
Expose the front end with service type ClusterIp
Deploy Nginx Ingress controller and expose it with service type Loadbalancer
Map the loadbalancer IP to the domain name.
Create an ingress object with a DNS name with the backend as a front-end service name.
Validate the application.

## Learn About Securing Kubernetes Cluster
Service account
Pod Security Context
Seccomp & AppArmor
Role Based Access Control (RBAC)
Attribute-based access control (ABAC) 
Network Policies

The following are the open-source tools you need to look at: Open Policy Agent ,Kyverno ,Kube-bench,Kube-hunter ,Falco

## Learn About Kubernetes Configuration Management Tools
Helm (Templating Engine)
Kuztomize (Overlay Engine)

## Learn About Kubernetes Operator Pattern
Custom resource definitions
Admission controllers
Validating & Mutating Webhooks

## Learn Important Kubernetes Configurations
Custom DNS server
Custom image registry
Shipping logs to external logging systems
Kubernetes OpenID Connect
Segregating & securing Nodes for PCI & PII Workloads


////////////////////////////////////////////////////
## Build local register with Volume on Windows
[- Build and run](./local-register-windows)

[- Push image in local registry](./local-register-windows)

## Deployment

### Static pod 

https://techdirectarchive.com/2021/09/09/how-to-create-a-static-pod-in-kubernetes-with-demos-that-can-help-you-become-a-better-kubernetes-administrator/

static pods are pods the kubelet have total control over:
- created and managed by kubelet daemon 
- without API server
- Control plane is not involved in lifecycle 

Static pods are usually used by software bootstrapping kubernetes itself.

#### Process to build 

1) First change to the directory tracked by kubelet
```
cd /etc/kubernetes/manifests
```
why ? because you get /etc/kubernetes/manifests when: 
```
cat /var/lib/kublet/config.yaml | grep staticPod
```

2) create the definition/manifest in that director
```
k run some-pod --image=python --command sleep 2017 --restart=Never --dry-run=client -o yaml > statuc-pod.yaml
```
remove it 
```
Go to that directory and remove the manifest/definition of the staic Pod (rm <STATIC_POD_PATH>/<POD_DEFINITION_FILE>)
```

### TIPS :

build pod yaml fast :
```
kubectl run some-pod -o yaml --image nginx-unprivileged --dry-run=client > pod.yaml
```

Test yaml : --dry-run=client
```
kubectl create -f .\pod.yaml --dry-run=client
```

if tow container in same pod we need to specify the container 
```
kubectl logs POD_NAME -c CONTAINER_NAME
```
 ### Get Metrics Kubernetes

 ```kubectl top node
 ```

 ```kubectl top pod -A --containers=true
 ```

 ### fake ssh docker-desktop
 ```
 docker run -it --rm --privileged --pid=host justincormack/nsenter1                                                   
 ```
