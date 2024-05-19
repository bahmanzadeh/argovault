# install ArgoCD with provided vaules
# REF: https://medium.com/@raosharadhi11/argocd-with-vault-using-argocd-vault-plugin-dccbc302f0c2
kubectl create ns argocd
kubectl apply -f vault-secret.yaml -n argocd
kubectl apply -f argocd-vault-plugin-cmp.yaml -n argocd
helm repo add argo https://argoproj.github.io/argo-helm
kubectl apply -k "https://github.com/argoproj/argo-cd/manifests/crds?ref=v2.4.9"
helm install argocd argo/argo-cd -n argocd -f argocd-helm-values.yaml

# get Argocd password: 
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
scFdHhChFhQh9vZW
# port forwarding:
kubectl port-forward svc/argocd-server 8080:80
# login to Argcd
http://localhost:8080