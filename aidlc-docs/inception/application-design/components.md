# Components Definition

## 1. Frontend Application (Next.js)

### 1.1 Auth Module
- **責務**: ユーザー認証UI（登録・ログイン・ログアウト・パスワードリセット）
- **インターフェース**: Next.js Pages/Components + API Routes (BFF)
- **備考**: メール+パスワード認証、Google OAuth連携（Calendar用）

### 1.2 Closet Module
- **責務**: クローゼット管理UI（服の登録・編集・削除・一覧表示）
- **インターフェース**: CRUD画面 + 画像アップロードUI
- **備考**: 写真アップロード時にAIタグ補助を同期/非同期で処理

### 1.3 Daily Input Module
- **責務**: 日次コンテキスト入力UI（予定入力・Google Calendar表示）
- **インターフェース**: フォームUI + カレンダー連携表示
- **備考**: Google Calendar APIからの予定自動取得と手動入力の両対応

### 1.4 Outfit Recommendation Module
- **責務**: AI服装提案の表示UI（今日の人格・推奨コーデ・提案理由・代替案・再生成）
- **インターフェース**: 提案結果画面 + 「別の提案」ボタン
- **備考**: インタラクティブな再生成機能を含む

### 1.5 BFF API Routes (Next.js API Routes)
- **責務**: フロントエンド用ゲートウェイ。クライアントからのリクエストを受け、FastAPI Core APIへ中継
- **インターフェース**: REST API (Next.js API Routes)
- **備考**: セッション管理、リクエスト変換、レスポンス整形を担当

---

## 2. Backend Core API (FastAPI)

### 2.1 Auth Service Component
- **責務**: ユーザー認証・認可のビジネスロジック（登録、ログイン、トークン管理）
- **インターフェース**: REST API endpoints
- **備考**: JWT発行、パスワードハッシュ化、セッション管理

### 2.2 Closet Service Component
- **責務**: クローゼットデータの管理（CRUD操作、画像管理、AIタグ付けオーケストレーション）
- **インターフェース**: REST API endpoints
- **備考**: S3への画像アップロード、Bedrock呼び出しによるタグ付け

### 2.3 Calendar Service Component
- **責務**: Google Calendar APIとの連携（OAuth管理、予定取得）
- **インターフェース**: REST API endpoints + Google Calendar API client
- **備考**: OAuth 2.0読み取り専用スコープ、リフレッシュトークン管理

### 2.4 Weather Service Component
- **責務**: 天気情報の取得とキャッシュ管理
- **インターフェース**: REST API endpoints + OpenWeatherMap API client
- **備考**: 15分短期キャッシュ、ユーザー位置情報ベース

### 2.5 Outfit Recommendation Service Component
- **責務**: AI服装提案のコアロジック（コンテキスト統合、Bedrock呼び出し、結果整形）
- **インターフェース**: REST API endpoints
- **備考**: 天気+予定+クローゼット+着用履歴を統合してプロンプト生成、再生成対応

### 2.6 Wear History Service Component
- **責務**: 着用履歴の記録・取得（直前着用の追跡）
- **インターフェース**: REST API endpoints
- **備考**: 簡易版 — 直前の着用記録による連続着用回避

---

## 3. External Integrations

### 3.1 Amazon Bedrock (Claude) Integration
- **責務**: LLM呼び出し（服装提案生成、画像AIタグ付け）
- **インターフェース**: AWS SDK (boto3) → Bedrock API
- **備考**: 2つの用途: (1) 服装提案プロンプト処理、(2) 服画像のタグ推定

### 3.2 OpenWeatherMap Integration
- **責務**: 天気情報の外部API取得
- **インターフェース**: HTTP client → OpenWeatherMap REST API
- **備考**: 無料枠利用、日本の都市対応

### 3.3 Google Calendar Integration
- **責務**: ユーザーの予定情報取得
- **インターフェース**: Google Calendar API client (OAuth 2.0)
- **備考**: 読み取り専用スコープ

---

## 4. Data Store

### 4.1 Amazon DynamoDB
- **責務**: 全データの永続化（ユーザー、服、予定、着用履歴、天気キャッシュ）
- **インターフェース**: AWS SDK (boto3) → DynamoDB API
- **備考**: シングルテーブル or マルチテーブル設計

### 4.2 Amazon S3
- **責務**: 服の画像ファイルの保存
- **インターフェース**: AWS SDK (boto3) → S3 API
- **備考**: プリサインドURL経由でフロントエンドから直接アップロード/取得
