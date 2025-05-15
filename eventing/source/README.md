# Knative Eventing ソース

イベントソースは、イベント生成者を表す Knative Eventing リソースです。外部システムからイベントを生成またはインポートし、イベントシンク（Broker、Channel、Service など）に送信します。

## イベントソースとは？

イベントソースは、以下を行う Kubernetes カスタムリソースです：
- 外部システムに接続するか、内部でイベントを生成する
- 受信データを CloudEvents 形式に変換する
- 指定されたシンクにイベントを配信する

## イベントソースの種類

Knative Eventing はいくつかの組み込みイベントソースを提供しています：

1. **PingSource**：
   - スケジュールに従ってイベントを生成（cron ジョブのように）
   - 定期的なタスクやスケジュールされたイベントに有用

2. **ApiServerSource**：
   - Kubernetes リソースの変更を監視
   - リソースが作成、更新、または削除されたときにイベントを生成

3. **KafkaSource**：
   - Kafka トピックからメッセージを読み取る
   - それらを CloudEvents に変換

4. **GitHubSource**：
   - GitHub からの webhook イベントを受信
   - CI/CD ワークフローに有用

5. **ContainerSource**：
   - イベントを生成するコンテナを実行
   - カスタムイベント生成ロジックを可能にする

6. **SinkBinding**：
   - PodSpec を持つ任意の Kubernetes リソースにシンク情報を注入
   - 任意のアプリケーションがシンクにイベントを送信できるようにする

## このディレクトリのファイル

- `ping-source.yaml`：毎分イベントを生成する PingSource の例

## PingSource の例

```yaml
apiVersion: sources.knative.dev/v1
kind: PingSource
metadata:
  name: test-ping
  namespace: default
spec:
  schedule: "*/1 * * * *"  # 毎分（cron 形式）
  data: '{"message": "Hello from PingSource!"}'
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: default
```

## 使用方法

1. PingSource の作成：
   ```bash
   kubectl apply -f ping-source.yaml
   ```

2. イベントが生成されていることを確認：
   ```bash
   # Broker に送信している場合、サブスクライバーサービスのログを確認
   kubectl logs -l serving.knative.dev/service=event-display -c user-container -f
   ```

## CloudEvent 形式

ソースによって生成されるイベントは CloudEvents 仕様に従います：

```json
{
  "specversion": "1.0",
  "id": "1234-abc...",
  "type": "dev.knative.sources.ping",
  "source": "/apis/v1/namespaces/default/pingsources/test-ping",
  "time": "2025-05-15T11:03:00.184993049Z",
  "data": {
    "message": "Hello from PingSource!"
  }
}
```

## 追加リソース

- [Knative Eventing ソースドキュメント](https://knative.dev/docs/eventing/sources/)
- [イベントソースの作成](https://knative.dev/docs/eventing/custom-event-source/)
