# bookmark-infra

This repo manages shared infrastructure resources installed into the cluster using Helm and other Kubernetes tools.

## Helm Packages

### cert-manager

* Purpose: Automatically issues and renews TLS certificates (e.g. via Let's Encrypt)

* Install command:
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

* Issuer configuration: Located in `bookmark-infra/cert-manager/cluster-issuer.yaml`

### ingress-nginx

* Purpose: Acts as the ingress controller, routing external traffic to internal services based on host/path

* Install command:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
```

### PostgreSQL

PostgreSQL is deployed in the bookmark-db namespace using the Bitnami Helm chart.

Installation:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

kubectl create namespace bookmark-db

helm install postgres bitnami/postgresql \
  --namespace bookmark-db \
  --set auth.username=bookmark \
  --set auth.password=bookmarkpass \
  --set auth.database=bookmark
```

Test connection inside the cluster:

```bash
kubectl run pg-client --rm -it --restart=Never --namespace bookmark-db \
  --image=bitnami/postgresql:latest \
  --env="PGPASSWORD=bookmarkpass" \
  --command -- psql -h postgres-postgresql -U bookmark -d bookmark
```

DATABASE_URL example:
```bash
postgresql+asyncpg://bookmark:bookmarkpass@postgres-postgresql.bookmark-db.svc.cluster.local:5432/bookmark
```
