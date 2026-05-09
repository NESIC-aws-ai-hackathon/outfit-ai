# Services Definition

## Service Architecture Overview

本サービスはBFF（Backend for Frontend）パターンを採用し、3層構造で構成される：

```
+------------------+     +------------------+     +------------------+
|   Client         |     |   BFF Layer      |     |   Core API       |
|   (Browser)      | --> |   (Next.js API)  | --> |   (FastAPI)      |
|                  |     |                  |     |                  |
|   React UI       |     |   Session Mgmt   |     |   Business Logic |
|   Components     |     |   Request Proxy  |     |   Data Access    |
|                  |     |   Auth Guard     |     |   External APIs  |
+------------------+     +------------------+     +------------------+
```

---

## 1. Authentication Service

### 責務
- ユーザー登録・ログイン・ログアウト
- JWT発行・検証
- パスワードハッシュ化・リセット
- Google OAuth 2.0トークン管理（Calendar連携用）

### オーケストレーション
```
Client → BFF(Auth Route) → Core API(Auth Service) → DynamoDB(Users Table)
                                                   → JWT Token Generation
```

### 他サービスとの関係
- すべてのサービスが認証ミドルウェアを経由
- Calendar Serviceと連携（Googleトークン保管）

---

## 2. Closet Management Service

### 責務
- 服アイテムのCRUD操作
- S3への画像アップロード管理（プリサインドURL生成）
- AIタグ付けオーケストレーション（同期/非同期）

### オーケストレーション
#### 服登録フロー（画像あり）
```
Client → BFF → Core API(Closet Service) → S3(Presigned URL生成)
         |                                  |
         | ← presigned URL                  |
         |                                  |
Client → S3(直接アップロード)                |
         |                                  |
Client → BFF → Core API(Closet Service)    |
                  |                         |
                  +-→ Bedrock(同期: 基本タグ) → Response(色・種別)
                  |                         
                  +-→ Bedrock(非同期: 詳細タグ) → DynamoDB(後から更新)
                  |
                  +-→ DynamoDB(アイテム保存)
```

### 他サービスとの関係
- Outfit Recommendation Serviceが服データを参照
- Wear History Serviceが服IDを参照

---

## 3. Calendar Service

### 責務
- Google Calendar OAuth 2.0認可フロー
- リフレッシュトークンの安全な保管と自動リフレッシュ
- 指定日のカレンダー予定取得と整形

### オーケストレーション
#### OAuth連携フロー
```
Client → Google OAuth Consent → BFF(Auth Code受取)
  → Core API(Calendar Service) → Google Token Exchange
  → DynamoDB(トークン保存)
```

#### 予定取得フロー
```
Client → BFF → Core API(Calendar Service)
                  |
                  +-→ DynamoDB(トークン取得)
                  +-→ Google Calendar API(予定取得)
                  +-→ Response(整形された予定リスト)
```

### 他サービスとの関係
- Outfit Recommendation Serviceが予定データを入力として使用

---

## 4. Weather Service

### 責務
- ユーザー位置情報に基づく天気情報取得
- 15分短期キャッシュ管理
- 天気データの服装判断向け整形

### オーケストレーション
```
Client → BFF → Core API(Weather Service)
                  |
                  +-→ Cache Check(DynamoDB/メモリ, 15min TTL)
                  |     |
                  |     +-→ HIT: キャッシュ返却
                  |     +-→ MISS: OpenWeatherMap API呼び出し
                  |               → キャッシュ保存
                  |               → Response
```

### 他サービスとの関係
- Outfit Recommendation Serviceが天気データを入力として使用

---

## 5. Outfit Recommendation Service (コアサービス)

### 責務
- 全コンテキストの統合（天気+予定+クローゼット+着用履歴）
- 「今日の人格」定義の生成
- Bedrockプロンプト構築と呼び出し
- 提案結果の整形（アイテム詳細+理由+代替案）
- 再生成対応（前回提案を除外した新提案）

### オーケストレーション
#### 提案生成フロー
```
Client → BFF → Core API(Outfit Recommendation Service)
                  |
                  +-→ Weather Service(天気取得)
                  +-→ Calendar Service(予定取得) or リクエストパラメータ
                  +-→ Closet Service(手持ち服取得)
                  +-→ Wear History Service(直近着用取得)
                  |
                  +-→ Context統合 → プロンプト構築
                  +-→ Bedrock API呼び出し
                  +-→ レスポンスパース
                  +-→ Response(人格+コーデ+理由+代替案)
```

#### 再生成フロー
```
Client(「別の提案」) → BFF → Core API(Outfit Recommendation Service)
                              |
                              +-→ 前回提案を除外条件に追加
                              +-→ 同上フロー（除外付きプロンプト）
```

### 他サービスとの関係
- Weather Service、Calendar Service、Closet Service、Wear History Serviceすべてを集約

---

## 6. Wear History Service

### 責務
- 着用記録の保存（ユーザーが提案を「採用」した時点）
- 直近着用アイテムの取得（連続着用回避用）

### オーケストレーション
```
Client(「この提案を採用」) → BFF → Core API(Wear History Service)
                                     |
                                     +-→ DynamoDB(着用記録保存)
```

### 他サービスとの関係
- Outfit Recommendation Serviceが直近着用データを参照
