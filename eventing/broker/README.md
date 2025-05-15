# Knative Eventing Brokers

Broker は、様々なソースからの CloudEvents を収集し、Trigger を通じてサブスクライバーが消費できるようにするイベントハブを提供する Knative Eventing リソースです。

## Broker とは？

Broker は以下のことができるイベントメッシュを表します：
- HTTP POST を介して CloudEvents を受信
- イベントを一時的に保存
- Trigger を介したイベントのフィルタリングを可能に
- サブスクライバーにイベントを配信

## Broker の実装

Knative Eventing は異なる Broker 実装をサポートしています：

1. **MTChannelBasedBroker**（マルチテナントチャネルベースの Broker）：
   - イベント保存のためにチャネル実装を使用
   - 異なるチャネルタイプ（InMemoryChannel、KafkaChannel など）を使用するように設定可能
   - Knative Eventing のデフォルト実装

2. **RabbitMQBroker**：
   - イベントメッシュとして RabbitMQ を使用
   - AMQP ベースのメッセージングを提供

3. **KafkaBroker**：
   - （チャネル抽象化なしで）Apache Kafka を直接使用
   - 高いスケーラビリティと永続性を提供

## このディレクトリのファイル

- `broker.yaml`：シンプルな MTChannelBasedBroker 定義
- `broker-direct.yaml`：明示的な InMemoryChannel 設定を持つ MTChannelBasedBroker

## Broker の作成

### デフォルト Broker

```yaml
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: default
  namespace: default
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
```

### 特定のチャネル設定を持つ Broker

```yaml
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: default
  namespace: default
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
spec:
  config:
    apiVersion: v1
    kind: ConfigMap
    name: in-memory-channel
    namespace: knative-eventing
```

## 使用方法

1. Broker の作成：
   ```bash
   kubectl apply -f broker.yaml
   ```

2. Broker へのイベント送信：
   ```bash
   curl -v -X POST \
     -H "Content-Type: application/cloudevents+json" \
     -H "Ce-Id: 123456789" \
     -H "Ce-Specversion: 1.0" \
     -H "Ce-Type: dev.knative.example" \
     -H "Ce-Source: example/source" \
     -d '{"message": "Hello from curl!"}' \
     http://broker-ingress.knative-eventing.svc.cluster.local/default/default
   ```

3. Broker からのイベントをサブスクライブするための Trigger を作成します。

## 追加リソース

- [Knative Eventing Broker ドキュメント](https://knative.dev/docs/eventing/brokers/)
- [Broker 実装](https://knative.dev/docs/eventing/broker-trigger/)
