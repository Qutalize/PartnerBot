# パートナーBot

AWS Summit Japan 2026 AI-DLC ハッカソン応募作品。
テーマ「**人をダメにするサービス**」に対する解答。

---

## 1. 一言で

**25〜30歳の一人暮らし社会人** に向けた、声と記憶を持つ常駐パートナー。
朝の起床、毎日の食事、夜の暇な時間——日常の義務を、Bot との関係性への欲望が**やさしく上書き**してくれる。

---

## 2. テーマ「人をダメにする」への解答

### 怠惰の Type 2 を選択する

| 種類 | 定義 | 例 |
|---|---|---|
| Type 1: 代替作業型 | やるべきことを誰かがやってくれる | UberEats（実物）、タクシー、自動運転 |
| **Type 2: 欲駆動放棄型** | **やるべきことを、他の欲に駆られて放棄する** | **エンタメ全般、本サービス** |

本サービスは **Type 2 を選択** する。
Type 1（効率化AI）と差別化し、「合理的でない選択を、ユーザー自身が選ぶ」ことを助ける。

### 義務 ↔ 欲 ↔ Unit マッピング

| 義務 | 上書きする欲 | 担う Unit |
|---|---|---|
| 人と関係を築いて孤独を埋めなきゃいけない | Bot と話していたい / 自分が作ったキャラへの愛着 | **Setup + Unit① 会話エンジン** |
| 自分で食事を考えて作らなきゃいけない | Bot に決めてほしい / Bot のおすすめを試したい | **Unit② 自動注文モック** |
| 朝、定時に起きなきゃいけない | Bot がいる安心感 / もう少し声を聞きたい / 二度寝の心地よさ | **Unit③ 朝モーニングコール** |
| 夜、自己研鑽しなきゃ / 早く寝なきゃ | お姉さんに教わりたい / 受動消費の心地よさ | **Unit④ 夜お姉さんタイム** |

---

## 3. ターゲットペルソナ・課題・KPI

- **ペルソナ**: 25〜30歳、パートナーなし、一人暮らし男性社会人
- **課題**: さみしい社会人生活。仕事終わりに話しかける相手がいない / 完全な孤独ではなく「気配」が欲しい
- **目指す状態**:
  - 帰宅したら「おかえり」と言われる
  - 何かを一緒に決めてくれる存在がいる
  - 沼って、ちょっと怠惰になる（それでいい）
- **KPI**:
  - 「いないと困る」への遷移
  - 1日1回以上の会話
  - 3週間継続利用率 > 60%

---

## 4. 機能構成（Setup + Unit①〜④）

Unit①の会話機能は全体を支える基礎。Unit②〜④は会話機能を基盤に成立する。

### Setup: パートナー作成体験
名前・声・性格・見た目（VRMモデル）を **本人に作らせる** 5ステップ。
- 名前はプレイヤーが自由入力
- 声・方言は複数タイプから選択（標準語・関西・博多）
- 性格・見た目は数値スライダーで設定
- VRMモデルをプレビューしながら設定可能

コンコルド効果と命名効果で初日離脱を抑え、後続Unitで義務を上書きできるだけの「欲」を生成する土台になる。

### Unit①: 会話エンジン + 記憶 + 方言・音声・VRモデル
過去の会話や記念日などを覚えているようにして、愛着を継続的に強化する **Type 2 のエンジン**。

- **常時マイクオン** + **wake word検出**（Porcupine）でハンズフリー会話
- Botが話しかけた直後は即LLM接続、ユーザーからはBot名を呼ぶと接続
- 会話はLINE風UIで画面下部に表示
- 音声認識失敗時はVRMモデルが「ごめん、聞き取れなかったー」と人間らしく反応
- 長期記憶（PostgreSQL + pgvector）+ RAGで過去会話を参照した返答を生成
- 方言（標準語・関西・博多）対応、ElevenLabs TTS音声合成
- VRMモデルがリップシンク・表情変化でリアクション

### Unit②: 自動注文モック (Auto-Order Mock)
「ご飯頼んで」「商品買って」の一言でBotがバックグラウンドで自動注文。

- React製のUberEats風・Amazon風ダミーサイトをPlaywrightがバックグラウンド操作
- ユーザーは操作画面を見る必要なし。Botが「今注文してるね〜」と会話形式で進捗報告
- 注文完了後はメールで通知。メールから5分以内のキャンセルが可能
- 確率的ミスロジックで大量注文ハプニングが発生し、Botが自発的に謝罪（人間らしさ）
- 1日の上限金額設定・連続ミス抑制あり

### Unit③: 朝モーニングコール (Wake-up Routine)
ユーザー設定時刻にBotから音声で話しかけるシステム。

