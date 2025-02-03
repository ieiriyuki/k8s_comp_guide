# Workload API
- pod
- replication controller
- replica set
- deployment
- daemon set
- stateful set
- job
- cron job

## Pod
- 基本的には 1 pod 1 container
- サブコンテナを起動することもある
- pod 単位で ip アドレスが割り当てられる
- サイドカー: メインコンテナに機能を追加する
- アンバサダー: 外部システムとのやりとりを代理する
  - 疎結合にできる
- アダプタ: 外部からのアクセスのインターフェースとなる
  - e.g. Prometheus の要求に対してフォーマットしたメトリクスを返す
- command: ENTRYPOINT
- args: CMD
- pod 名: alpha numeric
  - `.`, `-`
  - start & end with alphabet
- `spec.hostNetwork: true` でホストのネットワークを利用できるが基本的には利用しない
  - NodePort Service などを使う
- `spec.dnsPolicy` で DNS サーバに関する設定をする
  - ClusterFirst: デフォルト, クラスタ内の DNS に問い合わせを行う
  - None: Pod 定義内で静的に行う
    - `dnsConfig` などで設定
  - Default: ノードの `/etc/resolv.conf` を引き継ぐ
  - ClusterFirstWithHostNet: ClusterFirst と同等
- `spec.hostAliases` で `/etc/hosts` の書き換えができる
- `spec.containers[].workingDir` で作業ディレクトリを指定できる

## ReplicaSet
- `kind: ReplicaSet`
- セルフヒーリングしてくれる
- ラベルで対象を指定する
  - ラベルが一致しない場合はエラーになる
- ReplicaSet 以外で同ラベルの Pod を作るといずれかが削除される
  - ラベリングのルールを作るべき
- スケーリング
  - マニフェストを書き換えて `apply` する
  - `kubectl scale` する
- レプリカの制御条件
  - equality-based: 条件部に等価式を使う
  - set-based: 条件部に集合値ベースの条件も使用可能
    - `env In [development,staging]`
