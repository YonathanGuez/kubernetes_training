 ## Kubeconfig File Explained With Practical Examples
 A Kubeconfig is a YAML file with all the Kubernetes cluster details, certificate, and secret token to authenticate the cluster : $HOME/.kube/config

 ### Different Methods to Connect Kubernetes Cluster With Kubeconfig File
 #### Method 1: Connect to Kubernetes Cluster With Kubeconfig Kubectl Context
1)
 ```mv /path/to/kubeconfig ~/.kube
 ```
2)  List all cluster contexts: ```kubectl config get-contexts ```
3) Set the current context: ``` kubectl config use-context <cluster-name>  ```
4) Validate the Kubernetes cluster connectivity: ``` kubectl get nodes```
#### Method 2: Connect with KUBECONFIG environment variable
```
KUBECONFIG=$HOME/.kube/dev_cluster_config 
```
#### Method 3: Using Kubeconfig File With Kubectl
```
kubectl get nodes --kubeconfig=$HOME/.kube/dev_cluster_config
```
### Merging Multiple Kubeconfig Files
```
KUBECONFIG=config:dev_config:test_config kubectl config view --merge --flatten > config.new
mv $HOME/.kube/config $HOME/.kube/config.old
mv $HOME/.kube/config.new $HOME/.kube/config
kubectl config get-contexts
```
### How to Generate Kubeconfig File
Step 1: Create a Service Account : kubectl -n kube-system create serviceaccount devops-cluster-admin
Step 2: Create a Secret Object for the Service Account : 
```
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: devops-cluster-admin-secret
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: devops-cluster-admin
type: kubernetes.io/service-account-token
EOF
```
Step 3: Create a ClusterRole: 
```
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: devops-cluster-admin
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
EOF
```
Step 4: Create ClusterRoleBinding:
```
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: devops-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: devops-cluster-admin
subjects:
- kind: ServiceAccount
  name: devops-cluster-admin
  namespace: kube-system
EOF
```
Step 5: Get all Cluster Details & Secrets
```
export SA_SECRET_TOKEN=$(kubectl -n kube-system get secret/devops-cluster-admin-secret -o=go-template='{{.data.token}}' | base64 --decode)

export CLUSTER_NAME=$(kubectl config current-context)

export CURRENT_CLUSTER=$(kubectl config view --raw -o=go-template='{{range .contexts}}{{if eq .name "'''${CLUSTER_NAME}'''"}}{{ index .context "cluster" }}{{end}}{{end}}')

export CLUSTER_CA_CERT=$(kubectl config view --raw -o=go-template='{{range .clusters}}{{if eq .name "'''${CURRENT_CLUSTER}'''"}}"{{with index .cluster "certificate-authority-data" }}{{.}}{{end}}"{{ end }}{{ end }}')

export CLUSTER_ENDPOINT=$(kubectl config view --raw -o=go-template='{{range .clusters}}{{if eq .name "'''${CURRENT_CLUSTER}'''"}}{{ .cluster.server }}{{end}}{{ end }}')
```
Step 6: Generate the Kubeconfig With the variables.
```
cat << EOF > devops-cluster-admin-config
apiVersion: v1
kind: Config
current-context: ${CLUSTER_NAME}
contexts:
- name: ${CLUSTER_NAME}
  context:
    cluster: ${CLUSTER_NAME}
    user: devops-cluster-admin
clusters:
- name: ${CLUSTER_NAME}
  cluster:
    certificate-authority-data: ${CLUSTER_CA_CERT}
    server: ${CLUSTER_ENDPOINT}
users:
- name: devops-cluster-admin
  user:
    token: ${SA_SECRET_TOKEN}
EOF
```

Step 7: Validate the generated Kubeconfig
```
kubectl get nodes --kubeconfig=devops-cluster-admin-config 
```
