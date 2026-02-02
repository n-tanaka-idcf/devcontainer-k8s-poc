# devcontainer-k8s-poc

```console
cd kubernetes;
kind create cluster --config kind-config.yaml;
docker network connect kind $HOSTNAME;
kind get kubeconfig --internal > ~/.kube/config"
```
