# Knative Eventing チャネル

チャネルは、単一のイベント転送と永続化レイヤーを定義する Knative Eventing リソースです。メッセージング実装によって異なる機能と保証が提供されます。

## チャネルとは？

チャネルは以下の機能を持つイベントバスを表します：
- 様々なソースからのイベントの取り込み
- イベントの保存/バッファリング
- 複数のサブスクライバーへのイベントのファンアウト
- 実装に基づく配信保証

## チャネルの実装

Knative Eventing は異なるチャネル実装をサポートしています：

1. **InMemoryChannel**：
   - シンプルなインメモリ実装
   - 永続性なし（ポッドが再起動するとイベントは失われる）
   - 開発とテストに適している
   - 本番環境での使用は推奨されない

2. **KafkaChannel**：
   - イベントの保存と配信に Apache Kafka を使用
   - 永続性と高いスケーラビリティを提供
   - 本番環境での使用が推奨される

3. **NatssChannel**：
   - イベント配信に NATS Streaming を使用
   - 少なくとも1回の配信保証を提供

4. **GcpPubSubChannel**：
   - Google Cloud Pub/Sub を使用
   - Google Cloud 環境に適している

## このディレクトリのファイル

- `in-memory-channel-configmap.yaml`：InMemoryChannel 設定用の ConfigMap
- `in-memory-channel-correct.yaml`：InMemoryChannel の正しい設定

## チャネル設定

### InMemoryChannel ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: in-memory-channel
  namespace: knative-eventing
data:
  channelTemplateSpec: |
    {
      "apiVersion": "messaging.knative.dev/v1",
      "kind": "InMemoryChannel"
    }
```

## チャネルアーキテクチャ

```
┌──────────────┐     ┌──────────┐     ┌────────────┐
│ Event Source │────►│ Channel  │────►│ Subscriber │
└──────────────┘     └──────────┘     └────────────┘
                          │
                          │
                     ┌────▼────┐
                     │Dispatcher│
                     └─────────┘
```

## 使用方法

1. チャネルを直接作成：
   ```bash
   kubectl apply -f - <<EOF
   apiVersion: messaging.knative.dev/v1
   kind: InMemoryChannel
   metadata:
     name: my-channel
   EOF
   ```

2. チャネルへのイベント送信：
   ```bash
   curl -v -X POST \
     -H "Content-Type: application/cloudevents+json" \
     -H "Ce-Id: 123456789" \
     -H "Ce-Specversion: 1.0" \
     -H "Ce-Type: dev.knative.example" \
     -H "Ce-Source: example/source" \
     -d '{"message": "Hello from curl!"}' \
     http://my-channel-kn-channel.default.svc.cluster.local
   ```

3. チャネルからイベントを受信するためのサブスクリプションを作成します。

## 追加リソース

- [Knative Eventing チャネルドキュメント](https://knative.dev/docs/eventing/channels/)
- [チャネル実装](https://knative.dev/docs/eventing/channels/channel-types/)
