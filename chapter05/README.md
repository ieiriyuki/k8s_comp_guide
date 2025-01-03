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
