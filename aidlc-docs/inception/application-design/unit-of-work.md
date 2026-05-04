# Unit of Work 定義

## リポジトリ構成

**モノレポ構成**（1リポジトリ内にすべて配置）

```
partner-bot/                          # リポジトリルート
├── apps/
│   ├── frontend/                     # Next.js（React）フロントエンド
│   │   ├── src/
│   │   │   ├── components/           # UIコンポーネント
│   │   │   │   ├── setup/            # Unit-0: Setup UI
│   │   │   │   ├── conversation/     # Unit-1: 会話UI・VRMRenderer
│   │   │   │   ├── order/            # Unit-2: 注文UI
│   │   │   │   ├── morning-call/     # Unit-3: モーニングコールUI
│   │   │   │   └── night-content/    # Unit-4: 夜タイムUI
│   │   │   ├── hooks/                # カスタムフック
│   │   │   ├── lib/                  # WebSocket・API クライアント
│   │   │   └── pages/ (or app/)      # Next.js ルーティング
│   │   └── package.json
│   │
│   ├── backend/                      # Node.js（NestJS）バックエンド
│   │   ├── src/
│   │   │   ├── shared/               # Unit-X: SharedInfrastructure
│   │   │   │   ├── middleware/       # セキュリティヘッダー・レート制限
│   │   │   │   ├── logging/          # 構造化ログ
│   │   │   │   ├── validation/       # 入力バリデーション
│   │   │   │   └── adapters/         # STT・LLM・VectorDB アダプター
│   │   │   ├── setup/                # Unit-0: PartnerSetup モジュール
│   │   │   ├── conversation/         # Unit-1: ConversationEngine モジュール
│   │   │   │   ├── memory/           # MemoryRepository・RAG
│   │   │   │   └── wake-word/        # WakeWordDetector
│   │   │   ├── auto-order/           # Unit-2: AutoOrderMock モジュール
│   │   │   │   └── playwright/       # PlaywrightRunner
│   │   │   ├── morning-call/         # Unit-3: MorningCall モジュール
│   │   │   └── night-content/        # Unit-4: NightContent モジュール
│   │   └── package.json
│   │
│   └── dummy-sites/                  # Unit-2: ダミーサイト（React）
│       ├── ubereats-mock/            # UberEats風ダミーサイト
│       ├── amazon-mock/              # Amazon風ダミーサイト
│       └── package.json
│
├── packages/
│   └── shared-types/                 # 共有型定義（TypeScript）
│       └── package.json
│
├── package.json                      # ワークスペースルート（npm workspaces）
└── turbo.json (or nx.json)           # モノレポビルドツール設定
```

---

## Unit定義

### Unit-X: SharedInfrastructure（横断的基盤）

**開発順序**: 最初に実装（全Unitの前提）

**責務**:
- HTTPセキュリティヘッダーミドルウェア（SECURITY-04）
- 入力バリデーション基盤（SECURITY-05）
- 構造化ログ（SECURITY-03）
- レート制限（SECURITY-11）
- グローバルエラーハンドラー（SECURITY-15）
- STT / LLM / VectorDB アダプター（環境切り替え）
- AWS Secrets Manager連携

**含むコンポーネント**: SharedInfrastructure, AdapterService

**ディレクトリ**: `apps/backend/src/shared/`, `packages/shared-types/`

---

### Unit-0: Setup（パートナー作成体験）

**開発順序**: Unit-X完了後、Unit-1と並行可

**責務**:
- パートナー名・声・方言・性格・見た目の設定UI
- VRMモデルプレビュー
- 設定データのPostgreSQL永続化
- 初回挨拶トリガー（Unit-1に委譲）

**含むコンポーネント**: PartnerSetup, PartnerProfileRepository

**ディレクトリ**:
- `apps/frontend/src/components/setup/`
- `apps/backend/src/setup/`

**APIエンドポイント**:
- `POST /api/setup/profile`
- `GET /api/setup/profile`
- `PUT /api/setup/profile`

