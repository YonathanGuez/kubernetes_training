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

## Test deployment of my API
### 1) Example useing port-forward
Deployment Service :
```
kubectl apply -f  .\deployment.yaml
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
kubectl delete -f  .\deployment.yaml
```
