# 今日の人格、着せておきました。

天気・予定・TPO・手持ち服から、AIが"今日のあなた"を服として決めるサービス

## コンセプト

毎朝の服選びをAIに任せることで、その日の予定や天気に合った服装を自動で提案します。

ユーザーは手持ち服を登録するだけ。AIが天気、気温、予定内容、TPO、着用履歴をもとに「今日の人格」を定義し、それを服装として出力します。

- 顧客訪問の日 → 信頼される人
- 展示会の日 → 動ける人
- デートの日 → やわらかく見える人
- 在宅勤務の日 → 無理せず整っている人

## ハッカソンテーマ：「人をダメにする」

このサービスの狙いは、服選びを便利にすることではありません。
**朝の服選び・TPO判断・自己演出までAIに任せきる"着るだけの怠惰"を実現すること**です。

使うほど、ユーザーは以下を自分で決めなくなります：

| 放棄される意思決定 | AIがやること |
|------------------|------------|
| 服の組み合わせを考える | 手持ち服からAIが自動で選ぶ |
| 天気を確認して服を変える | 天気情報を自動取得し反映する |
| 予定に合う服を判断する | 予定からTPOを自動判定する |
| 今日どう見られたいか考える | 「今日の人格」をAIが定義する |
| 着たものを覚えておく | 着用履歴をAIが管理する |

> **「使うほど、自分で今日の自分を決めなくなる」**

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js, React, Tailwind CSS |
| BFF | Next.js API Routes |
| バックエンド | Python, FastAPI |
| データベース | Amazon DynamoDB |
| 画像ストレージ | Amazon S3 |
| AI | Amazon Bedrock (Claude) |
| 天気API | OpenWeatherMap API |
| カレンダー | Google Calendar API (OAuth 2.0) |
| インフラ | AWS (Lambda / API Gateway / S3 / CloudFront) |

## プロジェクト構成

```
├── frontend/          # Next.js (BFF + React UI)
│   └── src/
│       ├── app/       # App Router (ページ + API Routes)
│       ├── components/
│       ├── lib/
│       └── types/
├── backend/           # FastAPI Core API
│   └── app/
│       ├── auth/      # 認証
│       ├── closet/    # クローゼット管理
│       ├── calendar/  # Google Calendar連携
│       ├── weather/   # 天気情報
│       ├── outfit/    # AI服装提案
│       ├── history/   # 着用履歴
│       └── shared/    # 共通モジュール
└── aidlc-docs/        # 設計ドキュメント
```

## 主な機能

1. **ユーザー認証** — メール+パスワード登録・ログイン
2. **クローゼット管理** — 手持ち服の登録（写真アップロード + AIタグ補助）
3. **日次コンテキスト取得** — Google Calendarから予定を自動取得 + 手動での追加・編集も可能
4. **天気情報取得** — 位置情報ベースの天気自動取得（15分キャッシュ）
5. **AI服装提案** — 「今日の人格」定義 + コーデ提案 + 再生成機能
6. **着用履歴** — 直前の着用を記録し連続着用を回避

## セットアップ

### 前提条件

- Node.js 18+
- Python 3.11+
- AWS アカウント
- Google Cloud Console プロジェクト（Calendar API用）
- OpenWeatherMap API キー

### フロントエンド

```bash
cd frontend
npm install
npm run dev
```

### バックエンド

```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload
```

## 環境変数

### Frontend (.env.local)

```
NEXT_PUBLIC_API_URL=http://localhost:8000
GOOGLE_CLIENT_ID=<your-google-client-id>
GOOGLE_CLIENT_SECRET=<your-google-client-secret>
```

### Backend (.env)

```
AWS_REGION=ap-northeast-1
DYNAMODB_TABLE_PREFIX=outfit-ai
S3_BUCKET_NAME=<your-s3-bucket>
OPENWEATHERMAP_API_KEY=<your-api-key>
GOOGLE_CLIENT_ID=<your-google-client-id>
GOOGLE_CLIENT_SECRET=<your-google-client-secret>
JWT_SECRET_KEY=<your-secret-key>
BEDROCK_MODEL_ID=anthropic.claude-3-sonnet-20240229-v1:0
```

## ライセンス

MIT
