# Knative ドキュメントと例

このリポジトリには、Kubernetes ベースのサーバーレスワークロードのデプロイと管理のためのプラットフォームである Knative のドキュメントと例が含まれています。

## ディレクトリ構造

```
.
├── README.md                 # このファイル
├── serving/                  # Knative Serving 関連ファイル
│   └── hello.yaml            # Hello world サンプルサービス
└── eventing/                 # Knative Eventing 関連ファイル
    ├── broker/               # Broker 設定
    │   ├── broker.yaml       # デフォルト broker 設定
    │   └── broker-direct.yaml # 直接設定のある Broker
    ├── channel/              # Channel 設定
    │   ├── in-memory-channel-configmap.yaml # インメモリチャネル ConfigMap
    │   └── in-memory-channel-correct.yaml  # 正しいインメモリチャネル設定
    ├── source/               # イベントソース
    │   └── ping-source.yaml  # PingSource サンプル
    ├── trigger/              # Trigger 設定
    │   ├── event-display.yaml # イベント表示サービス
    │   └── trigger.yaml      # Trigger サンプル
    ├── examples/             # 追加サンプル
    ├── docs/                 # ドキュメントファイル
    │   ├── cloud-event.md    # CloudEvent ドキュメント
    │   ├── dead-letter-sink.md # Dead Letter Sink ドキュメント
    │   ├── dispacher.md      # Dispatcher ドキュメント
    │   ├── kafka-channel.md  # Kafka Channel ドキュメント
    │   └── use-case.md       # Knative のユースケース
```

## 概要

| コンポーネント              | 役割                                                |
| ---------------------- | ------------------------------------------------- |
| **Serving**            | 自動スケーリング（ゼロへのスケールを含む）を備えた HTTP アプリケーションのランタイム環境 |
| **Eventing**           | ソースからハンドラーへのイベントルーティングによるイベント駆動アーキテクチャの実現 |
| **Functions（旧 Build）** | ソースコードからコンテナをビルド（この機能は廃止予定） |

## Knative Serving

Knative Serving は以下を提供するコアコンポーネントです：

- **自動スケーリング（Scale-to-Zero）**：トラフィックがない時にポッドを停止し、リクエストが来たときに自動的に起動
- **リビジョン管理**：各デプロイメントのリビジョンを作成し、以前のバージョンを保持可能
- **トラフィック分割**：「新バージョンに10%、旧バージョンに90%」などのトラフィック分配が可能
- **Istio または Kourier との統合**：トラフィック制御と認証拡張のためのイングレスとして使用

## Knative Eventing

Knative Eventing はイベント駆動アプリケーションのためのインフラストラクチャを提供します：

- **イベントソース**：Kafka、Webhook、CronJob などの外部イベントを受信
- **Broker / Trigger モデル**：Broker がイベントを受け取り、Trigger が条件に応じて処理先にルーティング
- **CloudEvents 準拠**：イベント形式は CloudEvents に基づいて標準化

## 始め方

### Knative Serving のセットアップ

```bash
# Kubernetes クラスタの作成
kind create cluster --name knative

# Kourier（ネットワーキングレイヤー）のインストール
kubectl apply -f https://github.com/knative/net-kourier/releases/latest/download/kourier.yaml

# Knative Serving のインストール
kubectl apply -f https://github.com/knative/serving/releases/latest/download/serving-crds.yaml
kubectl apply -f https://github.com/knative/serving/releases/latest/download/serving-core.yaml

# イングレスの設定
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress.class":"kourier.ingress.networking.knative.dev"}}'

# ドメインの設定
kubectl patch configmap/config-domain -n knative-serving \
  --type merge \
  -p '{"data":{"127.0.0.1.sslip.io":""}}'
```

### Knative Eventing のセットアップ

```bash
# デフォルト名前空間で eventing を有効化
kubectl label namespace default knative-eventing-injection=enabled

# デフォルト broker の設定
kubectl patch configmap config-br-defaults -n knative-eventing --type merge \
  -p '{"data":{"default-br-config":"{\"brokerClass\":\"MTChannelBasedBroker\",\"broker.config\":{\"apiVersion\":\"v1\",\"kind\":\"ConfigMap\",\"name\":\"in-memory-channel\",\"namespace\":\"knative-eventing\"}}"}}'

# Knative Eventing のインストール
kubectl apply -f https://github.com/knative/eventing/releases/latest/download/eventing-crds.yaml
kubectl apply -f https://github.com/knative/eventing/releases/latest/download/eventing-core.yaml
```

## コンポーネントとアーキテクチャ

### Knative Serving コンポーネント

- **Knative Service**：ワークロードのライフサイクルを管理する抽象化されたアプリケーション
- **Revision**：イミュータブルな設定を持つ各デプロイメントのスナップショット
- **Route**：トラフィック分割とルーティングを制御
- **Activator**：ポッドがゼロの時にリクエストを処理し、起動をトリガー
- **Autoscaler**：リクエスト量に基づいてスケーリングを制御

### Knative Eventing コンポーネント

- **Event Source**：外部ソース（PingSource、KafkaSource など）からイベントをインポート
- **Broker**：イベントの受信とルーティングのハブ
- **Trigger**：条件に基づいてイベントをサブスクライバー（Sink）に転送
- **Channel**：イベントの永続性と配信を提供（InMemoryChannel、KafkaChannel）
- **Dispatcher**：チャネルを監視し、サブスクライバーにイベントを配信

## ユースケース

Knative は様々なユースケースを可能にします：

- 🌀 **イベント駆動のビジネスプロセス**：申請・更新・Webhook などに応じた自動処理
- ☁️ **サーバーレス API ホスティング**：ユーザーリクエストに応じて自動的にポッドが起動
- 🔁 **処理の疎結合化・非同期化**：処理をイベント単位で分離し、順次非同期で処理
- 🧵 **マイクロサービス間連携の標準化**：CloudEvents 形式でメッセージングを統一
- 💸 **低負荷バッチ処理の自動起動**：負荷があるときだけ自動起動、無負荷ならゼロポッド
- 🔔 **通知・Slack 連携処理**：Trigger → Webhook → 通知
- 📊 **業務ログの構造化イベントとして集約**：CloudEvents で各処理ログを統一的に出力
- 🧩 **CI/CD の自動フック**：GitHub Webhook → Broker → イメージビルド処理など

## 追加リソース

- [Knative 公式ドキュメント](https://knative.dev/docs/)
- [CloudEvents 仕様](https://cloudevents.io/)
- [Knative GitHub リポジトリ](https://github.com/knative)
