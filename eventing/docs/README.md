# Knative Eventing ドキュメント

このディレクトリには、Knative Eventing の様々な側面に関する詳細なドキュメントが含まれています。

## 内容

- `cloud-event.md`：CloudEvents 形式と例に関する詳細
- `dead-letter-sink.md`：イベント配信失敗時の処理のための Dead Letter Sink に関するドキュメント
- `dispacher.md`：Knative Eventing における Dispatcher コンポーネントに関する情報
- `kafka-channel.md`：InMemoryChannel の代替として Kafka Channel を使用するためのガイド
- `use-case.md`：企業環境での Knative の様々なユースケース

## CloudEvents

CloudEvents は、イベントデータを共通の方法で記述するための仕様です。Knative Eventing は、異なるソースとシンク間でイベント形式を標準化するために CloudEvents を使用します。

## Dead Letter Sink (DLS)

Dead Letter Sink は、意図した宛先に配信できなかったイベントを処理する方法を提供します。これは、本番環境での信頼性の高いイベント処理に不可欠です。

## Dispatcher

Dispatcher は、Knative Eventing のコンポーネントで、チャネルを監視し、トリガーフィルターに基づいてサブスクライバーにイベントを配信します。

## Kafka Channel

KafkaChannel は、InMemoryChannel の本番環境対応の代替品であり、イベント配信のための永続性、パーティショニング、高いスケーラビリティを提供します。

## ユースケース

Knative は、イベント駆動型ビジネスプロセス、サーバーレス API ホスティング、疎結合非同期処理など、様々な企業ユースケースを可能にします。

## 追加リソース

- [Knative Eventing ドキュメント](https://knative.dev/docs/eventing/)
- [CloudEvents 仕様](https://cloudevents.io/)
- [Knative GitHub リポジトリ](https://github.com/knative)
