# 指標と検証計画

## 1. MVPの評価方針

最初から「利用時間が何分減ったか」だけを第一KPIにしません。まず、適応ループが使われるかを確認します。

### Primary

- `Onboarding Completion Rate`
- `Day 1 Quest Acceptance Rate`
- `Day 1 Quest Execution Rate`
- `Result Input Rate`
- `Day 2 Return Rate`

特に `Result Input Rate` と `Day 2 Return Rate` を優先します。ここが低い場合、推薦精度より先にUXを改善します。

### Secondary

- Onboarding所要時間
- Quest開始までの時間
- Result入力所要時間
- Skip Rate
- Free-text Usage Rate
- Additional Question Rate
- Quest別のcompletion分布
- burden回答がある場合の分布

## 2. プロダクト固有の将来指標

### Intentional Action Rate

```text
意図を持って対象行動を行った回数
÷ 記録された対象行動回数
```

### Choice Point Creation Rate

```text
一度立ち止まってから選択した回数
÷ 記録された衝動または対象行動回数
```

### Intervention Execution Rate

対象行動をしなかった割合ではなく、Questで定義した介入を実行できた割合です。

これらは自己申告データだけでは測定誤差が大きいため、MVPでは探索指標として扱います。

## 3. 最小イベント

| Event | 主な属性 |
| --- | --- |
| `onboarding_started` | timestamp |
| `onboarding_question_answered` | questionId, inputType, skipped, elapsedMs |
| `onboarding_completed` | questionCount, totalElapsedMs |
| `quest_assigned` | questId, interventionId, selectorVersion |
| `quest_accepted` | questId |
| `quest_started` | questId |
| `quest_result_recorded` | questId, completion, failureReason? |
| `state_updated` | updateRuleVersion, changedDimensions[] |
| `next_quest_assigned` | questId, previousQuestId |
| `app_opened` | journeyState |

自由入力本文、視聴内容、詳細な感情は分析イベントへ含めません。

## 4. 初期テストの問い

1. 初回説明から「禁止アプリではない」と理解できるか。
2. 6問を負担に感じず完了できるか。
3. `決めずに進む` がChoiceの感覚に寄与するか、単なる離脱点になるか。
4. Resultの3択は迷わず答えられるか。
5. Day 2の理由文から「昨日の結果が反映された」と理解できるか。
6. 「できなかった」日でも再利用する気持ちが残るか。

## 5. 定性テスト

少人数テストでは、操作後に次を確認します。

- このアプリは何を助けるものだと感じたか。
- Questは命令、助言、実験のどれに感じたか。
- どの入力が面倒だったか。
- 結果選択で責められていると感じたか。
- Day 2の提案理由に納得できたか。
- 保存したくない情報があったか。

## 6. 判断基準の設定方法

最初の数名で数値目標を固定しません。イベント定義と操作フローを安定させた後、テスト母数・観察期間・許容離脱を決めてGo / Iterate / Stop基準を設定します。

利用時間削減や健康アウトカムを製品効果として主張するには、MVPの利用指標とは別の検証設計が必要です。

## 7. B2B2C向け指標の境界

スポンサー向け指標は、個人の改善を評価するものではなく、プログラムが利用されているかを集計で示すものに限定します。

- 契約席数、招待数、参加開始数
- 十分な母数がある場合の継続利用率
- 十分な母数がある場合のQuest実施率
- 任意回答によるwellbeing / time-use自己評価の集計

個人別の対象行動、利用時間、Quest結果、ValueTarget、自由入力はスポンサーへ提供しません。小さなcohortはReportingPolicyにより抑止し、集計利用には目的別Consentを要求します。
