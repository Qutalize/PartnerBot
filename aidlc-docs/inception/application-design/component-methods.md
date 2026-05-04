# コンポーネントメソッド定義

> **注意**: 詳細なビジネスロジックはCONSTRUCTION PHASEのFunctional Designで定義します。
> 本ドキュメントはインターフェース契約（シグネチャ・入出力型）を定義します。

---

## COMP-00: PartnerSetup

```typescript
// パートナープロファイルの作成・保存
createPartnerProfile(input: PartnerProfileInput): Promise<PartnerProfile>

// プロファイルの取得
getPartnerProfile(profileId: string): Promise<PartnerProfile>

// プロファイルの更新
updatePartnerProfile(profileId: string, updates: Partial<PartnerProfileInput>): Promise<PartnerProfile>

// 初回挨拶の生成トリガー
triggerFirstGreeting(profile: PartnerProfile): Promise<void>
```

**型定義**:
```typescript
interface PartnerProfileInput {
  name: string                    // パートナー名
  voiceType: 'standard' | 'kansai' | 'hakata'
  personalityParams: PersonalityParams  // スライダー値
  appearanceParams: AppearanceParams    // 見た目スライダー値
  vrmModelId: string
}

interface PartnerProfile extends PartnerProfileInput {
  id: string
  createdAt: Date
  firstGreetingDone: boolean
}
```

---

## COMP-01: ConversationEngine

```typescript
// 会話セッション開始（WebSocket接続時）
startSession(profileId: string): Promise<ConversationSession>

// 音声入力処理（STT → LLM → TTS パイプライン）
processVoiceInput(sessionId: string, audioChunk: AudioBuffer): Promise<ConversationResponse>

// wake word検出イベント処理
onWakeWordDetected(sessionId: string, wakeWord: string): Promise<void>

// テキスト入力処理（フォールバック）
processTextInput(sessionId: string, text: string): Promise<ConversationResponse>

// 会話履歴の取得
getConversationHistory(profileId: string, limit: number): Promise<ConversationMessage[]>

// 記憶の検索（RAG）
searchMemory(profileId: string, query: string): Promise<MemorySearchResult[]>

// 会話セッション終了
endSession(sessionId: string): Promise<void>
```

**型定義**:
```typescript
interface ConversationResponse {
  text: string
  audioBuffer: ArrayBuffer        // ElevenLabs TTS出力
  emotionSignal: EmotionSignal    // VRM表情制御用
  lipSyncData: LipSyncFrame[]     // リップシンク用
}

interface EmotionSignal {
  type: 'happy' | 'surprised' | 'shy' | 'sad' | 'neutral'
  intensity: number               // 0.0 - 1.0
}

type LLMProvider = 'gemini' | 'bedrock'
type STTProvider = 'web-speech-api' | 'amazon-transcribe'
```

---

## COMP-02: AutoOrderMock

```typescript
// 注文意図の抽出
extractOrderIntent(utterance: string): Promise<OrderIntent>

// 注文フローの実行（Playwright）
executeOrder(intent: OrderIntent, profileId: string): Promise<OrderResult>

// 注文のキャンセル
cancelOrder(orderId: string, cancelToken: string): Promise<CancelResult>

// ミスロジックの適用
applyMistakeLogic(order: OrderDraft, context: MistakeContext): Promise<OrderDraft>

// 1日上限チェック
checkDailyLimit(profileId: string, amount: number): Promise<LimitCheckResult>
```

**型定義**:
```typescript
interface OrderIntent {
  siteType: 'ubereats-mock' | 'amazon-mock'
  category?: string               // 食事カテゴリ等
  preferences?: string[]
}

interface OrderResult {
  orderId: string
  items: OrderItem[]
  totalAmount: number
  estimatedDelivery?: string
  cancelToken: string             // 5分間有効
  cancelExpiry: Date
  hasMistake: boolean
}
```

---

## COMP-03: MorningCall

```typescript
// 起床時刻の設定・EventBridgeスケジュール更新
setWakeUpTime(profileId: string, time: WakeUpTimeConfig): Promise<void>

// 重要予定の検出（Google Calendar）
detectCriticalEvents(profileId: string, date: Date): Promise<CalendarEvent[]>

// ミスパターンの選択
selectMistakePattern(context: MorningCallContext): MistakePattern

// 起床メッセージの生成・配信
deliverMorningCall(profileId: string): Promise<void>

// 応答チェック・再通知
checkResponseAndEscalate(profileId: string, callId: string): Promise<void>

// 睡眠声かけの判定
checkSleepReminder(profileId: string, currentTime: Date): Promise<SleepReminderResult>
```

**型定義**:
```typescript
interface WakeUpTimeConfig {
  hour: number
  minute: number
  timezone: string
}

type MistakePattern = 'normal' | 'delayed' | 'stale-info' | 'skip'

interface MorningCallContext {
  hasCriticalEvent: boolean
  consecutiveMistakeCount: number
  mistakeRate: number             // ユーザー設定 0.0-1.0
}
```

---

## COMP-04: NightContent

```typescript
// 夜タイムコンテンツの配信
deliverNightContent(profileId: string): Promise<void>

// コンテンツの取得（開発: 手動DB / 本番: パーソナライズDB）
getContent(profileId: string): Promise<NightContentItem>

// バッチ処理: キーワード抽出 → コンテンツ生成（本番）
runContentGenerationBatch(profileId: string): Promise<void>
```

**型定義**:
```typescript
interface NightContentItem {
  id: string
  trivia: string
  keywords: string[]              // 会話ログ由来キーワード（本番）
  generatedAt: Date
  isPersonalized: boolean
}
```

---

## COMP-05: VRMRenderer

```typescript
// VRMモデルのロード
loadVRMModel(modelId: string): Promise<VRMModel>

// リップシンクの更新（毎フレーム）
updateLipSync(frames: LipSyncFrame[]): void

// 表情モーフの適用
applyEmotion(signal: EmotionSignal): void

// アイドルアニメーションの開始/停止
setIdleAnimation(active: boolean): void
```

---

## COMP-08: MistakeEngine

```typescript
// モーニングコール用ミスパターン選択
selectMorningPattern(context: MorningCallContext): MistakePattern

// 注文用ミスロジック適用
applyOrderMistake(order: OrderDraft, context: MistakeContext): OrderDraft

// 連続ミス抑制チェック
shouldSuppressMistake(consecutiveCount: number): boolean
```
