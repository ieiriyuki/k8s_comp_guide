# k8s_comp_guide
[kubernetes完全ガイド第2版](https://book.impress.co.jp/books/1119101148)

## Basic Commands

```bash
kubectl version
Client Version: v1.32.2
Kustomize Version: v5.5.0
Server Version: v1.31.4

kubectl get namespace

kubectl config current-context
kubectl config get-contexts
kubectl config get-users
kubectl config set-context guide \
  --cluster=docker-desktop \
  --user=ieiri \
  --namespace=guide
kubectl config use-context guide
```
