# MVP Vertical Slice v0.1

## 1. 範囲

```text
Onboarding
  → BehaviorProfile / Initial BehaviorState
  → Day 1 Quest
  → Quest実行
  → Result Check-in
  → BehaviorState更新
  → Day 2 Recommendation
```

この1本が端から端まで動くことを、最初の完成条件とします。

## 2. シナリオ

### 前提ユーザー

```text
target: YouTube
desiredChange: unintentional_open
problemPattern: automatic_open
triggers: after_work, fatigue
motivation: high
interactionPreference: low
```

### Step 1: Welcome

表示:

> 自分の習慣を、少しずつコントロールできるように。やめることを強制するアプリではありません。毎日ひとつ、小さな実験を試して、自分に合う方法を見つけていきます。

CTA: `はじめる`

### Step 2: Onboarding

再利用可能な質問画面で、Target、Desired Change、Problem Pattern、Trigger、Motivation、Interaction Preferenceを取得します。

完了時に `BehaviorProfile` と、confidence付きの `BehaviorState` を保存します。

### Step 3: Day 1 selection

入力:

```text
problemPattern = automatic_open
interactionPreference = low
```

選択:

```text
interventionTemplate = intentional_opening
```

### Step 4: Day 1 Quest

表示:

> 「何を見る？」を決めてからYouTubeを開いてみよう。見ることを我慢する必要はありません。所要時間：約10秒。

CTA: `やってみる`

### Step 5: Quest execution

質問:

> 何を見る？

入力手段:

- 一言入力
- 音声入力（実装は後回しでもよい）
- `決めずに進む`

保存例:

```json
{
  "questId": "q_001",
  "timestamp": "2026-08-10T19:30:00+09:00",
  "declaredIntention": "SwiftUIの解説",
  "interventionPerformed": true
}
```

CTA: `YouTubeを開く`

MVPではYouTubeから戻ったことを自動検知しなくてよく、再起動・foreground時に未回答Questがあれば結果画面を出します。

### Step 6: Result Check-in

質問:

> さっきのQuest、どうだった？

- できた
- 少しできた
- できなかった

`partial` の理由が `continued_browsing` の場合、次の状態を推定します。

```text
awareness: 改善の観察、confidence上昇
stoppingControl: 課題の示唆、confidence上昇
```

### Step 7: Day 2 selection

候補:

```text
intentional_opening     mechanismFit 0.60
stop_after_one_video    mechanismFit 0.90
pause_1_minute          mechanismFit 0.45
```

選択: `stop_after_one_video`

### Step 8: Day 2 Quest

表示:

> 昨日は「見るものを決める」ことはある程度できていました。ただ、その後ほかの動画へ移りやすそうです。
>
> 今日の実験：動画を1本見終わったら、一度YouTubeを閉じてみよう。また見たくなったら、もう一度開いて構いません。

ここで、前日の結果から次が選ばれたことをユーザーへ説明します。

## 3. 必要画面

1. Welcome
2. Onboarding Question（単一選択・複数選択を再利用）
3. Quest Home
4. Quest Detail
5. Quest Input
6. Result Check-in
7. Day 2 Quest（Quest Detailを再利用）

## 4. 最小コンポーネント案

```text
AppRootView

OnboardingFlow
 ├─ WelcomeView
 ├─ SingleSelectQuestionView
 ├─ MultiSelectQuestionView
 └─ ShortInputQuestionView

MainFlow
 ├─ HomeView
 ├─ QuestDetailView
 ├─ QuestExecutionView
 └─ QuestResultView
```

## 5. Application Services

```text
OnboardingService
BehaviorStateEstimator
InteractionPlanner
InterventionSelector
QuestGenerator
QuestResultService
AIService (optional boundary)
```

## 6. Acceptance Criteria

- 初回起動から6問以内の標準フローでプロフィールを保存できる。
- `automatic_open` からDay 1に `intentional_opening` が選ばれる。
- 視聴目的を入力せずに進める。
- foreground復帰時に未回答Result Check-inを表示できる。
- `partial + continued_browsing` からDay 2に `stop_after_one_video` が選ばれる。
- Day 2画面に、前日の観察を理由として表示する。
- アプリ再起動後も状態が復元される。
- AIServiceなしでも上記シナリオが完走する。

## 7. Vertical Sliceの非対象

- 実際のYouTube利用時間の取得
- 通知の最適化
- 複数Target
- 週次レビュー
- サーバー同期
- 認証
- 課金
- 本格的なゲーミフィケーション

