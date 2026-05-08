# Requirements Verification Questions

本プロジェクト「今日の人格、着せておきました。」の要件を明確にするために、以下の質問にお答えください。
各質問の `[Answer]:` タグの後に、選択肢の文字（A, B, C, D, E, X）を記入してください。

---

## Question 1
このプロジェクトのMVP（最小限の実用的プロダクト）としてのプラットフォームは何ですか？

A) Webアプリケーション（レスポンシブ対応）
B) モバイルアプリ（iOS / Android）
C) Webアプリケーション＋モバイルアプリの両方
D) LINE Bot / チャットボット形式
X) Other (please describe after [Answer]: tag below)

[Answer]: Ａ

---

## Question 2
ユーザー認証の方式はどれを想定していますか？

A) メールアドレス＋パスワード
B) Googleアカウントなどのソーシャルログイン
C) LINEログイン
D) 認証なし（ローカルのみ、プロトタイプ段階）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ａ

---

## Question 3
手持ち服の登録方法として、MVP段階で優先すべき方法はどれですか？

A) 手動入力のみ（アイテム名・種別・色などをフォーム入力）
B) 写真アップロード＋AI自動タグ付け（画像解析）
C) 手動入力を基本＋写真アップロード補助（構想テキストの通り）
D) ECサイトの購入履歴との連携
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｃ

---

## Question 4
天気情報の取得に使用するAPIはどれを想定していますか？

A) OpenWeatherMap API（無料枠あり）
B) WeatherAPI（無料枠あり）
C) 気象庁API（日本の公式データ）
D) 特に指定なし（おすすめを提案してほしい）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｄ

---

## Question 5
AIによるコーデ提案に使用するLLM/AIサービスはどれを想定していますか？

A) Amazon Bedrock（Claude等）
B) OpenAI API（GPT-4等）
C) Google Gemini API
D) ローカルLLM
E) 特に指定なし（おすすめを提案してほしい）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ａ

---

## Question 6
バックエンドの技術スタックについて希望はありますか？

A) Python（FastAPI / Django）
B) Node.js（Express / NestJS）
C) Java / Kotlin（Spring Boot）
D) Go
E) 特に指定なし（おすすめを提案してほしい）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｅ

---

## Question 7
フロントエンドの技術スタックについて希望はありますか？

A) React（Next.js）
B) Vue.js（Nuxt.js）
C) Svelte（SvelteKit）
D) 特に指定なし（おすすめを提案してほしい）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｄ

---

## Question 8
データベースの選択について希望はありますか？

A) リレーショナルDB（PostgreSQL）
B) リレーショナルDB（MySQL）
C) NoSQL（DynamoDB）
D) NoSQL（MongoDB）
E) 特に指定なし（おすすめを提案してほしい）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｅ

---

## Question 9
デプロイ先のインフラについて希望はありますか？

A) AWS（EC2, ECS, Lambda等）
B) Vercel + その他クラウド
C) Google Cloud Platform
D) ローカル実行のみ（ハッカソンデモ用）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ａ

---

## Question 10
カレンダー連携はMVP段階で必要ですか？

A) はい — Google Calendar連携を最初から実装する
B) いいえ — MVPでは手動入力のみ、カレンダー連携は将来対応
C) はい — ただしGoogle Calendar以外（Outlook等）も対応したい
X) Other (please describe after [Answer]: tag below)

[Answer]: Ａ

---

## Question 11
着用履歴の追跡と「同じ服を避ける」機能はMVPに含めますか？

A) はい — 提案した服の着用記録を保存し、直近の重複を避ける
B) 簡易版 — 直前の着用だけ記録し、連続着用を避ける程度
C) いいえ — MVP段階では不要
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｂ

---

## Question 12
ハッカソンの期間・締め切りはいつですか？MVP完成の目標時期を教えてください。

A) 1日（本日中にデモ可能なレベル）
B) 2〜3日
C) 1週間程度
D) 2週間以上
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｂ

---

## Question 13: Security Extensions
本プロジェクトでセキュリティ拡張ルールを適用しますか？

A) はい — すべてのセキュリティルールをブロッキング制約として適用する（本番グレードのアプリケーション向け推奨）
B) いいえ — セキュリティルールをスキップする（PoC、プロトタイプ、実験的プロジェクト向け）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｂ

---

## Question 14: Property-Based Testing Extension
本プロジェクトでプロパティベーステスト（PBT）ルールを適用しますか？

A) はい — すべてのPBTルールをブロッキング制約として適用する（ビジネスロジック、データ変換、シリアライゼーション、ステートフルコンポーネントを含むプロジェクト向け推奨）
B) 部分的 — 純粋関数とシリアライゼーションのラウンドトリップにのみPBTルールを適用する
C) いいえ — PBTルールをスキップする（シンプルなCRUDアプリ、UIのみのプロジェクト向け）
X) Other (please describe after [Answer]: tag below)

[Answer]: Ｂ
