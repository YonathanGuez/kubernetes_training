
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
kubectl apply -f .\example-nginx-ingress\deployment-nginx-backend.yaml
kubectl apply -f .\example-nginx-ingress\service-nginx.yaml
kubectl apply -f .\example-nginx-ingress\ingress-nginx.yaml
```
```
kubectl get service 
```
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.105.159.229   <none>        80:32648/TCP   16m

curl -I http://127.0.0.1:32648/
curl -I IP_COMPUTER:32648

## Delete all deployment service ingress and namespace

```
kubectl delete -f .\example-nginx-ingress\deployment-nginx-backend.yaml
kubectl delete -f .\example-nginx-ingress\service-nginx.yaml
kubectl delete -f .\example-nginx-ingress\ingress-nginx.yaml
```

```
kubectl config set-context --current --namespace=default
```
delete namespace
```
kubectl delete namespaces java2days 
```