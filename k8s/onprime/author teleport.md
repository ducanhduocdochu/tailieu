# 1. set up folder role
mkdir -p /tools/teleport/rbac
cd /tools/teleport/rbac
nano administrator.yaml

```
kind: role
version: v7
metadata:
  name: administrator
spec:
  allow:
    logins: ["root"]
    node_labels:
      "*": "*"
    kubernetes_groups: ["system:masters"]
    rules:
      - resources: ["*"]
        verbs: ["*"]
  deny: {}
```

tctl create -f administrator.yaml
tctl users add administrator --role=administrator

nano devops.yaml
```
kind: role
version: v7
metadata:
  name: devops
spec:
  allow:
    logins: ["root"]
    node_labels: {"*": "*"}
    kubernetes_labels: {"*": "*"}
    database_labels: {"*": "*"}
    app_labels: {"*": "*"}
    rules:
      - resources: ["node", "k8s_cluster", "database", "app"]
        verbs: ["*"]
  deny:
    rules:
      - resources: ["role", "auth_server", "trusted_cluster", "token"]
        verbs: ["*"]
```

nano base.yaml
```
kind: role
version: v7
metadata:
  name: base
spec:
  allow:
    logins: ["dev1"]
  deny:
    rules:
      - resources: ["node"]
        verbs: ["*"]
```
