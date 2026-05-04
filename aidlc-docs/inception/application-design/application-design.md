# パートナーBot — アプリケーション設計書

## 1. 設計概要

### アーキテクチャスタイル
- **フロントエンド**: React / Next.js（Webアプリ）
- **バックエンド**: Node.js（NestJS）
- **通信**: REST + WebSocket ハイブリッド
- **DB**: PostgreSQL + pgvector（開発: ローカル / 本番: AWS RDS）
- **クラウド**: AWS

### 設計原則
- **疎結合**: Unit①〜④は独立モジュール、ConversationEngineを共通基盤として共有
- **環境切り替え**: 環境変数でSTT・LLMプロバイダーを切り替え可能
- **セキュリティ**: SharedInfrastructureで横断的セキュリティ制御（SECURITY拡張準拠）
- **人間らしさ**: MistakeEngineを独立コンポーネントとして分離し、全Unitで再利用

---

## 2. コンポーネント構成

```
パートナーBot
├── COMP-00: PartnerSetup          # パートナー作成体験
├── COMP-01: ConversationEngine    # 会話エンジン（全Unitの基盤）
│   ├── WakeWordDetector           # Porcupine wake word検出
│   ├── STTAdapter                 # 音声認識（開発: Web Speech / 本番: Transcribe）
│   ├── LLMAdapter                 # LLM（開発: Gemini / 本番: Bedrock）
│   └── TTSAdapter                 # ElevenLabs TTS
├── COMP-02: AutoOrderMock         # 自動注文モック
│   ├── PlaywrightRunner           # バックグラウンドブラウザ操作
│   └── DummySiteServer            # UberEats風 / Amazon風ダミーサイト
├── COMP-03: MorningCall           # 朝モーニングコール
├── COMP-04: NightContent          # 夜お姉さんタイム
├── COMP-05: VRMRenderer           # VRMモデル表示（フロントエンド）
├── COMP-06: PartnerProfileRepository  # プロファイルDB
├── COMP-07: MemoryRepository      # 会話記憶 + pgvector
├── COMP-08: MistakeEngine         # 確率的ミスエンジン（純粋関数）
└── COMP-09: SharedInfrastructure  # 共通基盤（ログ・セキュリティ）
```

---

## 3. サービス層

| サービス | 担当ユースケース |
|---|---|
| ConversationOrchestrationService | 会話フロー全体（WebSocket） |
| PartnerSetupService | パートナー作成フロー |
| AutoOrderService | 自動注文フロー |
| MorningCallService | モーニングコールフロー |
| NightContentService | 夜タイムコンテンツ配信 |
| AdapterService | 環境別アダプター切り替え |

---

## 4. 技術スタック（設計フェーズ確定分）

| 項目 | 開発時 | 本番時 |
|---|---|---|
| バックエンド | **Node.js（NestJS）** | 同左 |
| LLM | Google Gemini API | Amazon Bedrock（Claude） |
| TTS | ElevenLabs | 同左 |
| STT | Web Speech API | Amazon Transcribe |
| Wake Word | Porcupine | 同左 |
| ベクトルDB | pgvector（ローカルPG） | AWS RDS for PostgreSQL + pgvector |
| スケジューラ | AWS EventBridge Scheduler（動的更新） | 同左 |
| メール | Amazon SES | 同左 |
| シークレット | AWS Secrets Manager | 同左 |

---

## 5. セキュリティ設計（SECURITY拡張準拠）

| ルール | 対応コンポーネント |
|---|---|
| SECURITY-01: 暗号化 | PostgreSQL（TLS + at-rest暗号化） |
| SECURITY-03: 構造化ログ | SharedInfrastructure |
| SECURITY-04: HTTPヘッダー | SharedInfrastructure（ミドルウェア） |
| SECURITY-05: 入力バリデーション | SharedInfrastructure（全APIエンドポイント） |
| SECURITY-09: エラーハンドリング | SharedInfrastructure（グローバルエラーハンドラー） |
| SECURITY-10: 依存管理 | ロックファイル + CI脆弱性スキャン |
| SECURITY-11: レート制限 | SharedInfrastructure（公開APIエンドポイント） |
| SECURITY-12: 認証情報管理 | AWS Secrets Manager |
| SECURITY-15: 例外処理 | SharedInfrastructure（グローバルエラーハンドラー） |

---

## 6. PBT対象コンポーネント（PBT拡張 部分適用）

| コンポーネント | PBTルール | テスト対象プロパティ |
|---|---|---|
| MistakeEngine | PBT-02, PBT-03 | ミスパターン確率の不変条件（合計=100%）、連続ミス抑制の不変条件 |
| MemoryRepository | PBT-02 | 会話記憶のシリアライズ/デシリアライズ ラウンドトリップ |
| STTAdapter | PBT-02 | 音声→テキスト変換の冪等性 |

---

## 7. 参照ドキュメント

- `components.md` — コンポーネント詳細定義
- `component-methods.md` — メソッドシグネチャ
- `services.md` — サービス層定義
- `component-dependency.md` — 依存関係・データフロー
