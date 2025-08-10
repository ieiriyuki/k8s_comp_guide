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

## Deployment
- replicaset を管理する
- pod や replicaset よりも細かい管理ができるため deployment を使うべき
- `kubectl apply -f sample-deployment.yaml --record`
- `kubectl get replicasets -o yaml | head`
- `kubectl set image deployment sample-deployment nginx-container=nginx:1.17 --record`
- `kubectl get deployments`
- `kubectl rollout status deployment sample-deployment`
- Pod の内容の変更があると, ReplicaSet が作成される
- `kubectl rollout history deployment sample-deployment`
- `kubectl rollout undo deployment sample-deployment --to-revision 1`
- `kubectl get replicasets`
- pause / resume で一時停止したり, 再開したりできる
- `spec.strategy.type`
  - Recreate: 削除してから作成, 余計なリソースを使わない, 切り替えが速い
  - RollingUpdate
    - maxUnavailable: 許容される不足 Pod 数
    - maxSurge: 超過 Pod 数
- minReadySeconds: Pod が ready 状態になってから deployment リソース的に pod の起動が完了したと判断するまでの最低秒数
- revisionHistoryLimit: deployment が保持する replicaset の数. ロールバック可能な履歴数
- progressDeadlineSeconds: recreate / rollingupdate 処理のタイムアウト時間. タイムアウト時間が経過した場合, 自動でロールバックされます

## DaemonSet
- replicaset の特殊な形
- 各ノードに pod を1つずつ配置する
- ノードを増やしたら自動で pod を起動してくれる
- Fluentd や Datadog のように全ノードで動作させたいプロセスなどに使う
- OnDelete: 他の要因で pod が再作成されるときに, 新しい定義で作り直す
- RollingUpdate: 即時 pod の更新を行う. デフォルト
  - maxSurge を設定することはできない
  - maxUnavailable を 0 にできない

## StatefulSet
- replicaset の特殊な形
- 作成される pod 名のサフィックスは数字
- データを永続化する仕組みを持っている
- `kubectl get statefulsets`
- `kubectl get persistentvolumeclaims`
- `kubectl scale` を使える
- ReplicaSet の場合, ランダムに削除されるためマスターがいる場合には向かない
- StatefulSet の場合, 新しいものから削除されるため, 最初のものをマスターにできる
- `spec.podManagementPolicy` を PArallel に設定することで同時並列起動にできる
- `OnDelete` と `RollingUpdate` を使える
  - `RollingUpdate` では `maxUnavailable` を利用できない
  - `partition` を設定して更新されない Pod を指定できる
- `kubectl exec -it sample-statefulset-0 -- df -h | grep /dev/sd`
  - 永続領域の確認
- `kubectl get persistentvolumeclaims` pod を削除しても volume は残る
- `kubectl delete persistentvolumeclaims www-sample-statefulset-(0..2)`
  - 不要な volume の削除

## Job
- Pod が停止することを前提にして作られている
- `kubectl get pods`
- `restartPolicy`
  - `Never`: pod 障害時に新規作成する
  - `OnFailure`: 同一の pod を利用して job を再開する
- completion, parallelism, backoffLimit でワークロードを調整する
  - ワークキュー型のジョブを使用する場合, completion を指定しない
  - この時処理全体の進捗を関するものが必要
  - 並列数を1にすると, single worker queue になる
- `ttlSecondsAfterFinished` でジョブ終了後に削除する
- `kubectl create job` で簡単なジョブを作成できる

## CronJob
