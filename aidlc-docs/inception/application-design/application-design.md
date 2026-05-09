# Application Design — 統合ドキュメント

## 1. アーキテクチャ概要

「今日の人格、着せておきました。」は、予定・天気・TPO・手持ち服をもとに、AIが"今日どう見られるべきか"を定義し、それを服装として提案するサービスである。BFF（Backend for Frontend）パターンを採用した3層Webアーキテクチャで構成される。

### サービス概要

ユーザーは手持ち服を登録し、当日の予定はGoogle Calendarから自動取得される（手動での追加・編集も可能）。AIは天気、気温、降水確率、予定内容、移動量、過去の着用履歴を統合し、「どう見られるべきか」も含めた“今日の人格”を自動で生成する。

たとえば顧客訪問なら「信頼感のある人」、展示会なら「動ける人」、デートなら「やわらかく見える人」として、手持ち服から服装を決定し、選定理由・除外理由も提示する。

### 設計思想

狙いは、服選びを便利にすることではなく、**朝の服選び・TPO判断・自己演出までAIに任せきる"着るだけの怠惰"を実現すること**である。AIは予定・天気・手持ち服をもとに"今日どう見られるべきか"を定義し、それを服装として出力する。ここでいう「今日の人格」とは性格診断ではなく、予定・TPOからAIが定義する外向きの印象設計である。

MVPでは、手持ち服登録、予定・天気取得、人格定義、服装提案、選定理由・除外理由の表示、着用履歴管理を実装予定。アーキテクチャは、ユーザーの意思決定を最小化し、AIが文脈を統合して「今日のあなた」を決定するフローを最短で実現することを重視する。

> テーマ適合性：**「使うほど、自分で今日の自分を決めなくなる」**

ユーザーストーリーの詳細は [unit-of-work-story-map.md](unit-of-work-story-map.md) を参照。

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
- **Daily Context Module**: 日次コンテキスト取得UI（Calendar連携含む）
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

> ユーザーの操作は「提案して」の1タップのみ。以降のすべての判断はAIが行う。（US-004, US-005, US-006）

```
1. ユーザーが「今日のコーデを提案して」をリクエスト
2. BFF → Core API (Outfit Recommendation Service)
3. Outfit Recommendation Service が並行取得:
   a. Weather Service → 天気情報（キャッシュor API）    ← US-003: 天気確認を放棄
   b. Calendar Service → 今日の予定（Google Calendar）  ← US-002: 予定確認を放棄
   c. Closet Service → 手持ち服一覧                     ← US-001: 服管理を放棄
   d. Wear History Service → 直近着用履歴               ← US-007: 着用記憶を放棄
4. コンテキスト統合 → Bedrockプロンプト構築
5. Bedrock API呼び出し → 「今日の人格」＋コーデ生成       ← US-004: 自己演出を放棄
6. レスポンス整形（人格定義+推奨アイテム+選定理由+除外理由）  ← US-006: 判断根拠の思考を放棄
7. ユーザーに提案表示
8. (オプション) 「別の提案」→ 前回除外で再生成            ← US-008: やり直しの思考を放棄
9. 「採用」→ 着用履歴記録
```

---

## 5. 詳細ドキュメント参照

- コンポーネント定義: [components.md](components.md)
- メソッドシグネチャ: [component-methods.md](component-methods.md)
- サービス定義: [services.md](services.md)
- 依存関係: [component-dependency.md](component-dependency.md)
