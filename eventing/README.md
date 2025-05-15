# Knative Eventing

Knative Eventing は、疎結合されたサービスによるイベント駆動アーキテクチャを可能にする API のコレクションです。イベントソースとイベントコンシューマの遅延バインディングを可能にする構成可能なプリミティブを提供します。

## 主な機能

- **イベントソース**：外部システムに接続してイベントを受信
- **Broker/Trigger モデル**：イベントの生成と消費の分離
- **CloudEvents 準拠**：標準化されたイベント形式
- **配信保証**：様々なチャネル実装による少なくとも1回の配信保証
- **エラー処理**：配信失敗時の Dead Letter Sink
- **フィルタリング**：属性に基づくイベントのフィルタリング

## コンポーネント

### イベントソース

イベントソースは、イベント生成者を表す Kubernetes カスタムリソースです。例として：

- **PingSource**：指定されたスケジュールでイベントを生成（cron ジョブのように）
- **ApiServerSource**：Kubernetes リソースの変更を監視
- **KafkaSource**：Kafka トピックからイベントを読み取り
- **GitHubSource**：GitHub からの webhook イベントを受信

### Broker と Trigger

- **Broker**：コンシューマを知ることなくイベントを送信できるイベントハブとして機能
- **Trigger**：Broker からイベントをフィルタリングし、サブスクライバーに配信

### チャネル

チャネルは、Knative Channel CRD を実装するイベント永続化レイヤーです：

- **InMemoryChannel**：シンプルなインメモリ実装（本番環境には非推奨）
- **KafkaChannel**：永続性と配信に Apache Kafka を使用
- **NatssChannel**：永続性と配信に NATS Streaming を使用

## ディレクトリ構造

このディレクトリには以下が含まれています：

- `broker/`：Broker 設定
- `channel/`：Channel 設定
- `source/`：イベントソースの例
- `trigger/`：Trigger 設定
- `examples/`：追加サンプル
- `docs/`：Knative Eventing の様々な概念に関するドキュメントファイル

## イベントフローアーキテクチャ

```
┌──────────────┐     ┌──────────┐     ┌─────────┐     ┌────────────┐
│ Event Source │────►│  Broker  │────►│ Trigger │────►│ Subscriber │
└──────────────┘     └──────────┘     └─────────┘     └────────────┘
                          │
                          │
                     ┌────▼────┐
                     │ Channel │
                     └─────────┘
```

## Dispatcher アーキテクチャ

```
┌──────────────┐     ┌──────────┐     ┌─────────────┐     ┌─────────┐     ┌────────────┐
│ Event Source │────►│  Broker  │────►│ Channel     │────►│ Trigger │────►│ Subscriber │
└──────────────┘     └──────────┘     │ (InMemory/  │     └─────────┘     └────────────┘
                                      │  Kafka)     │
                                      └─────────────┘
                                            │
                                      ┌─────▼─────┐
                                      │ Dispatcher │
                                      └───────────┘
```

## 使用例

1. Broker の作成：
   ```bash
   kubectl apply -f broker/broker.yaml
   ```

2. イベントソースの作成：
   ```bash
   kubectl apply -f source/ping-source.yaml
   ```

3. サブスクライバーサービスの作成：
   ```bash
   kubectl apply -f trigger/event-display.yaml
   ```

4. Broker とサブスクライバーを接続する Trigger の作成：
   ```bash
   kubectl apply -f trigger/trigger.yaml
   ```

5. サブスクライバーのログを表示：
   ```bash
   kubectl logs -l serving.knative.dev/service=event-display -c user-container -f
   ```

## 追加リソース

- [Knative Eventing ドキュメント](https://knative.dev/docs/eventing/)
- [CloudEvents 仕様](https://cloudevents.io/)
- [イベント配信ドキュメント](https://knative.dev/docs/eventing/event-delivery/)
