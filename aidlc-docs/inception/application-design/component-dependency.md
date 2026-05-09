# Component Dependency

## Dependency Matrix

| Component | Depends On | Depended By |
|-----------|-----------|-------------|
| **Frontend (React)** | BFF API Routes | — |
| **BFF API Routes** | Core API (FastAPI) | Frontend |
| **Auth Service** | DynamoDB | All Services (middleware) |
| **Closet Service** | DynamoDB, S3, Bedrock | Outfit Recommendation |
| **Calendar Service** | DynamoDB, Google Calendar API | Outfit Recommendation |
| **Weather Service** | DynamoDB (cache), OpenWeatherMap API | Outfit Recommendation |
| **Outfit Recommendation** | Closet, Calendar, Weather, Wear History, Bedrock | Frontend |
| **Wear History** | DynamoDB | Outfit Recommendation |

---

## Communication Patterns

### Frontend ↔ BFF
- **Protocol**: HTTPS (REST)
- **Format**: JSON
- **Auth**: HttpOnly Cookie (JWT session)

### BFF ↔ Core API
- **Protocol**: HTTPS (REST)
- **Format**: JSON
- **Auth**: Internal JWT or API Key

### Core API ↔ External APIs
- **Bedrock**: AWS SDK (boto3), IAM Role認証
- **OpenWeatherMap**: HTTPS REST, API Key
- **Google Calendar**: HTTPS REST, OAuth 2.0 Bearer Token

### Core API ↔ Data Stores
- **DynamoDB**: AWS SDK (boto3), IAM Role認証
- **S3**: AWS SDK (boto3), IAM Role認証, Presigned URLs for client

---

## Data Flow Diagram

```
+-------------+          +-------------+          +-------------------+
|             |  HTTPS   |             |  HTTPS   |                   |
|   Browser   | -------> |   Next.js   | -------> |   FastAPI         |
|   (React)   | <------- |   BFF       | <------- |   Core API        |
|             |   JSON   |             |   JSON   |                   |
+-------------+          +-------------+          +--------+----------+
                                                           |
                          +--------------------------------+--+--+--+
                          |                |               |  |  |  |
                          v                v               v  |  |  |
                   +-----------+    +-----------+   +------+  |  |  |
                   | DynamoDB  |    |    S3      |   |Bedrock| |  |  |
                   | (Data)    |    |  (Images)  |   |(Claude| |  |  |
                   +-----------+    +-----------+   +-------+ |  |  |
                                                              |  |  |
                          +-----------------------------------+  |  |
                          |                +---------------------+  |
                          v                v                        v
                   +-----------+    +-----------+            +-----------+
                   |OpenWeather|    |  Google    |            |  Google   |
                   |  Map API  |    | Calendar   |            |  OAuth    |
                   +-----------+    |   API      |            |  Server   |
                                    +-----------+            +-----------+
```

---

## Dependency Direction Rules

1. **Frontend → BFF → Core API**: 単方向依存（フロントエンドがバックエンドに依存）
2. **Core API → External Services**: 外部APIへのアウトバウンド呼び出し
3. **Outfit Recommendation → 他の全Service**: ファンアウト集約パターン
4. **Auth middleware → 全エンドポイント**: 横断的関心事（Cross-cutting concern）

---

## Critical Dependencies

| Dependency | 障害時の影響 | 緩和策 |
|-----------|------------|--------|
| Amazon Bedrock | 服装提案不可、AIタグ付け不可 | エラーメッセージ表示、手動タグフォールバック |
| OpenWeatherMap API | 天気情報なしで提案 | キャッシュ延長、天気なし提案モード |
| Google Calendar API | 手動予定入力に切替 | 手動入力での代替取得（必須機能） |
| DynamoDB | 全機能停止 | AWSマネージドサービスのSLA依存 |
| S3 | 画像アップロード/表示不可 | 画像なしでの服登録許可 |
