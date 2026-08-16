# MVP Vertical Slice v0.2

## 1. 範囲

```text
Onboarding
  → BehaviorProfile / Initial BehaviorState
  → Awareness Practice Block
  → Day 1 Quest提示
  → 日常で本人が練習（アプリ操作は不要）
  → 定時のDaily Reflection
  → Day 2も同じQuestを継続
  → 最低練習回数後のBlock Review
  → 次のPractice Block候補
```

この1本が端から端まで動くことを、最初の完成条件とします。YouTube起動の検知や、利用直前のアプリ表示は完成条件に含めません。

## 2. シナリオ

### 前提ユーザー

```text
target: YouTube
desiredChange: unintentional_open
problemPattern: automatic_open
triggers: after_work, fatigue
motivation: high
interactionPreference: low
reflectionTime: 21:00 local time
```

### Step 1: Welcome

表示:

> 自分の習慣に、少しずつ早く気づけるように。やめることを強制するアプリではありません。毎日ひとつの小さな練習と短い振り返りで、自分で選べる瞬間を増やしていきます。

CTA: `はじめる`

### Step 2: Onboarding

再利用可能な質問画面で、Target、Desired Change、Problem Pattern、Trigger、Motivation、Interaction Preferenceを取得します。

完了時に `BehaviorProfile` と、confidence付きの `BehaviorState` を保存します。事前回答だけで高度な介入へ進めず、最初は低負担なAwareness Blockを作成します。

### Step 3: Reflection Time

提案時刻を表示し、本人が変更できるようにします。

> 毎日いつ振り返りますか？ 30秒ほどで終わります。

- 時刻を選ぶ
- 通知は後で設定する

通知許可がない場合も、ホーム画面から当日のReflectionを開始できます。

### Step 4: Practice Block assignment

入力:

```text
problemPattern = automatic_open
interactionPreference = low
practiceHistory = empty
```

選択:

```text
interventionTemplate = notice_opening
minimumPracticeCount = 3
```

### Step 5: Day 1 Quest

表示:

> 今日の練習
>
> YouTubeを開こうとしていることに気づいたら、心の中で「いまYouTubeを開こうとしている」と言ってみよう。そのまま開いて構いません。開いた後に気づいても練習になります。

補足:

- Choice Gapを先に開く必要はありません。
- 日中の入力はありません。
- 夜に短く振り返ります。

CTA: `今日やってみる`

### Step 6: Daily life

ユーザーは通常どおり端末を使います。Questを思い出した場合は心の中でラベリングします。

アプリは次を要求しません。

- YouTube起動前にChoice Gapを開く。
- 視聴目的を入力してからYouTubeへ遷移する。
- 視聴を我慢する。
- 気づくたびに即時記録する。

### Step 7: Scheduled Daily Reflection

設定時刻に、機微な内容を含まない通知を表示します。

> 今日の小さな練習を振り返りませんか？

標準質問:

1. 今日、この練習を思い出した？
2. 一番早く気づけたのはいつ？
3. 一番印象に残ったきっかけは？

Day 1の保存例:

```json
{
  "questId": "q_001",
  "practiceBlockId": "pb_001",
  "practiceOutcome": "remembered_afterward",
  "interventionPerformed": true,
  "behaviorPerformed": true,
  "intentional": "unknown",
  "awarenessTiming": "during_viewing",
  "observedTrigger": "fatigue",
  "createdAt": "2026-08-15T21:03:00+09:00"
}
```

「見ている途中で気づいた」ことを失敗として表示しません。

### Step 8: Day 2 continuation

`practiceCount = 1` で最低練習回数に達していないため、同じBlockを継続します。

表示:

> 昨日は、見ている途中で気づけました。今日も同じ「気づく」練習です。繰り返して、気づくタイミングの変化を見ていきます。

コピーは観察結果を反映できますが、InterventionTemplateは変えません。

### Step 9: no opportunity branch

Day 2の結果が `no_opportunity` の場合:

```text
practiceCount: 1のまま
calendarDay: 2へ進む
Practice Block: 継続
```

YouTubeを使わなかったこと自体へ報酬を与えず、練習機会がなかった日として扱います。

### Step 10: Block Review

3回の有効な練習結果が揃った時点でReviewを表示します。

例:

```text
practiceCount: 3
observed timings:
  during_viewing
  after_opening
  while_opening
common trigger: fatigue
```

表示:

> 3回の練習で、「見ている途中」から「開こうとしている途中」まで気づく場面が見つかりました。次は、気づいたときに一呼吸入れる練習を試せます。

次候補: `pause_one_breath`

CTA:

- `次の練習へ`
- `同じ練習を続ける`

本人が同じ練習の継続を選べるようにします。

## 3. 必要画面

1. Welcome
2. Onboarding Question（単一選択・複数選択を再利用）
3. Reflection Time Setup
4. Practice Home
5. Quest Detail
6. Daily Reflection
7. Block Review

## 4. 最小コンポーネント案

```text
AppRootView

OnboardingFlow
 ├─ WelcomeView
 ├─ SingleSelectQuestionView
 ├─ MultiSelectQuestionView
 └─ ReflectionTimeView

PracticeFlow
 ├─ PracticeHomeView
 ├─ QuestDetailView
 ├─ DailyReflectionView
 └─ BlockReviewView
```

## 5. Application Services

```text
OnboardingService
BehaviorStateEstimator
PracticeBlockService
InterventionSelector
ProgressionPolicy
QuestGenerator
QuestResultService
ReminderScheduler
AIService (optional boundary)
```

## 6. Acceptance Criteria

- 初回起動から6問以内の標準フローでプロフィールを保存できる。
- ユーザーがReflection時刻を設定または後回しにできる。
- `automatic_open` から最初に `notice_opening` が選ばれる。
- Day 1 Questが、アプリを先に開くことやYouTubeを我慢することを要求しない。
- 定時Reflectionで `practiceOutcome`、`awarenessTiming`、`observedTrigger` を保存できる。
- Day 1の結果後もDay 2に同じInterventionTemplateが継続される。
- `no_opportunity` が最低練習回数へ算入されない。
- `not_remembered` が最低練習回数へ算入されない。
- 1日の `not_remembered` だけで別のスキルへ切り替わらない。
- 3回の有効な練習後にBlock Reviewを表示できる。
- Reviewから次のBlockへ進むか、同じBlockを続けるか選べる。
- 通知許可がなくてもアプリ内からReflectionを完了できる。
- アプリ再起動後もPractice Blockと当日の進行状態が復元される。
- AIServiceなしでも上記シナリオが完走する。

## 7. Vertical Sliceの非対象

- YouTube起動や利用時間の自動取得
- 利用直前の通知・オーバーレイ
- 通知時刻の最適化
- 日中のEcological Momentary Assessmentを必須にすること
- 複数Target
- 週次レビュー
- サーバー同期
- 認証
- 課金
- 本格的なゲーミフィケーション
