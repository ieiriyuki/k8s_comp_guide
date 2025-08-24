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
