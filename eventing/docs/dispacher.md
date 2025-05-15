# Dispatcher

- [Dispatcher](#dispatcher)

```markdown
Channel 実装（例: InMemoryChannel）ごとに:

┌──────────────────────┐
│ 1つの dispatcher Pod │───→ Trigger を参照 → subscriber に配信
└──────────────────────┘
```

- 複数の Channel（CR）や Broker を処理していても、dispatcher は1つで全監視
- imc-dispatcher は全 InMemoryChannel のイベントを監視して処理します

| 目的               | どこに実装する？                | 説明                                               |
| ---------------- | ----------------------- | ------------------------------------------------ |
| イベント種別ごとに処理を変えたい | **Trigger** を追加         | `filter.attributes` で分岐。Subscriber をそれぞれ分ける      |
| 処理内容を変えたい        | **Subscriber（Service）** | Knative Service で好きなロジックを書く（Go/Python/Node.jsなど） |
| エラーハンドリングをしたい    | **DeliverySpec / DLQ**  | Trigger に `delivery.deadLetterSink` を追加          |
| 配信先を動的に切り替えたい    | **Trigger を動的に生成/削除**   | 条件に応じて Trigger を生成するControllerを作るなども可            |
| イベントを改変して転送したい   | **中継Serviceを挟む**        | 変換用の KnService を挟んで、Broker に再投稿する構成も可能           |

```mermaid
graph TD
  A[Broker] --> T1[Trigger A<br/>type=ping] --> S1[Subscriber A]
  A --> T2[Trigger B<br/>type=order] --> S2[Subscriber B]
  T3[Trigger C<br/>type=alert] --> S3[新しい通知処理]

  click T3 "https://github.com/your-org/my-alert-handler"
```

---

```mermaid
graph TD
  subgraph Cluster
    A["InMemoryChannel<br/>(CloudEventバッファ)"]
    B["Dispatcher<br/>(imc-dispatcher)"]
    C1[Trigger A<br/>filter: type=ping<br/>→ subscriber A]
    C2[Trigger B<br/>filter: type=order<br/>→ subscriber B]
    D1[Subscriber A<br/>event-display Service]
    D2[Subscriber B<br/>order-processor Service]
  end

  E[PingSource<br/>毎分CloudEvent生成] --> A
  A --> B
  B --> C1
  B --> C2
  C1 -->|フィルタ一致| D1
  C2 -->|フィルタ一致| D2
```

```mermaid
flowchart TD
    subgraph Knative Cluster
        A[CloudEvent 発生<br>from PingSource]
        B["Broker<br>(InMemoryChannel)"]
        C["dispatcher<br>(imc-dispatcher)"]
        D[Kubernetes API<br>Trigger リスト取得]
        E["Trigger: filter = { type: ping }<br>subscriber = event-display"]
        F[CloudEventを HTTP POST]
        G["Knative Service<br>(event-display)"]
    end

    A --> B
    B --> C
    C --> D
    D --> E
    C -->|CloudEvent 属性が Trigger の filter にマッチ| F
    F --> G
```

```markdown
[PingSource]
     |
     v
[InMemoryChannel (Broker)]
     |
     v
[Dispatcher (imc-dispatcher)]
     |
     |-- Trigger A: type=ping ----> [Subscriber A]
     |
     |-- Trigger B: type=order ---> [Subscriber B]
```
