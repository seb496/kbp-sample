# Kubernetes best practices

```shell
podman stop -t 60 kbp-worker kbp2-worker kbp-control-plane kbp2-control-plane
podman start kbp-worker kbp2-worker kbp-control-plane kbp2-control-plane
```

## Cluster setup

```shell
kind delete cluster -n kbp2

kind create cluster --name kbp2 --config kbp2.yaml
helm upgrade --install --namespace ingress-nginx --create-namespace ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --set controller.service.type=NodePort --set controller.hostPort.enabled=true --kube-context kind-kbp2

kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s --context kind-kbp2
```

## Sample app

Nginx fileserver for static files, nodejs frontend server, redis database.

```shell
podman build -t fileserver:v2 fileserver
#podman run --rm --name fileserver -d -p 81:80 fileserver:v1
TempFile=$(mktemp)
$TempFile = New-TemporaryFile
podman save fileserver:v2 > $TempFile
kind load image-archive --name kbp2 $TempFile
rm $TempFile
remove-item $TempFile

helm upgrade --install journal ./journal
```
