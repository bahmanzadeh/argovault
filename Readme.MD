# Install Vault
https://artifacthub.io/packages/helm/hashicorp/vault 
https://www.hashicorp.com/blog/announcing-the-vault-helm-chart 
https://medium.com/@raosharadhi11/argocd-with-vault-using-argocd-vault-plugin-dccbc302f0c2

helm install vault hashicorp/vault -n vault --create-namespace --set server.dev.enables=true 

After that intialize it and unseal it: 

kubectl exec -it vault-0 -n vault -- vault operator init 

Ubseal it with at least 3 keys (replace with your keys, run 3 times) 

kubectl exec -ti vault-0 -n vault -- vault operator unseal <---> 
you should save this somewhere: Initial Root Token: <-->

kubectl port-forward vault-0 8200
export VAULT_ADDR=http://localhost:8200
vault login root 

Now:
First create your secret engine and secret in vault 
Then you create a policy and give permissions on the secret that you just created 

path "kv/data/application" {
  capabilities = [ "read" ]
}


Then you need to use command to create the token and assign it to those policies: 

vault token create -policy=argopolicy

you can test like this:
vault login

vault kv get -mount="kv" "application"


And make sure to create a secret named vault-configuration.yaml so that argocd can access the vault with read access 


# install ArgoCD with provided vaules
# REF: https://medium.com/@raosharadhi11/argocd-with-vault-using-argocd-vault-plugin-dccbc302f0c2
kubectl create ns argocd
kubectl apply -f vault-secret.yaml -n argocd
kubectl apply -f argocd-vault-plugin-cmp.yaml -n argocd
helm repo add argo https://argoproj.github.io/argo-helm
kubectl apply -k "https://github.com/argoproj/argo-cd/manifests/crds?ref=v2.11.0"
helm install argocd argo/argo-cd -n argocd -f argocd-helm-values.yaml --version 6.9.3

# get Argocd password: 
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
0kz8gAKeK2-iun5E
# port forwarding:
kubectl port-forward svc/argocd-server 8080:80
# login to Argcd
http://localhost:8080

Make sure to change the name of the cluster in settings to "k8scluster1" that matches the tenancy mp file
and create the right namespaces 