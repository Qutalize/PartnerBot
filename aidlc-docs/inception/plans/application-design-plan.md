# アプリケーション設計計画

## 設計チェックリスト

- [x] Step 1: コンテキスト分析（requirements.md・stories.md読み込み済み）
- [x] Step 2: 設計計画作成（本ドキュメント）
- [x] Step 3: 必須成果物の計画
- [x] Step 4: 質問生成（下記参照）
- [x] Step 5: 計画ファイル保存
- [x] Step 6: ユーザー回答待ち
- [x] Step 7: 回答収集
- [x] Step 8: 回答分析（矛盾なし）
- [x] Step 9: フォローアップ不要
- [x] Step 10: 設計成果物生成
  - [x] components.md
  - [x] component-methods.md
  - [x] services.md
  - [x] component-dependency.md
  - [x] application-design.md（統合ドキュメント）
- [x] Step 11: 承認ログ
- [x] Step 12: 完了メッセージ提示

---

## 設計スコープ

本プロジェクトは以下の5コンポーネントで構成されます：

| コンポーネント | 概要 |
|---|---|
| Setup | パートナー作成体験（名前・声・性格・VRM） |
| Unit①: ConversationEngine | 音声入力・LLM・TTS・VRM・記憶・RAG |
| Unit②: AutoOrderMock | ダミーサイト・Playwright・メール通知 |
| Unit③: MorningCall | スケジューラ・Google Calendar・ミスパターン |
| Unit④: NightContent | コンテンツDB・バッチ処理・22:00配信 |

---

## 質問

以下の質問にお答えください。`[Answer]:` タグの後に選択肢の文字を記入してください。

---

### 質問 1
フロントエンドとバックエンドの通信方式はどれを希望しますか？

A) REST API（シンプル、標準的）
B) WebSocket（リアルタイム会話に最適）
C) REST + WebSocket のハイブリッド（会話はWS、その他はREST）
D) その他（[Answer]: タグの後に詳細を記入してください）

[Answer]: C

---

### 質問 2
会話記憶（長期記憶 + RAG）のベクトルDB・検索基盤はどれを使用しますか？

A) pgvector（PostgreSQL拡張、追加インフラ不要）
B) Amazon OpenSearch Serverless（AWSネイティブ）
C) Pinecone（マネージドベクトルDB）
D) その他（[Answer]: タグの後に詳細を記入してください）

[Answer]: D Aで開発し、そのスタックをAWS環境（AWS RDS for PostgreSQL）でそのまま運用したい

---

### 質問 3
音声入力（STT: Speech-to-Text）はどれを使用しますか？

A) Web Speech API（ブラウザ標準、無料）
B) OpenAI Whisper API
C) Amazon Transcribe
D) その他（[Answer]: タグの後に詳細を記入してください）

[Answer]: D 開発中はAを用い、本番はCを使いたい

---

### 質問 4
wake word検出（Botの名前を呼ぶ検出）はどのように実装しますか？

A) ブラウザのWeb Speech APIで常時認識し、キーワードマッチング
B) 軽量なローカルwake word検出ライブラリ（Porcupine等）
C) 常時マイクオンでサーバーサイド処理
D) その他（[Answer]: タグの後に詳細を記入してください）

[Answer]: B

---

### 質問 5
Unit③（モーニングコール）のスケジューラはどのように実装しますか？

A) フロントエンドのsetTimeout / Web Workers（ブラウザ常駐）
B) バックエンドのcronジョブ（サーバーサイドスケジューリング）
C) AWS EventBridge Scheduler
D) その他（[Answer]: タグの後に詳細を記入してください）

[Answer]: D 毎日、モーニングコールの時間が変化する可能性もある為、AWSの機能での実装を優先しつつ、変更のしやすさが適当な実装にして。

---

### 質問 6
メール通知（Unit②の注文完了通知）はどのサービスを使用しますか？

A) Amazon SES（AWS標準、低コスト）
B) SendGrid
C) Nodemailer（SMTPサーバー経由）
D) その他（[Answer]: タグの後に詳細を記入してください）

[Answer]: A

---

すべての質問に回答したら「完了しました」とお知らせください。
