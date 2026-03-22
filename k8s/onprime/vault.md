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


setup external
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets \
  -n external-secrets \
  --create-namespace

kubectl exec -it vault-0 -n vault -- sh
vault auth enable kubernetes

vault write auth/kubernetes/config \
  token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

vault policy write eso-policy - <<EOF
path "secret/data/auth" {
  capabilities = ["read"]
}
EOF

vault write auth/kubernetes/role/eso-role \
  bound_service_account_names=external-secrets \
  bound_service_account_namespaces=external-secrets \
  policies=eso-policy \
  ttl=24h

```
apiVersion: external-secrets.io/v1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: backend
spec:
  provider:
    vault:
      server: "http://vault.vault.svc.cluster.local:8200"
      path: "kv"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "backend-auth-role"
          serviceAccountRef:
            name: "vault-auth"
            audiences:
              - vault
```

```
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: backend-auth-secret
  namespace: backend
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: backend-auth-secret
    creationPolicy: Owner
  data:
    - secretKey: JWT_SECRET
      remoteRef:
        key: backend/auth
        property: JWT_SECRET
    - secretKey: URL_CONNECT_DB
      remoteRef:
        key: backend/auth
        property: URL_CONNECT_DB
```
