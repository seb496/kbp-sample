# Kubernetes best practices

## Cluster setup

```shell
kind delete cluster -n kbp

kind create cluster --config kbp.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml --context kind-kbp

kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s --context kind-kbp
```

## Sample app

Nginx fileserver for static files, nodejs frontend server, redis database.

```shell
kubectl create secret generic redis-passwd --from-literal=passwd=$RANDOM$RANDOM
kubectl create secret generic redis-passwd --from-literal=passwd=$(get-random)
kubectl create configmap frontend-config-v1 --from-literal=journalEntries=10
kubectl apply -f ingress.yaml
kubectl apply -f frontend-with-secret.yaml
kubectl apply -f frontend-service.yaml
kubectl create configmap redis-config --from-file=./launch.sh
kubectl apply -f redis.yaml
kubectl apply -f redis-svc.yaml
kubectl apply -f redis-write-svc.yaml

podman build -t fileserver:v2 fileserver
#podman run --rm --name fileserver -d -p 81:80 fileserver:v1
TempFile=$(mktemp)
$TempFile = New-TemporaryFile
podman save fileserver:v2 > $TempFile
kind load image-archive --name kbp $TempFile
rm $TempFile
remove-item $TempFile

kubectl apply -f fileserver.yaml
kubectl apply -f fileserver-svc.yaml

curl kbp.localdev.me:8080/api
curl kbp.localdev.me:8080/

```

## Monitoring

Deploy default prom stack to <http://grafana.localdev.me:8080/> username admin password prom-operator

```shell
kubectl create ns monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install --namespace monitoring prometheus prometheus-community/kube-prometheus-stack
kubectl create ingress -n monitoring grafana --rule grafana.localdev.me/=prometheus-grafana:80 
```
