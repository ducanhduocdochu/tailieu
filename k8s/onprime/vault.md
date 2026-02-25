helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

kubectl create namespace vault

helm install vault hashicorp/vault \
  --namespace vault \
  --set "server.dev.enabled=true" \
  --set "ui.enabled=true"

kubectl get pods -n vault

=> set up ingress nginx
=> add host

kubectl logs vault-0 -n vault
