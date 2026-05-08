# Unit of Work Dependency Matrix

## 依存関係マトリクス

| Unit | 依存先 | 依存種別 | 説明 |
|------|--------|---------|------|
| Unit 1: Auth | — | — | 依存なし（基盤ユニット） |
| Unit 2: Closet | Unit 1 (Auth) | 認証必須 | 認証ミドルウェアを経由 |
| Unit 2: Closet | Bedrock, S3 | 外部サービス | 画像保存・AIタグ付け |
| Unit 3: Calendar | Unit 1 (Auth) | 認証必須 | 認証ミドルウェア + OAuthトークン管理 |
| Unit 3: Calendar | Google Calendar API | 外部サービス | 予定取得 |
| Unit 4: Weather | Unit 1 (Auth) | 認証必須 | 認証ミドルウェアを経由 |
| Unit 4: Weather | OpenWeatherMap API | 外部サービス | 天気情報取得 |
| Unit 5: Outfit | Unit 1 (Auth) | 認証必須 | 認証ミドルウェアを経由 |
| Unit 5: Outfit | Unit 2 (Closet) | データ参照 | 手持ち服データ取得 |
| Unit 5: Outfit | Unit 3 (Calendar) | データ参照 | 予定データ取得（フォールバック: 手動入力） |
| Unit 5: Outfit | Unit 4 (Weather) | データ参照 | 天気データ取得（フォールバック: 天気なし提案） |
| Unit 5: Outfit | Bedrock | 外部サービス | AI提案生成 |

## 依存関係図

```
+------------+
| Unit 1     |
| Auth       |<--+--+--+--+
+------------+   |  |  |  |
                  |  |  |  |
+------------+   |  |  |  |     +------------+
| Unit 2     |---+  |  |  |     | Bedrock    |
| Closet     |------+--+--+---->| (Claude)   |
+------------+   |  |  |        +------------+
                  |  |  |
+------------+   |  |  |        +------------+
| Unit 3     |---+  |  +------->| Google     |
| Calendar   |------+--+        | Calendar   |
+------------+   |  |           +------------+
                  |  |
+------------+   |  |           +------------+
| Unit 4     |---+  +---------->| OpenWeather|
| Weather    |------+           | Map API    |
+------------+   |              +------------+
                  |
+------------+   |              +------------+
| Unit 5     |---+              |  S3        |
| Outfit Rec |                  | (Images)   |
+------------+                  +------------+
     |
     +--> Unit 2, 3, 4 のデータを集約
```

## クリティカルパス

1. **Unit 1 (Auth)** が最優先 — 全ユニットが認証に依存
2. **Unit 2 (Closet)** が次に優先 — Unit 5 の入力データとして必須
3. **Unit 3, 4** は並行可能 — フォールバック手段があるため非ブロッキング
4. **Unit 5 (Outfit)** は最後 — 他の全ユニットのデータを集約

## 共有リソース

| リソース | 使用ユニット | 管理方式 |
|---------|------------|---------|
| DynamoDB Client | 全ユニット | `backend/app/shared/db.py` |
| S3 Client | Unit 2 | `backend/app/shared/s3.py` |
| Bedrock Client | Unit 2, 5 | `backend/app/shared/bedrock.py` |
| Auth Middleware | 全ユニット（Unit 1除く） | `backend/app/shared/auth_middleware.py` |
