# 指標と検証計画 v0.2

## 1. MVPの評価方針

最初から「利用時間が何分減ったか」を第一KPIにしません。まず、利用直前のアプリ介入なしで反復練習と定時振り返りが成立するかを確認します。

### Primary

- `Onboarding Completion Rate`
- `Scheduled Reflection Completion Rate`
- `Quest Recall Rate`
- `In-context Practice Rate`
- `Practice Block Completion Rate`
- `Day 2 / Day 3 Return Rate`

特に `Scheduled Reflection Completion Rate` と `Quest Recall Rate` を優先します。ここが低い場合、推薦精度より先に通知、Questコピー、負担、練習期間を改善します。

### 定義

```text
Scheduled Reflection Completion Rate =
  完了した定時Reflection数
  ÷ 回答可能だったReflection数

Quest Recall Rate =
  practiced + remembered_afterward
  ÷ no_opportunityを除くReflection回答数

In-context Practice Rate =
  practiced
  ÷ no_opportunityを除くReflection回答数

Practice Block Completion Rate =
  Block Reviewへ到達したPractice Block数
  ÷ 開始したPractice Block数
```

`no_opportunity` と未回答を同じにしません。通知を開かなかったことから、YouTube利用やQuest実行の有無を推定しません。

### Secondary

- Onboarding所要時間
- Reflection入力所要時間
- 通知からReflection開始までの時間
- Notification Permission Rate
- Reminder Time Change Rate
- Skip / Snooze Rate
- `no_opportunity` Rate
- Awareness Timing分布
- observedTrigger分布
- 同一Blockの継続日数・有効練習回数
- Block別のhelpfulness / burden

## 2. プロダクト固有の探索指標

### Awareness Observation Rate

```text
not_noticed / unknown以外のAwareness Timingが記録された回数
÷ no_opportunityを除くReflection回答数
```

### Awareness Timing Forward Shift

同じPractice Block内で、気づいた位置が行動の後から前へ移ったかを探索的に見ます。

```text
after_viewing
  < during_viewing
  < after_opening
  < while_opening
  < before_opening
  < trigger_before_action
```

単純な初回・最終回比較だけで効果を断定しません。利用機会、回想誤差、日ごとの状況差があるため、個人内の観察系列として扱います。

### Choice Point Creation Rate

```text
気づいた後に、見る／見ない／後にする等を意識的に選んだ自己申告回数
÷ 気づきを記録した回数
```

### Intervention Execution Rate

対象行動をしなかった割合ではなく、Questで定義した練習を実行できた割合です。

これらは定時の自己申告データで測定誤差が大きいため、MVPでは探索指標として扱います。利用時間削減や臨床的改善を示す指標として使いません。

## 3. 最小イベント

| Event | 主な属性 |
| --- | --- |
| `onboarding_started` | timestamp |
| `onboarding_question_answered` | questionId, inputType, skipped, elapsedMs |
| `onboarding_completed` | questionCount, totalElapsedMs |
| `reflection_schedule_set` | localHour, notificationPermission |
| `practice_block_assigned` | blockId, interventionId, selectorVersion, minimumPracticeCount |
| `daily_quest_presented` | blockId, questId, practiceDay |
| `daily_reflection_started` | questId, entryPoint, reminderDelayMs? |
| `daily_reflection_completed` | questId, practiceOutcome, awarenessTiming, observedTrigger?, elapsedMs |
| `practice_observation_added` | blockId, observationCount, validPracticeCount |
| `practice_block_reviewed` | blockId, reviewReason, nextDecision |
| `practice_block_completed` | blockId, validPracticeCount, progressionPolicyVersion |
| `next_block_assigned` | blockId, previousBlockId |
| `app_opened` | journeyState |

自由入力本文、視聴内容、詳細な感情、通知本文、Personal IDは分析イベントへ含めません。`awarenessTiming` と `observedTrigger` は本人領域の機微データであり、端末外分析へ送る場合は別途目的・同意・保持期間を決めます。

## 4. 初期テストの問い

1. 初回説明から「禁止アプリではない」と理解できるか。
2. アプリを先に開く必要がないと理解できるか。
3. 同じQuestを繰り返す理由に納得できるか。
4. 日中にアプリを使わなくてもQuestを思い出せるか。
5. 定時通知の時刻と頻度は負担にならないか。
6. Awareness Timingの選択肢を迷わず選べるか。
7. 後から気づいた場合も責められていないと感じるか。
8. Block Reviewから「練習で分かったこと」を理解できるか。
9. 次のスキルへ進む／同じスキルを続ける選択に納得できるか。

## 5. 定性テスト

少人数テストでは、操作後に次を確認します。

- このアプリは何を助けるものだと感じたか。
- Questは命令、助言、練習、実験のどれに感じたか。
- Questを思い出したのはどんな状況だったか。
- 思い出せなかった日は、何が合図になればよかったか。
- 定時の振り返りで一日の場面を思い出せたか。
- 正確に答えられないと感じた項目はあったか。
- 同じQuestの反復は安心、退屈、負担のどれに近かったか。
- Awareness Timingの変化は自分でも意味があると感じたか。
- 保存したくない情報があったか。

## 6. 判断基準の設定方法

最初の数名で有効性の数値目標を固定しません。まず次を確認します。

- Questの意味を誤解せず、現実の場面で少なくとも複数回試せる。
- 定時Reflectionから、無理なくAwareness Timingを回答できる。
- 同じスキルの反復とBlock Reviewが学習として理解される。
- 通知、回答、コピーが羞恥や強制感を生まない。

イベント定義と操作フローが安定した後、テスト母数、観察期間、許容離脱、Go / Iterate / Stop基準を事前登録します。

利用時間削減、依存症改善、健康アウトカムを製品効果として主張するには、MVPの利用指標とは別の比較可能な検証設計と専門家レビューが必要です。

## 7. B2B2C向け指標の境界

スポンサー向け指標は、個人の改善や気づきの早さを評価するものではなく、プログラムが利用されているかを集計で示すものに限定します。

- 契約席数、招待数、参加開始数
- 十分な母数がある場合の継続利用率
- 十分な母数がある場合のReflection参加率、Practice Block到達率
- 任意回答によるwellbeing / time-use自己評価の集計

個人別のAwareness Timing、トリガー、対象行動、利用時間、QuestResult、ValueTarget、自由入力はスポンサーへ提供しません。小さなcohortはReportingPolicyにより抑止し、集計利用には目的別Consentを要求します。
