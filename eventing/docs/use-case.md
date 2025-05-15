# ✅ Knative を使って社内で実現できること一覧

- [✅ Knative を使って社内で実現できること一覧](#-knative-を使って社内で実現できること一覧)

| 実現できること                  | 説明                                    | Knativeの機能                          |
| ------------------------ | ------------------------------------- | ----------------------------------- |
| 🌀 **イベント駆動の業務処理**       | 申請・更新・Webhook などに応じて自動処理              | Eventing（Broker + Trigger）          |
| ☁️ **サーバレスAPIのホスティング**   | ユーザーリクエストに応じて自動でPodが立ち上がる             | Serving（scale-to-zero）              |
| 🔁 **処理の疎結合化・非同期化**      | 処理をイベント単位で分離し、順次非同期で処理                | Eventing + CloudEvents              |
| 🧵 **マイクロサービス間連携の標準化**   | CloudEvents形式でメッセージングを統一              | Eventing（Ping/Kafka/HTTP Source）    |
| 💸 **社内用の低負荷バッチ処理の自動起動** | 負荷があるときだけ自動起動 → 無負荷なら0Pod             | Serving（AutoScaling, scale-to-zero） |
| 🔔 **社内の通知・Slack連携処理**   | Trigger → Webhook → 通知                | Trigger + SinkBinding               |
| 📊 **業務ログの構造化イベントとして集約** | CloudEventsで各処理ログを統一的に出力              | CloudEvents + Logging連携             |
| 🧩 **CI/CD の自動フック**      | GitHub Webhook → Broker → イメージビルド処理など | HTTPSource + Serving                |
