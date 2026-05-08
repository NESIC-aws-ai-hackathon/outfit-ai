# Application Design — 統合ドキュメント

## 1. アーキテクチャ概要

「今日の人格、着せておきました。」は、BFF（Backend for Frontend）パターンを採用した3層Webアーキテクチャで構成される。

```
Browser (React/Next.js) → BFF (Next.js API Routes) → Core API (FastAPI) → AWS Services + External APIs
```

### 設計判断サマリ
| 判断項目 | 選択 | 理由 |
|---------|------|------|
| FE/BE分離方式 | BFF パターン | セッション管理をBFFに集約、Core APIは純粋なビジネスロジック |
| 画像AIタグ付け | ハイブリッド（同期+非同期） | 基本タグは即時表示、詳細タグはバックグラウンド |
| AI提案レスポンス | インタラクティブ（再生成可） | ユーザー体験の向上、複数提案の比較 |
| Calendar認証 | OAuth 2.0 読み取り専用 | MVP段階での最小権限 |
| 天気キャッシュ | オンリクエスト+15分キャッシュ | 鮮度とAPI呼び出し削減のバランス |
| 位置情報 | デフォルト設定+Geolocation上書き | 日常使い（固定）と外出先（動的）の両対応 |

---

## 2. コンポーネント構成

### Frontend (Next.js)
- **Auth Module**: 認証UI
- **Closet Module**: クローゼット管理UI
- **Daily Input Module**: 日次コンテキスト入力UI（Calendar連携含む）
- **Outfit Recommendation Module**: 提案表示・再生成UI
- **BFF API Routes**: フロントエンド用ゲートウェイ

### Backend Core API (FastAPI)
- **Auth Service**: 認証・認可ロジック（JWT）
- **Closet Service**: 服データCRUD + AIタグオーケストレーション
- **Calendar Service**: Google Calendar OAuth + 予定取得
- **Weather Service**: 天気取得 + 15分キャッシュ
- **Outfit Recommendation Service**: コンテキスト統合 + Bedrock呼び出し + 提案生成
- **Wear History Service**: 簡易着用履歴

### External Integrations
- **Amazon Bedrock (Claude)**: 服装提案 + 画像タグ付け
- **OpenWeatherMap API**: 天気情報
- **Google Calendar API**: 予定取得

### Data Stores
- **Amazon DynamoDB**: 全データ永続化
- **Amazon S3**: 服画像保存

---

## 3. サービス間通信

| 通信経路 | プロトコル | 認証方式 |
|---------|----------|---------|
| Browser ↔ BFF | HTTPS/JSON | HttpOnly Cookie |
| BFF ↔ Core API | HTTPS/JSON | Internal JWT/API Key |
| Core API ↔ AWS Services | AWS SDK | IAM Role |
| Core API ↔ OpenWeatherMap | HTTPS/JSON | API Key |
| Core API ↔ Google Calendar | HTTPS/JSON | OAuth 2.0 Bearer |

---

## 4. コアフロー: 服装提案

```
1. ユーザーが「今日のコーデを提案して」をリクエスト
2. BFF → Core API (Outfit Recommendation Service)
3. Outfit Recommendation Service が並行取得:
   a. Weather Service → 天気情報（キャッシュor API）
   b. Calendar Service → 今日の予定（Google Calendar）
   c. Closet Service → 手持ち服一覧
   d. Wear History Service → 直近着用履歴
4. コンテキスト統合 → Bedrockプロンプト構築
5. Bedrock API呼び出し → 「今日の人格」＋コーデ生成
6. レスポンス整形（人格定義+推奨アイテム+理由+代替案）
7. ユーザーに提案表示
8. (オプション) 「別の提案」→ 前回除外で再生成
9. 「採用」→ 着用履歴記録
```

---

## 5. 詳細ドキュメント参照

- コンポーネント定義: [components.md](components.md)
- メソッドシグネチャ: [component-methods.md](component-methods.md)
- サービス定義: [services.md](services.md)
- 依存関係: [component-dependency.md](component-dependency.md)
