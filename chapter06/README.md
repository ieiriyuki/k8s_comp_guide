# Service API
- クラスタ上のコンテナに対するエンドポイントの提供, ラベルに一致するコンテナのディスカバリ
- Service: L4 ロードバランス
  - ClusterIP
  - ExternalIP
  - NodePort
  - LoadBalancer
  - Headless
  - ExternalName
  - None-Selector
- Ingress: L7 ロードバランス

## K8s Cluster Network & Service
- 1 Pod: 1 IP
  - Pod 内: localhost
  - Pod 間: IP
- 基本知識
  - ノードごとのネットワークセグメント
  - VXLAN や L2 Routing でノード間通信
- Service のロードバランシングには, 外部サービスが払い出す Virtual IP やクラスタ内でのみ利用可能な ClusterIP などがある
- `sample-deployment.yaml` に対して Service を設定する
- `kubectl get pod sample-deployment-c94c6879f-2ctzx -o jsonpath='{.metadata.labels}'`
  - `{"app":"sample-app","pod-template-hash":"c94c6879f"}`
  - `kubectl get pods`
- `kubectl get pods -l app=sample-app -o custom-columns="NAME:{metadata.name},IP:{status.podIP}"`
- `kubectl get service sample-clusterip`

```bash
kubectl describe  service sample-clusterip
Name:                     sample-clusterip
Namespace:                guide
Labels:                   <none>
Annotations:              <none>
Selector:                 app=sample-app
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.100.67.253
IPs:                      10.100.67.253
Port:                     http-port  8080/TCP
TargetPort:               80/TCP
Endpoints:                10.1.2.2:80,10.1.2.1:80,10.1.2.0:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

- `kubectl run --image=amsy810/tools:v2.0 --restart=Never --rm -i testpod --command -- curl -s http://10.100.67.253:8080`
- 複数ポートを service に持たせることもできる
  - named-port

### DNS & Service Discovery

環境変数を利用したサービスディスカバリ
- `kubectl exec -it deployment-name -- env | grep -i sample_clusterip`

DNS A レコードを利用したサービスディスカバリ
```bash
kubectl run --image=amsy810/tools:v2.0 --restart=Never --rm -i testpod \
  --command -- curl -s http://sample-clusterip:8080

kubectl run --image=amsy810/tools:v2.0 --restart=Never --rm -i testpod \
  --command -- dig sample-clusterip.guide.svc.cluster.local
;; QUESTION SECTION:
;sample-clusterip.guide.svc.cluster.local. IN A

;; ANSWER SECTION:
sample-clusterip.guide.svc.cluster.local. 30 IN A 10.96.202.230

kubectl run --image=amsy810/tools:v2.0 --restart=Never --rm -i testpod \
  --command -- cat /etc/resolv.conf
```

DNS SRV レコードを利用したサービスディスカバリ

Port名とProtocolを利用することでサービス名を提供しているPort番号を含めたエンドポイントをDNSで解決する仕組み


```bash
[_ServiceのPort名].[_Protocol].[Service名].[Namespace名].svc.cluser.local

kubectl run --image=amsy810/tools:v2.0 --restart=Never --rm -i testpod \
  --command -- dig _http-port._tcp.sample-clusterip.guide.svc.cluster.local SRV
;; ANSWER SECTION:
_http-port._tcp.sample-clusterip.guide.svc.cluster.local. 30 IN SRV 0 100 8080 sample-clusterip.guide.svc.cluster.local.

;; ADDITIONAL SECTION:
sample-clusterip.guide.svc.cluster.local. 30 IN A 10.96.202.230
```

dnsPolicyを使ってPodのDNSサーバの設定を明示的に行わない限り, 起動するすべてのPodはクラスタ内DNSを利用して名前解決を行う (`*.cluster.local`が保存されている)

Node Local DNS Cache

## ClusterIP Service
- type: ClusterIP
  - クラスタ内からのみ疎通性があるInternal Networkに作られる仮想IP
