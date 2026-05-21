# k8s_comp_guide
[kubernetes完全ガイド第2版](https://book.impress.co.jp/books/1119101148)

## Basic Commands

after 2026-04-06
```bash
kubectl version
Client Version: v1.34.1
Kustomize Version: v5.7.1
Server Version: v1.34.3

kubectl get namespace
NAME                 STATUS   AGE
default              Active   18d
kube-node-lease      Active   18d
kube-public          Active   18d
kube-system          Active   18d
local-path-storage   Active   18d

kubectl create namespace guide
```

before 2026-04-06
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
  --user=docker-desktop \
  --namespace=guide
kubectl config use-context guide
```
