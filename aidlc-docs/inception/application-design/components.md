# コンポーネント定義

## 技術スタック決定（設計フェーズ確定分）

| 項目 | 開発時 | 本番時 |
|---|---|---|
| 通信方式 | REST + WebSocket ハイブリッド | 同左 |
| ベクトルDB | pgvector（ローカルPostgreSQL） | AWS RDS for PostgreSQL + pgvector |
| STT | Web Speech API（ブラウザ） | Amazon Transcribe |
| Wake Word検出 | Porcupine（ローカルライブラリ） | 同左 |
| スケジューラ | AWS EventBridge Scheduler（動的更新） | 同左 |
| メール通知 | Amazon SES | 同左 |

---

## コンポーネント一覧

### COMP-00: PartnerSetup（パートナー作成）

**責務**:
- パートナーの名前・声・方言・性格・見た目パラメータの収集
- VRMモデルのプレビュー表示
- 設定データのDBへの永続化
- 初回挨拶トリガー

**インターフェース**:
- 入力: ユーザーの設定入力（名前・声タイプ・性格スライダー値・VRMモデルID）
- 出力: PartnerProfile（永続化済み設定オブジェクト）

**依存コンポーネント**:
- PartnerProfileRepository（DB永続化）
- VRMRenderer（プレビュー表示）
- ConversationEngine（初回挨拶生成）

---

### COMP-01: ConversationEngine（会話エンジン）

**責務**:
- 常時マイクオン管理・wake word検出（Porcupine）
- 音声入力のSTT変換（開発: Web Speech API / 本番: Amazon Transcribe）
- LLMへのプロンプト構築・送信（開発: Gemini / 本番: Bedrock）
- 長期記憶の保存・RAG検索
- ElevenLabsによるTTS音声合成
- VRMモデルへのリップシンク・表情制御シグナル送信
- LINE風会話UIへのメッセージ送信

**インターフェース**:
- 入力: 音声ストリーム / テキスト / システムイベント（記念日等）
- 出力: 音声バイナリ + VRMアニメーションシグナル + テキストメッセージ

**依存コンポーネント**:
- WakeWordDetector
- STTAdapter（開発/本番切り替え）
- LLMAdapter（開発/本番切り替え）
- MemoryRepository（長期記憶DB）
- RAGSearchService
- TTSAdapter（ElevenLabs）
- VRMRenderer

---

### COMP-02: AutoOrderMock（自動注文モック）

**責務**:
- 会話からの注文意図抽出
- Playwrightによるダミーサイト（UberEats風 / Amazon風）のバックグラウンド自動操作
- 注文完了後のメール通知（Amazon SES）
- 5分間キャンセルトークン管理
- 確率的ミスロジック実行
- 1日上限金額チェック

**インターフェース**:
- 入力: 注文意図テキスト / キャンセルリクエスト
- 出力: 注文結果オブジェクト + メール送信 + 会話フィードバックテキスト

**依存コンポーネント**:
- ConversationEngine（意図抽出・フィードバック）
- PlaywrightRunner（バックグラウンドブラウザ操作）
- DummySiteServer（UberEats風 / Amazon風ダミーサイト）
- EmailService（Amazon SES）
- OrderRepository（注文履歴・上限管理）
- MistakeEngine（確率的ミスロジック）

---

### COMP-03: MorningCall（朝モーニングコール）

**責務**:
- ユーザー設定起床時刻の管理
- AWS EventBridge Schedulerへの動的スケジュール登録・更新
- Google Calendar APIによる重要予定検出
- 確率的ミスパターンエンジン実行
- パーソナライズ起床メッセージのLLM生成
- 起床通知5分後の応答チェック・再通知
- 夜の会話中の睡眠7時間声かけロジック

**インターフェース**:
- 入力: 起床時刻設定 / Googleカレンダーイベント / スケジューラトリガー
- 出力: 音声起床メッセージ + VRMアニメーションシグナル

**依存コンポーネント**:
- ConversationEngine（メッセージ生成・音声再生）
- GoogleCalendarAdapter
- EventBridgeSchedulerAdapter
- MistakeEngine（ミスパターン）
- UserSettingsRepository

---

### COMP-04: NightContent（夜お姉さんタイム）

**責務**:
- 毎晩22:00のコンテンツ配信スケジューリング
- コンテンツDBからのトリビア取得
- お姉さん風口調でのLLMプロンプト制御
- 本番時: 会話ログからのキーワード抽出バッチ処理
- 本番時: LLMによるパーソナライズコンテンツ事前生成・DB保存

**インターフェース**:
- 入力: スケジューラトリガー / バッチ処理トリガー
- 出力: 音声コンテンツ + VRMアニメーションシグナル

**依存コンポーネント**:
- ConversationEngine（音声再生）
- ContentRepository（トリビアDB）
- EventBridgeSchedulerAdapter
- BatchContentGenerator（本番時）

---

### COMP-05: VRMRenderer（VRMモデル表示）

**責務**:
- VRMモデルのブラウザ内レンダリング（Three.js / @pixiv/three-vrm）
- リップシンク制御（音声波形→口形素マッピング）
- 表情モーフ制御（感情シグナル→表情適用）
- アイドルアニメーション管理

**インターフェース**:
- 入力: 音声波形データ / 感情シグナル / アニメーション指示
- 出力: レンダリング済みVRMモデル（Canvas）

---

### COMP-06: PartnerProfileRepository（プロファイルDB）

**責務**:
- パートナー設定データのCRUD
- PostgreSQLへの永続化

---

### COMP-07: MemoryRepository（会話記憶DB）

**責務**:
- 会話履歴のCRUD
- pgvectorによるベクトル埋め込み保存
- セマンティック検索（RAG用）
- 記念日・重要発言の管理

---

### COMP-08: MistakeEngine（確率的ミスエンジン）

**責務**:
- Unit②用: 注文ミスロジック（確率的ハプニング生成）
- Unit③用: モーニングコールミスパターン選択（通常65%・遅延20%・古い情報10%・スキップ5%）
- 連続ミス抑制ロジック（ETH-03）
- mistake_rateユーザー設定の適用

**インターフェース**:
- 入力: コンテキスト（重要予定フラグ・連続ミス回数・mistake_rate設定）
- 出力: ミスパターン選択結果

---

### COMP-09: SharedInfrastructure（共通基盤）

**責務**:
- 構造化ログ（SECURITY-03準拠）
- HTTPセキュリティヘッダーミドルウェア（SECURITY-04）
- 入力バリデーション（SECURITY-05）
- レート制限（SECURITY-11）
- グローバルエラーハンドラー（SECURITY-15）
- シークレット管理（AWS Secrets Manager連携）
