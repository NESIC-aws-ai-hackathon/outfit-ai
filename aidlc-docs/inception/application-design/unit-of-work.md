# Unit of Work Definition

## 分割方針
機能単位の論理分割を採用する。各ユニットはマイクロサービスとして物理分離するのではなく、責務・入力・出力・依存関係を明確にするための論理ユニットとして定義する。

デプロイは1つのFrontend（Next.js）と1つのBackend（FastAPI）としてまとめるが、開発・設計・テストは各ユニット単位で進行する。

## コード構成

```
<workspace-root>/
+-- frontend/                    # Next.js (BFF + React UI)
|   +-- src/
|   |   +-- app/                 # Next.js App Router
|   |   |   +-- (auth)/         # Auth関連ページ
|   |   |   +-- closet/         # クローゼット管理ページ
|   |   |   +-- outfit/         # 提案表示ページ
|   |   |   +-- api/            # BFF API Routes
|   |   |       +-- auth/
|   |   |       +-- closet/
|   |   |       +-- calendar/
|   |   |       +-- weather/
|   |   |       +-- outfit/
|   |   +-- components/          # 共通UIコンポーネント
|   |   +-- lib/                 # ユーティリティ・API Client
|   |   +-- types/               # TypeScript型定義
|   +-- public/
|   +-- package.json
|   +-- next.config.js
|   +-- tailwind.config.js
|
+-- backend/                     # FastAPI Core API
|   +-- app/
|   |   +-- main.py              # FastAPIアプリケーションエントリ
|   |   +-- config.py            # 設定管理
|   |   +-- auth/                # Unit 1: Auth
|   |   |   +-- router.py
|   |   |   +-- service.py
|   |   |   +-- models.py
|   |   |   +-- schemas.py
|   |   +-- closet/              # Unit 2: Closet
|   |   |   +-- router.py
|   |   |   +-- service.py
|   |   |   +-- models.py
|   |   |   +-- schemas.py
|   |   +-- calendar/            # Unit 3: Calendar
|   |   |   +-- router.py
|   |   |   +-- service.py
|   |   |   +-- models.py
|   |   |   +-- schemas.py
|   |   +-- weather/             # Unit 4: Weather
|   |   |   +-- router.py
|   |   |   +-- service.py
|   |   |   +-- models.py
|   |   |   +-- schemas.py
|   |   +-- outfit/              # Unit 5: Outfit Recommendation
|   |   |   +-- router.py
|   |   |   +-- service.py
|   |   |   +-- models.py
|   |   |   +-- schemas.py
|   |   |   +-- prompts.py
|   |   +-- history/             # Unit 5に含む: Wear History
|   |   |   +-- router.py
|   |   |   +-- service.py
|   |   |   +-- models.py
|   |   +-- shared/              # 共通モジュール
|   |       +-- db.py            # DynamoDBクライアント
|   |       +-- s3.py            # S3クライアント
|   |       +-- bedrock.py       # Bedrockクライアント
|   |       +-- auth_middleware.py
|   +-- requirements.txt
|   +-- Dockerfile
|
+-- aidlc-docs/                  # AI-DLCドキュメント（コード外）
+-- .gitignore
+-- README.md
```

---

## Unit Definitions

### Unit 1: Auth（認証）
- **責務**: ユーザー登録・ログイン・ログアウト・パスワードリセット・JWT管理
- **Frontend範囲**: 認証ページ（登録・ログイン）+ BFF Auth API Routes
- **Backend範囲**: `backend/app/auth/` — Auth Service全体
- **データ**: Users テーブル（DynamoDB）
- **外部依存**: なし
- **優先度**: 最高（他ユニットの前提）

### Unit 2: Closet（クローゼット管理）
- **責務**: 服アイテムのCRUD、画像アップロード、AIタグ付け（同期/非同期）
- **Frontend範囲**: クローゼット管理ページ + BFF Closet API Routes
- **Backend範囲**: `backend/app/closet/` — Closet Service全体
- **データ**: ClothingItems テーブル（DynamoDB）、S3（画像）
- **外部依存**: Amazon Bedrock（画像タグ付け）、Amazon S3
- **優先度**: 高（提案の入力データ）

### Unit 3: Calendar（カレンダー連携）
- **責務**: Google Calendar OAuth連携、予定取得、予定データ整形
- **Frontend範囲**: カレンダー連携UI + BFF Calendar API Routes
- **Backend範囲**: `backend/app/calendar/` — Calendar Service全体
- **データ**: OAuthTokens テーブル（DynamoDB）
- **外部依存**: Google Calendar API、Google OAuth 2.0
- **優先度**: 中（手動入力でフォールバック可）

### Unit 4: Weather（天気情報）
- **責務**: 天気情報取得、15分キャッシュ管理、位置情報管理
- **Frontend範囲**: 天気表示UI + BFF Weather API Routes
- **Backend範囲**: `backend/app/weather/` — Weather Service全体
- **データ**: WeatherCache テーブル（DynamoDB）
- **外部依存**: OpenWeatherMap API
- **優先度**: 中（天気なし提案モードでフォールバック可）

### Unit 5: Outfit Recommendation（服装提案）
- **責務**: コンテキスト統合、AI提案生成、再生成、着用履歴記録
- **Frontend範囲**: 提案表示・再生成UI + BFF Outfit API Routes
- **Backend範囲**: `backend/app/outfit/` + `backend/app/history/`
- **データ**: Recommendations テーブル、WearHistory テーブル（DynamoDB）
- **外部依存**: Amazon Bedrock（Claude）
- **優先度**: 最高（コアバリュー）

---

## 開発順序

並行開発を基本とし、API仕様を先に合意する。

```
Phase 0: API仕様合意（全ユニット共通）
   |
   +-- Phase 1（並行）:
   |     +-- Unit 1: Auth（FE + BE）
   |     +-- Unit 2: Closet（FE + BE）
   |
   +-- Phase 2（並行、Unit 1完了後）:
   |     +-- Unit 3: Calendar（FE + BE）
   |     +-- Unit 4: Weather（FE + BE）
   |
   +-- Phase 3（Unit 1-4完了後）:
   |     +-- Unit 5: Outfit Recommendation（FE + BE）
   |
   +-- Phase 4: 統合テスト・デモ準備
```