---

### Unit-1: ConversationEngine（会話エンジン）⭐ 最優先

**開発順序**: Unit-X完了後、最優先で実装

**責務**:
- 常時マイクオン管理
- Porcupine wake word検出
- STT（開発: Web Speech API / 本番: Amazon Transcribe）
- LLM会話生成（開発: Gemini / 本番: Bedrock）
- 長期記憶保存・RAG検索（pgvector）
- ElevenLabs TTS音声合成
- VRMモデルリップシンク・表情制御
- LINE風会話UI
- WebSocketリアルタイム通信

**含むコンポーネント**: ConversationEngine, MemoryRepository, VRMRenderer, WakeWordDetector, MistakeEngine（共有）

**ディレクトリ**:
- `apps/frontend/src/components/conversation/`
- `apps/backend/src/conversation/`

**APIエンドポイント**:
- `WS /ws/conversation`
- `GET /api/conversation/history`

---

### Unit-2: AutoOrderMock（自動注文モック）

**開発順序**: Unit-1完了後（Unit-3・4と並行可）

**責務**:
- 注文意図抽出（Unit-1のConversationEngineを利用）
- Playwrightバックグラウンド自動操作
- UberEats風・Amazon風ダミーサイト
- Amazon SESメール通知
- 5分間キャンセルトークン管理
- 確率的ミスロジック（MistakeEngine利用）
- 1日上限金額チェック

**含むコンポーネント**: AutoOrderMock, PlaywrightRunner, DummySiteServer, OrderRepository

**ディレクトリ**:
- `apps/frontend/src/components/order/`
- `apps/backend/src/auto-order/`
- `apps/dummy-sites/ubereats-mock/`
- `apps/dummy-sites/amazon-mock/`

**APIエンドポイント**:
- `POST /api/order/execute`
- `POST /api/order/cancel`
- `GET /api/order/history`

---

### Unit-3: MorningCall（朝モーニングコール）

**開発順序**: Unit-1完了後（Unit-2・4と並行可）

**責務**:
- 起床時刻設定・AWS EventBridge Scheduler動的更新
- Google Calendar API重要予定検出
- 確率的ミスパターンエンジン（MistakeEngine利用）
- パーソナライズ起床メッセージ生成（Unit-1のLLM利用）
- 応答チェック・再通知
- 夜の睡眠7時間声かけ

**含むコンポーネント**: MorningCall, GoogleCalendarAdapter, EventBridgeSchedulerAdapter

**ディレクトリ**:
- `apps/frontend/src/components/morning-call/`
- `apps/backend/src/morning-call/`

**APIエンドポイント**:
- `PUT /api/morning-call/settings`
- `POST /api/morning-call/trigger`（内部）

---

### Unit-4: NightContent（夜お姉さんタイム）

**開発順序**: Unit-1完了後（Unit-2・3と並行可）

**責務**:
- 毎晩22:00 EventBridgeトリガー
- コンテンツDB管理（開発: 手動 / 本番: バッチ生成）
- お姉さん風口調プロンプト制御（Unit-1のLLM利用）
- 本番バッチ: 会話ログキーワード抽出→LLM生成→DB保存

**含むコンポーネント**: NightContent, ContentRepository, BatchContentGenerator

**ディレクトリ**:
- `apps/frontend/src/components/night-content/`
- `apps/backend/src/night-content/`

**APIエンドポイント**:
- `POST /api/night-content/trigger`（内部）
- `POST /api/night-content/batch`（内部）

---

## 開発順序サマリー

```
Phase 1（必須前提）:
  Unit-X: SharedInfrastructure

Phase 2（順次）:
  Unit-0: Setup
  Unit-1: ConversationEngine ⭐ 最優先

Phase 3（Unit-1完了後、並行開発可）:
  Unit-2: AutoOrderMock  ─┐
  Unit-3: MorningCall    ─┼─ 並行
  Unit-4: NightContent   ─┘
```
