# コンポーネント依存関係

## 依存関係マトリクス

| コンポーネント | 依存先 |
|---|---|
| PartnerSetup | PartnerProfileRepository, VRMRenderer, ConversationEngine |
| ConversationEngine | STTAdapter, LLMAdapter, TTSAdapter, MemoryRepository, RAGSearchService, VRMRenderer, WakeWordDetector |
| AutoOrderMock | ConversationEngine, PlaywrightRunner, DummySiteServer, EmailService, OrderRepository, MistakeEngine |
| MorningCall | ConversationEngine, GoogleCalendarAdapter, EventBridgeSchedulerAdapter, MistakeEngine, UserSettingsRepository |
| NightContent | ConversationEngine, ContentRepository, EventBridgeSchedulerAdapter, BatchContentGenerator |
| VRMRenderer | （外部依存なし — フロントエンド完結） |
| MistakeEngine | （外部依存なし — 純粋関数） |
| SharedInfrastructure | AWS Secrets Manager, CloudWatch Logs |

---

## データフロー図

```
ユーザー（ブラウザ）
    |
    | WebSocket（会話）/ REST（その他）
    v
+------------------+
| API Gateway /    |
| Next.js API      |
+------------------+
    |
    +---> ConversationOrchestrationService
    |         |
    |         +---> WakeWordDetector（Porcupine）
    |         +---> STTAdapter（Web Speech API / Transcribe）
    |         +---> LLMAdapter（Gemini / Bedrock）
    |         +---> MemoryRepository（pgvector）
    |         +---> TTSAdapter（ElevenLabs）
    |         +---> VRMRenderer（フロントエンド）
    |
    +---> PartnerSetupService
    |         +---> PartnerProfileRepository（PostgreSQL）
    |
    +---> AutoOrderService
    |         +---> PlaywrightRunner（バックグラウンド）
    |         +---> DummySiteServer（UberEats風/Amazon風）
    |         +---> EmailService（Amazon SES）
    |         +---> MistakeEngine
    |
    +---> MorningCallService
    |         +---> EventBridgeSchedulerAdapter（AWS）
    |         +---> GoogleCalendarAdapter
    |         +---> MistakeEngine
    |
    +---> NightContentService
              +---> ContentRepository（PostgreSQL）
              +---> EventBridgeSchedulerAdapter（AWS）
```

---

## 通信パターン

| パターン | 使用箇所 |
|---|---|
| WebSocket（双方向リアルタイム） | 会話セッション（音声ストリーム・TTS再生・VRMシグナル） |
| REST API（同期） | Setup・注文・設定変更・履歴取得 |
| イベント駆動（EventBridge） | モーニングコール・夜タイム配信トリガー |
| バックグラウンドプロセス | Playwright自動操作・バッチコンテンツ生成 |

---

## 外部サービス依存

| サービス | 用途 | 環境 |
|---|---|---|
| Google Gemini API | LLM（会話生成） | 開発 |
| Amazon Bedrock（Claude） | LLM（会話生成） | 本番 |
| ElevenLabs API | TTS（音声合成） | 両環境 |
| Web Speech API | STT（音声認識） | 開発 |
| Amazon Transcribe | STT（音声認識） | 本番 |
| Porcupine | Wake word検出 | 両環境 |
| Google Calendar API | 重要予定検出 | 両環境 |
| Amazon SES | メール通知 | 両環境 |
| AWS EventBridge Scheduler | スケジューリング | 両環境 |
| AWS Secrets Manager | シークレット管理 | 両環境 |
| PostgreSQL + pgvector | DB・ベクトル検索 | 両環境 |
