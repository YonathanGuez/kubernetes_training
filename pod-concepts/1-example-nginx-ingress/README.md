
# Example to use Ingress

## create namespace :
```
kubectl create namespace java2days
```

Switch namespace:
```
kubectl get namespace
kubectl config set-context --current --namespace=java2days
```
## Deployment Ingress with Nginx
```
kubectl apply -f .\pod-concepts\1-example-nginx-ingress\deployment-nginx-backend.yaml
kubectl apply -f .\pod-concepts\1-example-nginx-ingress\service-nginx.yaml
kubectl apply -f .\pod-concepts\1-example-nginx-ingress\ingress-nginx.yaml
```
```
kubectl get service 
```
```
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.105.159.229   <none>        80:32648/TCP   16m
```
curl -I http://127.0.0.1:32648/
curl -I IP_COMPUTER:32648

## Delete all deployment service ingress and namespace

```
kubectl delete -f .\pod-concepts\1-example-nginx-ingress\deployment-nginx-backend.yaml
kubectl delete -f .\pod-concepts\1-example-nginx-ingress\service-nginx.yaml
kubectl delete -f .\pod-concepts\1-example-nginx-ingress\ingress-nginx.yaml
```

```
kubectl config set-context --current --namespace=default
```
delete namespace
```
kubectl delete namespaces java2days 
```


### Test deployment Nginx on Docker Windows:

#### 1) We need to do pull nginx for that exist in docker windows:
```
docker pull nginx
```
#### 2) Deployment Nginx and service:
```
kubectl apply -f  .\pod-concepts\1-example-nginx-ingress\deployment-nginx.yaml
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
