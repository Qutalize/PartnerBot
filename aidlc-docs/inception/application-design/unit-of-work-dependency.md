# Unit of Work 依存関係

## 依存関係マトリクス

| Unit | 依存先Unit | 依存種別 |
|---|---|---|
| Unit-X: SharedInfrastructure | なし | — |
| Unit-0: Setup | Unit-X | 実行時依存（ミドルウェア・ログ） |
| Unit-1: ConversationEngine | Unit-X, Unit-0 | 実行時依存（プロファイル参照） |
| Unit-2: AutoOrderMock | Unit-X, Unit-1 | 実行時依存（会話・意図抽出） |
| Unit-3: MorningCall | Unit-X, Unit-1 | 実行時依存（メッセージ生成・音声） |
| Unit-4: NightContent | Unit-X, Unit-1 | 実行時依存（音声再生・LLM） |

## 依存関係図

```
Unit-X: SharedInfrastructure
    |
    +──────────────────────────────────────────+
    |              |              |             |
Unit-0: Setup  Unit-1: Conv  Unit-2: Order  Unit-3: Morning  Unit-4: Night
               ⭐最優先
                    |
          +---------+---------+
          |                   |
    Unit-2: Order      Unit-3: Morning
    Unit-4: Night
```

## 共有コンポーネント

| コンポーネント | 利用Unit |
|---|---|
| MistakeEngine | Unit-2（注文ミス）, Unit-3（モーニングコールミス） |
| ConversationEngine | Unit-2（意図抽出・フィードバック）, Unit-3（メッセージ生成）, Unit-4（音声再生） |
| EventBridgeSchedulerAdapter | Unit-3（起床スケジュール）, Unit-4（22:00スケジュール） |
| AdapterService（LLM/STT） | Unit-1（主要利用）, Unit-3・4（間接利用） |

## 開発ブロッカー分析

| Unit | ブロッカー | 解除条件 |
|---|---|---|
| Unit-0 | Unit-X完了 | SharedInfrastructure実装済み |
| Unit-1 | Unit-X完了 | SharedInfrastructure実装済み |
| Unit-2 | Unit-1完了 | ConversationEngine（意図抽出API）実装済み |
| Unit-3 | Unit-1完了 | ConversationEngine（音声生成API）実装済み |
| Unit-4 | Unit-1完了 | ConversationEngine（音声再生API）実装済み |

## 外部サービス依存

| Unit | 外部サービス | 開発時代替 |
|---|---|---|
| Unit-1 | Gemini API, ElevenLabs, Porcupine | Gemini無料枠, ElevenLabs試用 |
| Unit-1（本番） | Amazon Bedrock, Amazon Transcribe | — |
| Unit-2 | Amazon SES | ローカルSMTPモック |
| Unit-3 | Google Calendar API, EventBridge | カレンダーモック |
| Unit-3・4 | EventBridge Scheduler | ローカルcronモック |
