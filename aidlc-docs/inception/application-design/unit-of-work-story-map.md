# Unit of Work — ストーリーマッピング

## ストーリー → Unit マッピング

| ストーリーID | タイトル | 担当Unit | 優先度 |
|---|---|---|---|
| US-00-01 | パートナーの名前をつける | Unit-0: Setup | 🟠 |
| US-00-02 | 声・方言を選ぶ | Unit-0: Setup | 🟠 |
| US-00-03 | 性格・見た目を設定する | Unit-0: Setup | 🟠 |
| US-00-04 | パートナーの初回挨拶を受ける | Unit-0: Setup + Unit-1 | 🟠 |
| US-01-01 | 音声でパートナーに話しかける | Unit-1: ConversationEngine | 🔴 |
| US-01-02 | VRMモデルがリアクションする | Unit-1: ConversationEngine | 🔴 |
| US-01-03 | 方言で話してもらう | Unit-1: ConversationEngine | 🔴 |
| US-01-04 | 過去の会話を覚えていてもらう | Unit-1: ConversationEngine | 🔴 |
| US-01-05 | LLMプロバイダーを切り替えられる | Unit-X: SharedInfrastructure | 🔴 |
| US-02-01 | 「なんか頼んで」と言うだけで注文される | Unit-2: AutoOrderMock | 🟡 |
| US-02-02 | バックグラウンドでBotがダミーサイトを自動操作する | Unit-2: AutoOrderMock | 🟡 |
| US-02-03 | Botがミスをして謝罪する | Unit-2: AutoOrderMock + Unit-X（MistakeEngine） | 🟡 |
| US-03-01 | 設定時刻にBotから起こしてもらう | Unit-3: MorningCall | 🟡 |
| US-03-02 | 重要な日は確実に起こしてもらう | Unit-3: MorningCall | 🟡 |
| US-03-03 | たまにBotが寝坊・ミスをする | Unit-3: MorningCall + Unit-X（MistakeEngine） | 🟡 |
| US-03-04 | 睡眠7時間を勧めてもらう | Unit-3: MorningCall + Unit-1 | 🟡 |
| US-04-01 | 毎晩22:00にBotがトリビアを話しかけてくる | Unit-4: NightContent | 🟡 |
| US-04-02 | お姉さん風口調でトリビアを聞く | Unit-4: NightContent | 🟡 |

---

## Unit別ストーリー数

| Unit | ストーリー数 | 主要ストーリー |
|---|---|---|
| Unit-X: SharedInfrastructure | 1 | US-01-05（LLM切り替え） |
| Unit-0: Setup | 4 | US-00-01〜04 |
| Unit-1: ConversationEngine | 4 | US-01-01〜04 |
| Unit-2: AutoOrderMock | 3 | US-02-01〜03 |
| Unit-3: MorningCall | 4 | US-03-01〜04 |
| Unit-4: NightContent | 2 | US-04-01〜02 |
| **合計** | **18** | |

---

## 受け入れ基準カバレッジ

| Unit | 受け入れ基準数 | セキュリティ関連AC | PBT関連AC |
|---|---|---|---|
| Unit-X | 4 | SECURITY-03・04・05・11・15 | PBT-09（フレームワーク選定） |
| Unit-0 | 15 | SECURITY-05（バリデーション） | — |
| Unit-1 | 22 | SECURITY-03（ログ）, SECURITY-15（エラー） | PBT-02・03（MistakeEngine・記憶） |
| Unit-2 | 15 | SECURITY-05（入力）, SECURITY-15（エラー） | PBT-02（シリアライズ） |
| Unit-3 | 14 | SECURITY-15（エラー） | PBT-02・03（MistakeEngine） |
| Unit-4 | 8 | — | — |

---

## 開発フェーズ別ストーリー割り当て

### Phase 1: SharedInfrastructure（前提）
- US-01-05: LLMプロバイダー切り替え基盤

### Phase 2: Setup + ConversationEngine（順次）
- US-00-01〜04: パートナー作成体験
- US-01-01〜04: 音声会話・VRM・記憶

### Phase 3: 並行開発
**Unit-2 (AutoOrderMock)**:
- US-02-01〜03: 自動注文・ミスハプニング

**Unit-3 (MorningCall)**:
- US-03-01〜04: モーニングコール・ミスパターン

**Unit-4 (NightContent)**:
- US-04-01〜02: 夜お姉さんタイム
