# ✅ ③ CloudEvent 属性の具体例

- [✅ ③ CloudEvent 属性の具体例](#--cloudevent-属性の具体例)
  - [💡 CloudEvent の HTTP POST 例](#-cloudevent-の-http-post-例)
  - [🔍 Trigger で使える filter.attributes](#-trigger-で使える-filterattributes)

## 💡 CloudEvent の HTTP POST 例

```http
POST / HTTP/1.1
Content-Type: application/cloudevents+json

{
  "specversion": "1.0",
  "id": "abc-123",
  "type": "dev.knative.sources.ping",
  "source": "/apis/v1/namespaces/default/pingsources/test-ping",
  "subject": "daily-job",
  "time": "2025-05-15T12:34:56Z",
  "datacontenttype": "application/json",
  "data": {
    "message": "Hello!"
  }
}
```

## 🔍 Trigger で使える filter.attributes

```yaml
filter:
  attributes:
    type: dev.knative.sources.ping
    source: /apis/v1/namespaces/default/pingsources/test-ping
    subject: daily-job
```
