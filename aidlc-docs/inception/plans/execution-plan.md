# 実行計画（Execution Plan）

## 詳細分析サマリー

### 変更影響評価
- **ユーザー向け変更**: Yes — Setup〜Unit④すべてがユーザー体験に直結
- **構造的変更**: Yes — 新規システム（グリーンフィールド）、5コンポーネント構成
- **データモデル変更**: Yes — 会話記憶・パートナー設定・注文履歴・コンテンツDBなど複数スキーマ
- **API変更**: Yes — LLM・TTS・Google Calendar・ElevenLabs等の外部API統合
- **NFR影響**: Yes — セキュリティ（SECURITY拡張有効）・パフォーマンス・PBT（部分適用）

### リスク評価
- **リスクレベル**: High
- **ロールバック複雑度**: Moderate（各Unitが独立モジュールのため部分ロールバック可能）
- **テスト複雑度**: Complex（LLM・TTS・Playwright・外部API統合テストが必要）

---

## ワークフロー可視化

```
INCEPTION PHASE
  [x] Workspace Detection       COMPLETED
  [ ] Reverse Engineering       SKIPPED（グリーンフィールド）
  [x] Requirements Analysis     COMPLETED
  [ ] User Stories              EXECUTE（複数ペルソナ・複雑ビジネスロジック）
  [ ] Workflow Planning         IN PROGRESS
  [ ] Application Design        EXECUTE（新規コンポーネント設計が必要）
  [ ] Units Generation          EXECUTE（5Unit分解が必要）

CONSTRUCTION PHASE（各Unitに対して）
  [ ] Functional Design         EXECUTE（新規データモデル・複雑ビジネスロジック）
  [ ] NFR Requirements          EXECUTE（セキュリティ・パフォーマンス要件あり）
  [ ] NFR Design                EXECUTE（NFR Requirementsを実行するため）
  [ ] Infrastructure Design     EXECUTE（AWS・PostgreSQL・外部API統合）
  [ ] Code Generation           EXECUTE（ALWAYS）
  [ ] Build and Test            EXECUTE（ALWAYS）

OPERATIONS PHASE
  [ ] Operations                PLACEHOLDER
```

---

## 実行フェーズ詳細

### 🔵 INCEPTION PHASE

- [x] **Workspace Detection** — COMPLETED
  - 理由: グリーンフィールド確認済み
- [x] **Reverse Engineering** — SKIPPED
  - 理由: 既存コードなし
- [x] **Requirements Analysis** — COMPLETED
  - 理由: 要件定義書作成・承認済み
- [ ] **User Stories** — EXECUTE
  - 理由: 複数ユーザーペルソナ、新規ユーザー向け機能、複雑なビジネスロジック（ミスパターン・感情設計）、受け入れ基準が必要
- [ ] **Workflow Planning** — IN PROGRESS（本ドキュメント）
- [ ] **Application Design** — EXECUTE
  - 理由: Setup・Unit①〜④の新規コンポーネント設計、サービス層設計、コンポーネント間依存関係の定義が必要
- [ ] **Units Generation** — EXECUTE
  - 理由: 5つの独立Unit（Setup・Unit①〜④）への分解と依存関係整理が必要

### 🟢 CONSTRUCTION PHASE（各Unitに対して実行）

- [ ] **Functional Design** — EXECUTE
  - 理由: 会話記憶スキーマ・ミスパターンエンジン・RAGロジック・注文フロー等の複雑なビジネスロジック設計が必要
- [ ] **NFR Requirements** — EXECUTE
  - 理由: セキュリティ拡張（全15ルール）・PBT拡張（部分適用）・パフォーマンス要件（5秒以内応答等）あり
- [ ] **NFR Design** — EXECUTE
  - 理由: NFR Requirementsを実行するため自動実行
- [ ] **Infrastructure Design** — EXECUTE
  - 理由: AWS（Bedrock・PostgreSQL・シークレットマネージャー）・ElevenLabs・Google Calendar API等のインフラ設計が必要
- [ ] **Code Generation** — EXECUTE（ALWAYS）
  - 理由: 実装計画・コード生成
- [ ] **Build and Test** — EXECUTE（ALWAYS）
  - 理由: ビルド・テスト・検証

### 🟡 OPERATIONS PHASE

- [ ] **Operations** — PLACEHOLDER
  - 理由: 将来のデプロイ・監視ワークフロー用

---

## Unitスコープ（Units Generation後に詳細化）

| Unit | 内容 | 依存 |
|---|---|---|
| Unit-0: Setup | パートナー作成体験（名前・声・性格・VRM） | なし |
| Unit-1: 会話エンジン | 音声入力・LLM・TTS・VRM・記憶・RAG | Unit-0 |
| Unit-2: 自動注文モック | ダミーサイト・Playwright・メール通知 | Unit-1 |
| Unit-3: モーニングコール | スケジューラ・Google Calendar・ミスパターン | Unit-1 |
| Unit-4: 夜お姉さんタイム | コンテンツDB・バッチ処理・22:00配信 | Unit-1 |

---

## 推定タイムライン

- **合計フェーズ数**: 10ステージ（スキップ除く）
- **Inceptionフェーズ**: User Stories → Application Design → Units Generation
- **Constructionフェーズ**: 5Unit × (Functional Design + NFR Requirements + NFR Design + Infrastructure Design + Code Generation) + Build and Test

---

## 成功基準

- **主目標**: ハッカソン提出可能なパートナーBotの完全実装
- **主要成果物**:
  - Setup + Unit①〜④の動作するWebアプリ
  - 音声会話・VRMモデル・長期記憶・RAG
  - UberEats/Amazon風ダミーサイト + Playwright自動注文
  - モーニングコール + Google Calendar連携
  - 夜お姉さんタイム
- **品質ゲート**:
  - セキュリティ拡張 全15ルール準拠
  - PBT拡張 部分適用（PBT-02・03・07・08・09）準拠
  - 会話応答5秒以内
