# Argocd and Vault Integration
## Install Vault
```bash
https://artifacthub.io/packages/helm/hashicorp/vault 
https://www.hashicorp.com/blog/announcing-the-vault-helm-chart 
https://medium.com/@raosharadhi11/argocd-with-vault-using-argocd-vault-plugin-dccbc302f0c2
helm install vault hashicorp/vault -n vault --create-namespace --set server.dev.enables=true 
```
## Initialize it and unseal it: 
```bash
kubectl exec -it vault-0 -n vault -- vault operator init 

## Seal it with at least 3 keys (replace with your keys, run 3 times) 
```bash
kubectl exec -ti vault-0 -n vault -- vault operator unseal <---> 
```
Save the tokens: Initial Root Token: <-->

## Port Forwarding
```bash
kubectl port-forward vault-0 8200
export VAULT_ADDR=http://localhost:8200
vault login root 
```
## Create your secret engine and secret in vault 
Create a policy and give permissions on the secret that you just created 
```bash
path "kv/data/application" {
  capabilities = [ "read" ]
}
```

## Create the token and assign it to those policies: 
```bash
vault token create -policy=argopolicy
vault login
vault kv get -mount="kv" "application"
```
## Create a secret named vault-configuration.yaml so that argocd can access the vault with read access 
 Install ArgoCD with the provided values
 REF: https://medium.com/@raosharadhi11/argocd-with-vault-using-argocd-vault-plugin-dccbc302f0c2
```bash
kubectl create ns argocd
kubectl apply -f vault-secret.yaml -n argocd
kubectl apply -f argocd-vault-plugin-cmp.yaml -n argocd
helm repo add argo https://argoproj.github.io/argo-helm
kubectl apply -k "https://github.com/argoproj/argo-cd/manifests/crds?ref=v2.11.0"
helm install argocd argo/argo-cd -n argocd -f argocd-helm-values.yaml --version 6.9.3
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward svc/argocd-server 8080:80
http://localhost:8080
```
**Make sure to change the name of the cluster in settings to "k8scluster1" that matches the tenancy Map file, and create the right namespaces**
