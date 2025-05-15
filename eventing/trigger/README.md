# Knative Eventing トリガー

トリガーは、イベント属性に基づいて Broker からイベントをフィルタリングし、一致するイベントをサブスクライバーに配信する Knative Eventing リソースです。

## トリガーとは？

トリガーは、特定の属性を持つイベントを Broker からサブスクライブするものを表します。トリガーは：
- 特定の Broker に接続
- CloudEvent 属性に基づいてイベントをフィルタリング
- 一致するイベントをサブスクライバー（Knative Service、Channel など）に配信
- エラー処理のための配信仕様を含めることができる

## トリガーのコンポーネント

トリガーは以下で構成されています：

1. **Broker 参照**：イベントを受信する Broker
2. **フィルター**：CloudEvents と照合する属性
3. **サブスクライバー**：一致するイベントの送信先
4. **配信仕様**（オプション）：配信試行、再試行、デッドレターシンクの設定

## このディレクトリのファイル

- `trigger.yaml`：PingSource イベントをフィルタリングするトリガーの例
- `event-display.yaml`：受信したイベントをログに記録する Knative Service

## トリガーの例

```yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: ping-trigger
  namespace: default
spec:
  broker: default
  filter:
    attributes:
      type: dev.knative.sources.ping
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: event-display
```

## イベント表示サービス

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: event-display
  namespace: default
spec:
  template:
    spec:
      containers:
      - image: gcr.io/knative-releases/knative.dev/eventing/cmd/event_display
```

## イベントのフィルタリング

トリガーは CloudEvent 属性に基づいてイベントをフィルタリングできます：

```yaml
filter:
  attributes:
    type: dev.knative.sources.ping
    source: /apis/v1/namespaces/default/pingsources/test-ping
    subject: daily-job
```

## デッドレターシンク

配信に失敗したイベントのためにデッドレターシンクを設定できます：

```yaml
spec:
  # ... 他のフィールド
  delivery:
    deadLetterSink:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: error-logger
```

## 使用方法

1. イベント表示サービスの作成：
   ```bash
   kubectl apply -f event-display.yaml
   ```

2. トリガーの作成：
   ```bash
   kubectl apply -f trigger.yaml
   ```

3. サブスクライバーのログの表示：
   ```bash
   kubectl logs -l serving.knative.dev/service=event-display -c user-container -f
   ```

## 追加リソース

- [Knative Eventing トリガードキュメント](https://knative.dev/docs/eventing/triggers/)
- [イベントフィルタリング](https://knative.dev/docs/eventing/event-filtering/)
- [イベント配信](https://knative.dev/docs/eventing/event-delivery/)
