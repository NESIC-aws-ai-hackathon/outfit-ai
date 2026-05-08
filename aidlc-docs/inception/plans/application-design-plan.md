# Application Design Plan

## 概要
「今日の人格、着せておきました。」のアプリケーション設計計画です。
以下の質問に回答し、設計の方向性を確定させてください。

---

## Design Questions

### Component Identification

#### Question 1
フロントエンドとバックエンドの分離方式について、どのアプローチを希望しますか？

A) 完全分離 — Next.jsフロントエンド（SSG/CSR）+ FastAPIバックエンドを独立デプロイ
B) BFF（Backend for Frontend）パターン — Next.js API Routesをフロントエンド用ゲートウェイにし、FastAPIをコアAPIとして使用
C) モノリシック — Next.jsのAPI Routesで全ロジックを実装（FastAPIなし）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

#### Question 2
画像（服の写真）のアップロードとAIタグ付けの処理方式はどれが良いですか？

A) 同期処理 — アップロード時に即座にAIタグ付けを実行し、結果を返す
B) 非同期処理 — アップロード後にバックグラウンドでAIタグ付けを実行し、後から結果を反映
C) ハイブリッド — 簡易タグ（色・種別）は同期、詳細タグは非同期
X) Other (please describe after [Answer]: tag below)

[Answer]:  Ｃ

アップロード直後に色・服種などの基本タグを同期的に付与し、季節感・素材感・TPO適性・フォーマル度などの詳細タグは非同期でバックグラウンド処理する。
これにより、ユーザーには即時性のある体験を提供しつつ、AI処理の遅延や失敗時にも再実行しやすい構成にする。

---

### Service Layer Design

#### Question 3
AI服装提案のAPIレスポンス形式について、どのような出力が望ましいですか？

A) シンプル — 「今日の人格」テキスト＋推奨アイテムIDのリスト
B) リッチ — 「今日の人格」テキスト＋推奨アイテム詳細＋提案理由＋代替案1つ
C) インタラクティブ — 提案後にユーザーが「別の提案」をリクエストできる再生成機能付き
X) Other (please describe after [Answer]: tag below)

[Answer]: C

APIレスポンスは、今日の人格テキスト、推奨アイテム詳細、提案理由、代替案を含むリッチ形式とし、さらにユーザーが「別の提案」をリクエストできる再生成機能を持たせる。
予定・天気・手持ち服・過去の選好をもとに、複数回の提案改善ができる体験を目指す。

---

#### Question 4
Google Calendar連携の認証フローについて、どの方式を想定していますか？

A) OAuth 2.0標準フロー — ユーザーが初回にGoogleアカウントを認可し、リフレッシュトークンで継続利用
B) サービスアカウント — 管理者がサービスアカウントで全ユーザーのカレンダーにアクセス
C) MVP段階ではOAuth 2.0を実装し、カレンダー読み取り専用スコープのみ
X) Other (please describe after [Answer]: tag below)

[Answer]: C

MVP段階ではOAuth 2.0を実装し、Google Calendarの読み取り専用スコープに限定する。
ユーザーの予定情報は服装提案に必要な範囲でのみ参照し、書き込み権限は要求しない。

---

### Component Dependencies

#### Question 5
天気情報の取得タイミングとキャッシュ戦略はどれが適切ですか？

A) リアルタイム取得 — ユーザーがコーデ提案を要求するたびにAPIを呼ぶ
B) 定期キャッシュ — 1時間ごとに天気情報をキャッシュし、キャッシュから提供
C) ユーザーリクエスト時に取得＋短期キャッシュ（15分）— 同じユーザーの連続リクエストにはキャッシュ利用
X) Other (please describe after [Answer]: tag below)

[Answer]: C

ユーザーがコーディネート提案を要求したタイミングで天気情報を取得し、同一ユーザー・同一地点の連続リクエストでは15分程度の短期キャッシュを利用する。
天気の鮮度を保ちながら、API呼び出し回数とレスポンス遅延を抑える。

---

### Design Patterns

#### Question 6
ユーザーの位置情報（天気取得用）の管理方法はどうしますか？

A) ユーザー登録時に都市名/郵便番号を設定し、固定で使用
B) ブラウザのGeolocation APIで毎回自動取得
C) 登録時にデフォルト設定＋ブラウザ位置情報で上書きオプション
X) Other (please describe after [Answer]: tag below)

[Answer]: C

ユーザー登録時にデフォルトの都市または郵便番号を設定し、必要に応じてブラウザのGeolocation APIで現在地に上書きできるようにする。
普段使いでは固定地点を利用し、外出先や旅行先では現在地ベースの天気提案に切り替えられる構成にする。

---

## Design Plan Checklist

上記の質問回答後、以下の成果物を生成します：

- [x] Generate components.md — コンポーネント定義と責務
- [x] Generate component-methods.md — メソッドシグネチャとI/O定義
- [x] Generate services.md — サービス定義とオーケストレーション
- [x] Generate component-dependency.md — 依存関係と通信パターン
- [x] Validate design completeness and consistency
