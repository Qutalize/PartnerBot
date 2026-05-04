# サービス定義

## サービス層の役割

サービス層はコンポーネント間のオーケストレーションを担います。
各サービスは複数のコンポーネントを協調させてユースケースを実現します。

---

## SVC-01: ConversationOrchestrationService

**責務**: 会話フロー全体のオーケストレーション

**オーケストレーション**:
1. WebSocket接続受信 → ConversationEngine.startSession()
2. 音声ストリーム受信 → WakeWordDetector → STTAdapter → LLMAdapter
3. LLM応答 → TTSAdapter → VRMRenderer（リップシンク・表情）
4. 会話履歴 → MemoryRepository（保存）
5. RAG検索 → MemoryRepository.searchMemory() → LLMプロンプト注入

**エンドポイント**:
- `WS /ws/conversation` — 会話WebSocketセッション
- `GET /api/conversation/history` — 会話履歴取得（REST）

---

## SVC-02: PartnerSetupService

**責務**: パートナー作成フローのオーケストレーション

**オーケストレーション**:
1. 設定入力受信 → バリデーション → PartnerProfileRepository.create()
2. VRMモデル選択 → VRMRenderer.loadVRMModel()
3. 設定完了 → ConversationEngine（初回挨拶生成・音声再生）

**エンドポイント**:
- `POST /api/setup/profile` — プロファイル作成（REST）
- `GET /api/setup/profile` — プロファイル取得（REST）
- `PUT /api/setup/profile` — プロファイル更新（REST）

---

## SVC-03: AutoOrderService

**責務**: 自動注文フローのオーケストレーション

**オーケストレーション**:
1. 注文発話 → ConversationEngine（意図抽出）
2. 意図確定 → MistakeEngine.applyOrderMistake()
3. 注文実行 → PlaywrightRunner（バックグラウンド）
4. 完了 → EmailService（Amazon SES通知）
5. キャンセル受付 → OrderRepository（5分間トークン検証）
6. 結果 → ConversationEngine（会話フィードバック）

**エンドポイント**:
- `POST /api/order/execute` — 注文実行（REST）
- `POST /api/order/cancel` — 注文キャンセル（REST）
- `GET /api/order/history` — 注文履歴（REST）

---

## SVC-04: MorningCallService

**責務**: モーニングコールフローのオーケストレーション

**オーケストレーション**:
1. 起床時刻設定 → EventBridgeSchedulerAdapter（動的スケジュール登録/更新）
2. スケジューラトリガー → GoogleCalendarAdapter（重要予定検出）
3. MistakeEngine.selectMorningPattern() → メッセージ生成
4. ConversationEngine（音声生成・WebSocket配信）
5. 5分後 → 応答チェック → 未応答かつ重要予定あり → 再通知
6. 夜の会話中 → 睡眠7時間チェック → 声かけ

**エンドポイント**:
- `PUT /api/morning-call/settings` — 起床時刻設定（REST）
- `POST /api/morning-call/trigger` — スケジューラからの内部トリガー（REST、内部のみ）

---

## SVC-05: NightContentService

**責務**: 夜タイムコンテンツ配信のオーケストレーション

**オーケストレーション**:
1. 22:00 EventBridgeトリガー → ContentRepository.getContent()
2. LLMプロンプト構築（お姉さん口調） → ConversationEngine（音声生成・配信）
3. バッチ処理（本番）: 会話ログ → キーワード抽出 → LLM生成 → ContentRepository.save()

**エンドポイント**:
- `POST /api/night-content/trigger` — スケジューラからの内部トリガー（REST、内部のみ）
- `POST /api/night-content/batch` — バッチ処理トリガー（REST、内部のみ）

---

## SVC-06: AdapterService（環境切り替え）

**責務**: 開発/本番環境に応じたアダプター切り替え

| アダプター | 開発 | 本番 |
|---|---|---|
| STTAdapter | Web Speech API（ブラウザ） | Amazon Transcribe |
| LLMAdapter | Google Gemini API | Amazon Bedrock（Claude） |
| VectorDBAdapter | pgvector（ローカルPG） | AWS RDS for PostgreSQL + pgvector |
| SchedulerAdapter | EventBridge Scheduler | EventBridge Scheduler（同一） |

**切り替え方法**: 環境変数（`STT_PROVIDER`, `LLM_PROVIDER`）で制御
