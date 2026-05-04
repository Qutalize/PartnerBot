# Unit of Work 計画

## チェックリスト

### PART 1: 計画
- [x] Step 1: Unit of Work計画作成
- [x] Step 2: 必須成果物の計画
- [x] Step 3: 質問生成（下記参照）
- [x] Step 4: 計画ファイル保存
- [x] Step 5: ユーザー回答待ち
- [x] Step 6: 回答収集
- [x] Step 7: 回答分析（矛盾なし）
- [x] Step 8: フォローアップ不要
- [x] Step 9: 承認（回答受信をもって承認とみなす）

### PART 2: 生成
- [x] Step 12: unit-of-work.md 生成
- [x] Step 13: unit-of-work-dependency.md 生成
- [x] Step 14: unit-of-work-story-map.md 生成
- [x] Step 15: 進捗更新・完了確認

---

## 想定Unit構成（Application Designより）

| Unit | コンポーネント | 概要 |
|---|---|---|
| Unit-0: Setup | PartnerSetup, PartnerProfileRepository | パートナー作成体験 |
| Unit-1: ConversationEngine | ConversationEngine, MemoryRepository, VRMRenderer, MistakeEngine（共有） | 会話・記憶・VRM |
| Unit-2: AutoOrderMock | AutoOrderMock, PlaywrightRunner, DummySiteServer, OrderRepository | 自動注文モック |
| Unit-3: MorningCall | MorningCall, GoogleCalendarAdapter, EventBridgeSchedulerAdapter | 朝モーニングコール |
| Unit-4: NightContent | NightContent, ContentRepository, BatchContentGenerator | 夜お姉さんタイム |
| Unit-X: SharedInfrastructure | SharedInfrastructure, AdapterService | 共通基盤（全Unitに横断） |

---

## 質問

以下の質問にお答えください。`[Answer]:` タグの後に選択肢の文字を記入してください。

---

### 質問 1
コードのディレクトリ構成はどれを希望しますか？

A) モノレポ（1リポジトリ内にフロントエンド・バックエンド・ダミーサイトを配置）
B) マルチレポ（フロントエンド・バックエンド・ダミーサイトを別リポジトリ）
C) その他（[Answer]: タグの後に詳細を記入してください）

[Answer]: A

---

### 質問 2
Unit間の開発順序はどれを希望しますか？

A) 順次（Unit-0 → Unit-1 → Unit-2 → Unit-3 → Unit-4）
B) Unit-1を最優先で完成させ、その後Unit-2〜4を並行開発
C) その他（[Answer]: タグの後に詳細を記入してください）

[Answer]: B

---

すべての質問に回答したら「完了しました」とお知らせください。
