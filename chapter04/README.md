# API リソースと kubectl

## 書籍の前提
kubectl: 1.18.2

```bash
gcloud container clusters create k8s \
  --cluster-version 1.16.8-gke.15 \
  --zone asia-northeast1-a \
  --machine-type n1-standard-4 \
  --enable-network-policy \
  --enable-vertical-pod-autoscaling

gcloud container clusters get-credentials k8s --zone asia-northeast1-a

# alpha features
gcloud container clusters create k8s-alpha \
  --cluster-version 1.16.8-gke.15 \
  --zone asia-northeast1-a \
  --num-node 3 \
  --machine-type n1-standard-4 \
  --enable-network-policy \
  --enable-vertical-pod-autoscaling \
  --enable-kubernetes-alpha \
  --no-enable-autorepair \
  --no-enable-autoupgrade

kubectl create clusterrolebinding \
  user-cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=example@gmail.com

gcloud container clusters delete k8s --zone asia-northeast1-a

# workload identity
gcloud container clusters update k8s \
  --zone asia-northeast1-a \
  --workload-pool=${PROJECT}.svc.id.goog

gcloud container node-pools update default-pool \
  -cluster k8s \
  --zone asia-northeast1-a \
  --workload-metadata=GKE_METADATA
```

でもとりあえず Rancher Desktop の Kubernetes 機能を使うよ

## API
- workloads api
- service api
- config & storage api
- cluster api
- metadata api

## namespace
- 仮想的なクラスタ分離機能
- 初期状態
  - kube-system: クラスタのコンポーネントやアドオン
  - kube-public: 全ユーザーが利用できる configmap など
  - kube-node-lease: ノードのハートビート情報
  - default
- マイクロサービスチームごとに分ける
  - prd, stg, dev はクラスタで分けるべき
  - クラスタ更新の影響が独立する
  - 名前を流用できる

### 権限
- RBAC
- NetworkPolicy

## kubetcl
- 認証情報: `~/.kube/config`
- context: cluster-user-namespace の組
- sample
  - `kubectl config set-cluster name --server=url`
  - `kubectl config set-credentials name --client-certificate=.crt --client-key=.key --embed-certs=true`
  - `kubectl config set-context name --cluster=c --user=u --namespace=n`

## 4.5.3
```bash
kubectl config set-context kubectl create namespace chapter-04
kubectl config set-context chapter-04 --cluster=rancher-desktop --user=rancher-desktop --namespace=chapter-04
kubectl config use-context chapter-04

# 作成に create が使えるが, 更新もできる apply を使う
kubectl apply -f sample-pod.yaml
kubectl get pods
kubectl delete pod sample-pod # --grace-period 0
# 前回適用したマニフェストは
# metadata.annotations.kubectl.kubernetes.io/last-applied-configuration にある

kubectl apply -f sample-pod.yaml --server-side
kubectl set image pod sample-pod sample-container=nginx:1.17
kubectl apply -f sample-pod.yaml --server-side

error: Apply failed with 1 conflict: conflict with "kubectl-set" using v1: .spec.containers[name="sample-container"].image
Please review the fields above--they currently have other managers. Here
are the ways you can resolve this warning:
* If you intend to manage all of these fields, please re-run the apply
  command with the `--force-conflicts` flag.
* If you do not intend to manage all of the fields, please edit your
  manifest to remove references to the fields that should keep their
  current managers.
* You may co-own fields by updating your manifest to match the existing
  value; in this case, you'll become the manager if the other manager(s)
  stop managing the field (remove it from their configuration).
See https://kubernetes.io/docs/reference/using-api/server-side-apply/#conflicts
```

```bash
# サーバーサイドの更新は metadata.managedFields で見れる, らしい
kubectl get pod sample-pod -o yaml
# --field-manager オプションで manager 名を変えることができる
```

## 4.5.6 rollout restart
- deployment に対して利用可能
- `kubectl apply -f sample-deployment.yaml`
- `kubectl rollout restart deployment sample-deployment`
- pod に対しては利用不可能

## 4.5.7 generateName
- kubectl create で利用可能
- `metadata: generateName: prefix-` をマニフェストに書く

## 4.5.8 wait
- リソースの状態を待ってから実行
- `kubectl wait --for [state] --timeout [sec]`
- `kubectl wait --for=condition=Ready pod/sample-pod`
  - condition=PodScheduled
  - delete
  - --all