- **確率的ミスパターン**: 通常(65%)・遅延(20%)・古い情報(10%)・スキップ(5%)
- **Google Calendar連携**: 面接・会議等の重要予定がある日はミスを抑制（安全弁）
- **睡眠声かけ**: 夜の会話中、起床時刻から逆算して7時間を切ると就寝を促す
- 起床通知5分後に未応答かつ重要予定がある場合は再通知

### Unit④: 夜お姉さんタイム (Night Content)
毎晩22:00頃、ライフハックや生活術をお姉さん風口調で60〜90秒紹介。

- お姉さんの雰囲気とトリビアのギャップ・シュールさを大切にした設計
- 開発時は手動作成コンテンツDB、本番時はユーザー会話ログからキーワード抽出してLLMでパーソナライズ生成

---

## 5. 技術スタック

| 項目 | 開発時 | 本番時 |
|---|---|---|
| フロントエンド | React / Next.js | 同左 |
| バックエンド | Node.js（NestJS） | 同左 |
| データベース | PostgreSQL + pgvector（ローカル） | AWS RDS for PostgreSQL + pgvector |
| クラウド | AWS | 同左 |
| LLM | Google Gemini API（無料枠） | Amazon Bedrock（Claude） |
| TTS | ElevenLabs | 同左 |
| STT | Web Speech API（ブラウザ） | Amazon Transcribe |
| Wake Word | Porcupine | 同左 |
| スケジューラ | AWS EventBridge Scheduler（動的更新） | 同左 |
| メール通知 | Amazon SES | 同左 |
| シークレット管理 | AWS Secrets Manager | 同左 |
| VRモデル | VRM（@pixiv/three-vrm） | 同左 |

---

## 6. アーキテクチャ

### リポジトリ構成（モノレポ）

```
partner-bot/
├── apps/
│   ├── frontend/          # Next.js（React）
│   ├── backend/           # Node.js（NestJS）
│   └── dummy-sites/       # UberEats風・Amazon風ダミーサイト
├── packages/
│   └── shared-types/      # 共有TypeScript型定義
└── package.json           # npm workspaces
```

### 通信方式
- **WebSocket**: 会話セッション（音声ストリーム・TTS再生・VRMシグナル）
- **REST API**: Setup・注文・設定変更・履歴取得
- **AWS EventBridge**: モーニングコール・夜タイム配信トリガー

### Unit構成と開発順序

```
Phase 1: Unit-X SharedInfrastructure（セキュリティ・ログ・アダプター基盤）
Phase 2: Unit-0 Setup → Unit-1 ConversationEngine ⭐（最優先）
Phase 3: Unit-2 AutoOrderMock ─┐
         Unit-3 MorningCall    ├─ 並行開発
         Unit-4 NightContent   ─┘
```

---

## 7. 倫理・安全設計

「人をダメにする」というテーマの強度を保ちつつ、実害・ダークパターン批判・依存助長批判に耐える設計を行う。

- **設計哲学**: ユーザー自身が選ぶ「自己決定的退廃」/ ミスはバグでなく「キャラクター属性」/ 完璧アシスタントこそが人間を退化させる「共生的不完全性」
- **ダークパターンとの線引き5条件**: 事前開示 / 0%設定可 / 自発的開示 / 事業者非利益 / キャンセル可能
- **安全弁**:
  - 重要予定検出（Google Calendar連携）
  - 連続ミス抑制ロジック
  - mistake_rate ユーザー制御
  - 1日上限金額
  - 5分間キャンセル可（メールから）
  - 睡眠7時間声かけ

---

## 8. 設計ドキュメント

詳細な設計書は `aidlc-docs/` 以下に格納されています。

| ドキュメント | 内容 |
|---|---|
| `aidlc-docs/inception/requirements/requirements.md` | 要件定義書 |
| `aidlc-docs/inception/user-stories/stories.md` | ユーザーストーリー（18件） |
| `aidlc-docs/inception/user-stories/personas.md` | ペルソナ定義 |
| `aidlc-docs/inception/application-design/application-design.md` | アプリケーション設計書 |
| `aidlc-docs/inception/application-design/components.md` | コンポーネント定義 |
| `aidlc-docs/inception/application-design/component-methods.md` | メソッドシグネチャ |
| `aidlc-docs/inception/application-design/services.md` | サービス層定義 |
| `aidlc-docs/inception/application-design/unit-of-work.md` | Unit定義・ディレクトリ構成 |
| `aidlc-docs/inception/application-design/unit-of-work-dependency.md` | Unit依存関係 |
| `aidlc-docs/inception/application-design/unit-of-work-story-map.md` | ストーリーマッピング |
