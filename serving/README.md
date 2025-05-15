# Knative Serving

Knative Serving は、Kubernetes 上でコンテナ化されたアプリケーションをデプロイして実行するためのサーバーレスプラットフォームを提供する Knative のコンポーネントです。

## 主な機能

- **ゼロへのスケーリング（Scale-to-Zero）**：トラフィックがない場合、自動的にポッド数をゼロにスケールダウンしてリソースを節約
- **迅速なスケールアップ**：トラフィックが到着したときに素早くスケールアップ
- **リクエストベースの自動スケーリング**：同時リクエスト数に基づいてスケーリング
- **リビジョン管理**：コードと設定の不変なスナップショットを作成
- **トラフィック分割**：カナリアやブルー/グリーンなどの高度なデプロイ戦略を可能に
- **ネットワーキングレイヤーとの統合**：Istio、Kourier、または Contour と連携

## コンポーネント

- **Service (service.serving.knative.dev)**：ワークロードのライフサイクルを管理するトップレベルリソース
- **Route (route.serving.knative.dev)**：ネットワークエンドポイントを1つ以上のリビジョンにマッピング
- **Configuration (configuration.serving.knative.dev)**：デプロイメントの望ましい状態を維持
- **Revision (revision.serving.knative.dev)**：コードと設定の特定時点のスナップショット

## サンプル

このディレクトリには、シンプルな Hello World サンプルが含まれています：

- `hello.yaml`：Hello World アプリケーションをデプロイする基本的な Knative Service

## デプロイメント

Hello World サンプルをデプロイするには：

```bash
kubectl apply -f hello.yaml
```

デプロイされたサービスにアクセスするには：

```bash
# URL を取得
kubectl get ksvc helloworld

# Kourier ポートをフォワード
kubectl port-forward -n kourier-system svc/kourier 8080:80

# サービスにアクセス
curl -H "Host: helloworld.default.127.0.0.1.sslip.io" http://localhost:8080
```

## アーキテクチャ

```
┌───────────────┐
│ Knative      │
│ Service      │
└───────┬───────┘
        │
        ▼
┌───────────────┐     ┌───────────────┐
│ Configuration │     │ Route         │
└───────┬───────┘     └───────┬───────┘
        │                     │
        ▼                     │
┌───────────────┐             │
│ Revision      │◄────────────┘
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ Deployment    │
│ (Kubernetes)  │
└───────────────┘
```

## 追加リソース

- [Knative Serving ドキュメント](https://knative.dev/docs/serving/)
- [自動スケーリングドキュメント](https://knative.dev/docs/serving/autoscaling/)
- [トラフィック分割ドキュメント](https://knative.dev/docs/serving/traffic-management/)
