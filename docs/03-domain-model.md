# ドメインモデル v0.1

## 1. モデル化の方針

MVPの中心は次の4オブジェクトです。

```text
BehaviorTarget
BehaviorState
InterventionTemplate
QuestResult
```

実装では、オンボーディング回答を保持する `BehaviorProfile` と、割り当てを表す `Quest` を加えます。

## 2. 関係

```text
User
 └─ BehaviorTarget
     ├─ BehaviorProfile
     ├─ BehaviorState
     └─ Quest *
         └─ QuestResult ?

InterventionTemplate
 └─ Quest *
```

## 3. Entity

### BehaviorTarget

ユーザーが変えたい対象と希望を表します。

```text
id
userId
type: youtube
desiredChange: unintentional_open | long_session | high_frequency | general_control | other
status: active | paused | archived
createdAt
```

MVPは1ユーザー1件のactive targetに限定してよいものの、型としては複数を許容します。

### BehaviorProfile

ユーザーが明示した、比較的安定した情報です。

```text
behaviorTargetId
problemPattern
triggers[]
motivation
interactionPreference
safetyLevel
updatedAt
```

### BehaviorState

観察や結果から推定する、変化する状態です。各値を `value` と `confidence` に分けます。

```text
automaticity
awareness
urgeControl
stoppingControl
selfEfficacy
motivation
receptivity
updatedAt
```

値の範囲は `0.0...1.0`。confidenceが低いことと、状態値が低いことは別です。

```json
{
  "automaticity": { "value": 0.85, "confidence": 0.75 },
  "awareness": { "value": 0.35, "confidence": 0.55 },
  "urgeControl": { "value": 0.50, "confidence": 0.20 },
  "stoppingControl": { "value": 0.50, "confidence": 0.20 },
  "selfEfficacy": { "value": 0.50, "confidence": 0.20 },
  "motivation": { "value": 0.85, "confidence": 0.80 },
  "receptivity": { "value": 0.70, "confidence": 0.30 }
}
```

### InterventionTemplate

検証済みまたは検証対象の介入を管理するテンプレートです。

```text
id
version
name
family
targetMechanism
applicableTargets[]
interactionCost
difficulty
preconditions[]
safetyConstraints[]
instructionTemplate
outcomeDefinition
evidenceStatus
active
```

テンプレートはコードから分離し、選択理由を説明できるメタデータを持たせます。

### Quest

特定ユーザー・特定日に割り当てられた介入です。

```text
id
userId
behaviorTargetId
interventionTemplateId
templateVersion
title
instruction
reason
status
assignedAt
acceptedAt?
startedAt?
```

Quest文面は生成後のスナップショットを保存します。テンプレート更新で過去の表示が変わらないようにします。

### QuestResult

介入の実行と対象行動の結果を分けて保存します。

```text
questId
completion: success | partial | failed
interventionPerformed: boolean
behaviorPerformed: boolean | unknown
intentional: boolean | unknown
failureReason?
declaredIntention?
helpfulness?
burden?
createdAt
```

## 4. 重要な不変条件

1. `BehaviorState` の推定値には必ずconfidenceがある。
2. `Quest` は参照したInterventionTemplateのversionを保存する。
3. `QuestResult.interventionPerformed` と `behaviorPerformed` を同じ成功判定にしない。
4. ユーザー向けの「できなかった」と内部値 `failed` を分離する。
5. 安全ゲートを通らない対象に通常Questを割り当てない。
6. 推薦理由を再現できる入力スナップショットまたはdecision logを残す。

## 5. 状態遷移

### Quest

```text
assigned → accepted → started → completed
                              ├─ success
                              ├─ partial
                              └─ failed
```

`skipped` と `expired` は実装時に必要になる可能性がありますが、Vertical Sliceでは省略できます。

### User Journey

```text
newUser
  → onboarding
  → activeExperiment
  → waitingForResult
  → resultRecorded
  → stateUpdated
  → nextQuestReady
  → activeExperiment
```

## 6. State Update例

Day 1で目的は決められたが、別の動画へ移り続けた場合:

```text
観察:
  Awareness介入は実行できた
  stopping controlの課題が示唆された

更新:
  awareness.value ↑
  awareness.confidence ↑
  stoppingControl.value ↓
  stoppingControl.confidence ↑
```

MVPでは複雑なベイズ更新を行わず、明示的なルールとテスト可能な差分で更新します。

## 7. 実装境界

推奨するProtocol相当:

```text
BehaviorProfileRepository
BehaviorStateRepository
InterventionTemplateRepository
QuestRepository
QuestResultRepository

BehaviorStateEstimating
InteractionPlanning
InterventionSelecting
QuestGenerating
```

永続化技術やLLM SDKをDomain層の型へ漏らさないようにします。

