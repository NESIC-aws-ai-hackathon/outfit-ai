# Unit of Work Plan

## 概要
アプリケーション設計に基づき、システムを開発可能なユニット（Unit of Work）に分解します。

---

## Questions

### Question 1
ユニットの分割粒度について、どのアプローチが適切ですか？

A) 2ユニット — Frontend（Next.js BFF含む）と Backend（FastAPI Core API）の2分割
B) 3ユニット — Frontend、Backend API、Infrastructure（CDK/CloudFormation）の3分割
C) 機能単位 — Auth, Closet, Calendar, Weather, Outfit Recommendation をそれぞれ独立ユニットに
X) Other (please describe after [Answer]: tag below)

[Answer]: C

Auth, Closet, Calendar, Weather, Outfit Recommendation を機能単位のユニットとして分割する。
ただし、これはマイクロサービスとして物理的に完全分離する意味ではなく、Unit of Workとして責務・入力・出力・依存関係を明確にするための論理分割とする。

---

### Question 2
フロントエンドとバックエンドの開発順序について、どの戦略が適切ですか？

A) バックエンドファースト — Core APIを先に完成させ、その後フロントエンドを接続
B) フロントエンドファースト — モックデータでUI/UXを先に構築し、その後APIを接続
C) 並行開発 — インターフェース（API仕様）を先に合意し、FE/BEを同時進行
X) Other (please describe after [Answer]: tag below)

[Answer]: C

最初にAPI仕様・データモデル・主要I/Oを合意し、FrontendとBackendを並行開発する。
Frontendはモックデータで画面を先行実装し、Backendは同じAPI契約に沿って実装することで、ハッカソン期間内でもデモ完成度を高める。

---

### Question 3
コードのディレクトリ構成について、どの方式を希望しますか？

A) モノレポ — 1つのリポジトリに `frontend/` と `backend/` のディレクトリを配置
B) マルチレポ — フロントエンドとバックエンドを別リポジトリで管理
C) モノレポ（Turborepo/nx等のツール使用） — ビルドシステム統合付き
X) Other (please describe after [Answer]: tag below)

[Answer]: A

1つのリポジトリ内に frontend/ と backend/ を配置するモノレポ構成とする。
ハッカソンでは環境構築・共有・デプロイ・レビューを簡単にすることを優先し、Turborepo/nxなどの追加ツールは使わない。

---

## Plan Checklist

回答後、以下の成果物を生成します：

- [x] Generate unit-of-work.md — ユニット定義と責務
- [x] Generate unit-of-work-dependency.md — ユニット間依存関係
- [x] Generate unit-of-work-story-map.md — 機能要件とユニットのマッピング
- [x] Document code organization strategy
- [x] Validate unit boundaries and dependencies
- [ ] Generate unit-of-work-story-map.md — 機能要件とユニットのマッピング
- [ ] Document code organization strategy
- [ ] Validate unit boundaries and dependencies
