# Kubernetes Training

## Build local register with Volume on Windows
[- Build and run](./local-register-windows)

[- Push image in local registry](./local-register-windows)

## Deployment
### 1) Example with Nginx 
 [- Deployment Ingress with Nginx](./1-example-nginx-ingress)

 [- Test deployment on Docker Windows](./1-example-nginx-ingress)

### 2) Test deployment of my API
  [- Example ClusterIp and port-forward for verification](./2-examples-services#1-example-useing-clusterip-and-port-forward-for-verification)

  [- Example NodePort and Ingress](./2-examples-services)

  [- Example LoadBalancer](./2-examples-services)

  [- Example Headless Service](./2-examples-services)

  [- Exemple service without Selector](./2-examples-services)

  [- Example ExternalIP](./2-examples-services)

  [- Example ExternalName](./2-examples-services)

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