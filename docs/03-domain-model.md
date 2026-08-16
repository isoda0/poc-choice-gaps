# ドメインモデル v0.3 — B2C / B2B2C対応

## 1. 設計目標

MVPは個人向けYouTube習慣改善から始めますが、将来は企業・大学などが利用料を負担し、本人がアプリを使うB2B2Cモデルを想定します。

後の大きな手戻りを防ぐため、最初から次を分離します。

1. **本人の行動変容データ**
2. **企業・大学などのスポンサー関係**
3. **料金を負担する主体と、機能を使う主体**
4. **同意された目的と、組織向け集計レポート**
5. **減らしたい行動と、時間を使いたい価値ある活動**

最重要ルールは次です。

> `BehaviorTarget`、`BehaviorState`、`PracticeBlock`、`Quest`、`QuestResult`、`ValueTarget` に `organizationId` を持たせない。

企業が料金を払っても、個人の行動データの所有・管理主体が企業へ移るわけではありません。

## 2. Bounded Context

```text
┌────────────────────────────────────────────┐
│ Personal Behavior Change                   │
│ PersonalSpace, BehaviorTarget, ValueTarget │
│ BehaviorState, PracticeBlock               │
│ Quest, QuestResult                         │
└────────────────────────────────────────────┘
                 ▲ entitlement only
                 │
┌────────────────────────────────────────────┐
│ Sponsorship & Entitlements                 │
│ Organization, Program, Enrollment, Grant   │
└────────────────────────────────────────────┘
                 │ policy / consent
                 ▼
┌────────────────────────────────────────────┐
│ Privacy & Consent                          │
│ ConsentRecord, ReportingPolicy             │
└────────────────────────────────────────────┘
                 │ allowed aggregates only
                 ▼
┌────────────────────────────────────────────┐
│ Analytics & Sponsor Reporting              │
│ MetricDefinition, AggregateMetricSnapshot  │
└────────────────────────────────────────────┘

┌────────────────────────────────────────────┐
│ Commercial                                 │
│ CustomerAccount, Plan, Contract, SeatPool  │
└────────────────────────────────────────────┘
```

MVPで実装する中心は `Personal Behavior Change` です。他Contextは型と境界だけ先に決め、必要になるまでバックエンド実装を増やしません。

## 3. 全体関係

```text
Person
 └─ PersonalSpace
     ├─ BehaviorTarget *
     │   ├─ BehaviorProfile
     │   ├─ BehaviorState
     │   └─ PracticeBlock *
     │       └─ Quest *
     │           └─ QuestResult ?
     ├─ ValueTarget *
     │   └─ ValueActivity *
     └─ RecoveredTimeEstimate *

Organization
 └─ SponsoredProgram *
     ├─ ProgramInvitation *
     ├─ ProgramEnrollment * ── restricted link ── Person
     ├─ SeatPool
     └─ AggregateMetricSnapshot *

EntitlementGrant
 ├─ beneficiary: Person
 └─ source: IndividualSubscription | SponsoredProgram

InterventionTemplate
 └─ PracticeBlock *
     └─ Quest *
```

## 4. Personal Behavior Change Context

### Person

サービスを利用する自然人を表す安定した識別子です。認証手段やメールアドレスそのものをDomainの主体にしません。

```text
id
status: active | suspended | deleted
createdAt
```

認証ID、Apple Sign-In、企業SSOなどはIdentity層で `Person` に関連付けます。企業SSOで登録しても、企業がPersonを所有するモデルにはしません。

### PersonalSpace

本人だけが利用する行動変容データのAggregate Rootです。

```text
id
ownerPersonId
status: active | export_pending | deletion_pending | deleted
createdAt
```

初期実装は1 Person = 1 PersonalSpaceで構いません。Personal entityは `personId` を直接ばらまかず、原則 `personalSpaceId` を参照します。

### BehaviorTarget

本人が減らしたい／コントロールしたい行動です。

```text
id
personalSpaceId
type: youtube
desiredChange: unintentional_open | long_session | high_frequency | general_control | other
status: active | paused | archived
createdAt
```

MVPではactive targetを1件に制限できますが、モデル上は複数を許容します。

### ValueTarget

取り戻した時間や注意を、本人が何に使いたいかを表します。BehaviorTargetとは独立させます。

```text
id
personalSpaceId
category:
  technical_learning | certification | reading | exercise |
  sleep | rest | family | hobby | creative_work | other
description?
status: active | achieved | paused | archived
priority?
createdAt
updatedAt
```

重要なルール:

- 作成・変更・公開範囲を決めるのは本人だけ。
- 企業や大学が「資格勉強」を本人のValueTargetとして設定しない。
- スポンサーは個人のValueTargetを閲覧できない。
- 組織が提示できるのは任意の選択肢や教育コンテンツまでで、選択結果は個人データとする。

### BehaviorProfile

本人が明示した、比較的安定した情報です。

```text
behaviorTargetId
problemPattern
triggers[]
motivation
interactionPreference
safetyLevel
updatedAt
```

ValueTargetはBehaviorProfileへ埋め込みません。BehaviorTargetとValueTargetは、本人のPersonalSpace内で必要に応じて関連付けます。

### BehaviorState

複数回の観察から推定する変化する状態です。各値を `value` と `confidence` に分けます。

```text
behaviorTargetId
automaticity
awareness
urgeControl
stoppingControl
selfEfficacy
motivation
receptivity
updatedAt
estimatorVersion
```

値の範囲は `0.0...1.0`。confidenceが低いことと、状態値が低いことは別です。
1件のReflectionから値を大きく変更せず、主に観察数とconfidenceを更新します。

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

### AwarenessTiming

特定の利用場面で、本人がどの時点で自動的な行動に気づいたかを表す順序付きの観察値です。固定的な能力値ではありません。

```text
not_noticed
after_viewing
during_viewing
after_opening
while_opening
before_opening
trigger_before_action
unknown
no_opportunity
```

`unknown` は回想できない場合、`no_opportunity` は対象場面がなかった場合です。この2つはAwarenessの高低や進歩へ換算しません。

### InterventionTemplate

検証済みまたは検証対象の介入テンプレートです。

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
minimumPracticeCount
progressionCriteria[]
evidenceStatus
active
```

### PracticeBlock

同じスキルを複数回練習する単位です。毎日の目新しさではなく、反復とBlock Reviewを管理します。

```text
id
personalSpaceId
behaviorTargetId
interventionTemplateId
templateVersion
status: assigned | active | review_ready | completed | paused | abandoned
minimumPracticeCount
validPracticeCount
selectionReason
startedAt
reviewedAt?
completedAt?
```

状態遷移:

```text
assigned → active → review_ready → completed
              │          └────────→ active（本人が継続を選択）
              ├─ paused
              └─ abandoned
```

`validPracticeCount` は `practiced` と `remembered_afterward` だけを数えます。`not_remembered` と `no_opportunity` は除外します。Blockを自動完了させず、Reviewで本人に継続または次の練習を選べるようにします。

### Quest

Practice Blockから特定の日へ割り当てた練習のスナップショットです。同じInterventionTemplateから複数日のQuestを作れます。

```text
id
personalSpaceId
behaviorTargetId
practiceBlockId
interventionTemplateId
templateVersion
practiceDay
relatedValueTargetId?
title
instruction
reason
status
assignedAt
acceptedAt?
reflectionDueAt?
reflectedAt?
```

`relatedValueTargetId` は、本人の価値目標とQuestを結びつける場合だけ設定します。スポンサーのProgramを推薦理由として直接埋め込みません。

Questの状態遷移:

```text
assigned → accepted → waiting_for_reflection → reflected
                       └──────────────────────→ expired
```

### QuestResult

毎日の定時Reflectionで得た自己申告です。練習の想起、スキル実行、気づいた位置、対象行動の結果を分けます。

```text
questId
practiceBlockId
practiceOutcome:
  practiced | remembered_afterward | not_remembered | no_opportunity
interventionPerformed: boolean
behaviorPerformed: boolean | unknown
intentional: boolean | unknown
awarenessTiming: AwarenessTiming
observedTrigger?
barrier?
declaredIntention?
helpfulness?
burden?
createdAt
```

`practiceOutcome` は人の成功・失敗を評価する値ではありません。`behaviorPerformed == true` でも、気づきや介入を実行できていれば練習結果として記録します。

### RecoveredTimeEstimate

「取り戻した時間」は観測事実ではなく推定である場合が多いため、Value Objectではなく、算出根拠を持つ履歴Entityとして扱います。

```text
id
personalSpaceId
periodStart
periodEnd
estimatedMinutes
confidence: 0.0...1.0
method: self_report | baseline_delta | hybrid
methodVersion
inputCoverage
createdAt
```

表示では必ず「約」「推定」を伴わせます。企業向けには個人値を出さず、本人向けにも生産性や因果効果として断定しません。

### ValueActivity

本人が価値ある活動へ時間を使った記録です。

```text
id
valueTargetId
startedAt?
durationMinutes?
source: self_report | timer | estimate
note?
createdAt
```

`RecoveredTimeEstimate` と `ValueActivity` を直接1:1対応させません。「YouTubeを30分減らしたから勉強30分が生まれた」という因果は自動的には証明できないためです。

### State Update例

同じAwareness Blockで3回の有効な観察が得られた場合:

```text
観察:
  during_viewing
  after_opening
  while_opening
  共通トリガー: fatigue

更新:
  awareness.value: 小さく上げる
  awareness.confidence ↑
  automaticity.confidence ↑
  trigger observationを追加

次候補:
  pause_one_breath
```

1日の結果だけでは主要なBehaviorStateやPractice Blockを切り替えません。MVPでは複雑なベイズ更新を行わず、明示的なルールとテスト可能な差分で更新します。

## 5. Sponsorship & Entitlements Context

### Organization

料金を負担し、プログラムを提供する組織です。

```text
id
type: company | university | school | nonprofit | other
displayName
status: trial | active | suspended | ended
createdAt
```

`school` と未成年者対応は将来用のtypeです。年齢・保護者同意・教育機関特有の要件が整うまでProgramを有効化しません。

### SponsoredProgram

組織が提供する福利厚生・学生支援等の単位です。同じOrganizationが複数Programを持てます。

```text
id
organizationId
name
purpose: digital_wellbeing | self_management | education
status: draft | active | paused | ended
startsAt
endsAt?
planId
reportingPolicyId
createdAt
```

ProgramはPersonal Spaceの介入方針を所有しません。任意の教材やテーマを提示できますが、本人のBehaviorTarget、ValueTarget、QuestResultを読み書きしません。

### ProgramInvitation

スポンサーが利用資格を案内するための一時的な招待です。

```text
id
programId
deliveryTargetReference
status: pending | accepted | expired | revoked
expiresAt
createdAt
```

メールアドレス等は暗号化またはハッシュ化し、行動データのストアへ複製しません。

### ProgramEnrollment

本人がProgramへ参加している関係です。

```text
id
programId
participantId
status: invited | active | left | ended
joinedAt?
leftAt?
```

`participantId` はProgram内の仮名識別子です。`Person` との対応はアクセス制限された `EnrollmentIdentityLink` で管理し、スポンサー向け画面や分析テーブルへ出しません。

### EnrollmentIdentityLink

```text
enrollmentId
personId
createdAt
```

利用権の付与や本人のProgram一覧取得に必要な内部リンクです。スポンサー管理者からはアクセスできません。

### EntitlementGrant

「誰が払ったか」と「誰がPremiumを使えるか」を分離する中心Entityです。

```text
id
beneficiaryPersonId
planId
sourceType: individual_subscription | sponsored_program | trial | promotion
sourceId
validFrom
validUntil?
status: active | scheduled | expired | revoked
createdAt
```

本人が個人サブスクと企業Programの両方を持つ場合も、複数Grantを合成して `EffectiveEntitlement` を計算します。User/Personへ `isPremium` や機能フラグを直接保存しません。

### EffectiveEntitlement

Entityではなく、時点と複数Grantから計算するProjectionです。

```text
personId
features[]
effectiveFrom
effectiveUntil?
sources[]
calculatedAt
resolverVersion
```

退職・卒業・契約終了でSponsored Grantが失効しても、PersonalSpaceと履歴は削除しません。本人はFreeへ戻る、個人課金へ切り替える、または削除を選べます。

## 6. Privacy & Consent Context

### ConsentRecord

```text
id
personId
purpose:
  core_service | product_analytics | sponsor_aggregate_reporting | ai_personalization
policyVersion
status: granted | withdrawn
recordedAt
effectiveAt
```

同意は包括的なbooleanにせず、目的・versionごとに履歴として保持します。撤回は将来の処理を止め、既に作られた集計の削除可否はポリシーで明示します。

### ReportingPolicy

```text
id
version
allowedMetricIds[]
minimumCohortSize
allowedDimensions[]
smallCellSuppression: boolean
status: draft | active | retired
```

スポンサーとの契約よりもプライバシー上の制約が弱くならないよう、アプリ共通の最低基準を別途持ちます。

## 7. Analytics & Sponsor Reporting Context

### MetricDefinition

```text
id
version
name
description
calculation
requiredConsentPurpose
dataClassification
```

### AggregateMetricSnapshot

```text
id
programId
metricDefinitionId
metricVersion
periodStart
periodEnd
cohortKey?
value
unit
participantCount
reportingPolicyVersion
privacyStatus: publishable | suppressed_small_cohort | suppressed_policy
generatedAt
```

スポンサーへ提供可能な候補:

- 契約席数、招待数、参加開始数
- 十分な母数がある場合の継続利用率
- 集計されたQuest実施率
- 任意・匿名の「時間を有意義に使えている」自己評価

スポンサーへ提供しないもの:

- 個人別のBehaviorTarget、利用時間、Trigger、ValueTarget
- 個人別のQuest、Practice Outcome、Awareness Timing、自由入力、深夜利用
- 個人が特定できる小さなcohortや属性の組み合わせ
- 人事評価、学業評価、懲戒へ使える個人スコア

## 8. Commercial Context

将来の料金・契約の最小モデルです。行動変容Domainへ課金ロジックを混ぜません。

```text
CustomerAccount
  payerType: person | organization

Plan
  featureSetVersion
  pricingModel: individual | per_seat | annual_license

Contract / Subscription
  customerAccountId
  planId
  status
  term

SeatPool
  sponsoredProgramId
  purchasedSeats
  assignedSeats
```

価格はPlan/Contract側に置き、EntitlementGrantは利用可能性だけを表します。

## 9. 不変条件

1. Personal entityは `organizationId` を持たない。
2. スポンサー管理者がPersonal Repositoryへ到達するUse Caseを作らない。
3. スポンサー関係の終了はEntitlementを変えるが、PersonalSpaceを削除しない。
4. ValueTargetは本人だけが作成・変更・閲覧範囲決定できる。
5. RecoveredTimeはmethod、version、confidenceを伴う推定値として保存する。
6. Questの練習結果、Awareness Timing、対象行動の結果を分離する。
7. Entitlementは複数Grantから計算し、Personへ `isPremium` を保存しない。
8. 個人データをスポンサー集計へ使うには、目的別の有効なConsentが必要。
9. 集計はReportingPolicyの最小cohort数を下回る場合に抑止する。
10. Sponsor向けProjectionにpersonId、enrollmentId、自由入力を含めない。
11. Programが本人のBehaviorTarget、ValueTarget、Questを強制・上書きしない。
12. PracticeBlock、Quest、InterventionTemplateはversionを固定し、推薦と進行判断を再現可能にする。
13. 1日の結果だけでPracticeBlockのスキルを自動変更しない。
14. `not_remembered` と `no_opportunity` を有効練習回数へ算入せず、`no_opportunity` をAwarenessの高低へ換算しない。

## 10. 状態遷移

### ProgramEnrollment

```text
invited → active → left
                 └→ ended
```

### EntitlementGrant

```text
scheduled → active → expired
                   └→ revoked
```

### Personal Journey

```text
newUser
  → onboarding
  → practiceBlockAssigned
  → dailyQuestActive
  → waitingForScheduledReflection
  → reflectionRecorded
  → practiceBlockUpdated
      ├─ nextDailyQuestReady → dailyQuestActive
      └─ blockReviewReady
           ├─ continueSameBlock → dailyQuestActive
           └─ nextBlockReady → practiceBlockAssigned
```

ProgramEnrollmentの状態とPersonal Journeyは独立です。

## 11. 最初から入れる薄い実装フック

B2B機能を今すぐ作る必要はありません。MVPには次の境界だけ入れます。

1. `Person` と `PersonalSpace` を作り、Personal entityは `personalSpaceId` でscopeする。
2. `EntitlementResolving` interfaceを作り、当面はローカルのFree Grantを返す。
3. `ValueTarget` を独立型として定義し、オンボーディングでは任意入力にする。
4. Domain Eventにversionを持たせる。
5. Analytics Eventから自由入力・Personal IDを除く。
6. Organization / Programは別moduleまたは別packageに置き、Personal Domainからimportしない。

この6点で、現在の小さなPoCを過度に複雑化せず、将来のスポンサー契約とデータ境界を守れます。

## 12. 推奨Repository / Service境界

```text
// Personal
PersonalSpaceRepository
BehaviorTargetRepository
BehaviorProfileRepository
BehaviorStateRepository
ValueTargetRepository
InterventionTemplateRepository
PracticeBlockRepository
QuestRepository
QuestResultRepository

BehaviorStateEstimating
InteractionPlanning
InterventionSelecting
PracticeBlockManaging
ProgressionPolicy
QuestGenerating
RecoveredTimeEstimating

// Sponsorship (future module)
OrganizationRepository
SponsoredProgramRepository
ProgramEnrollmentRepository
EntitlementGrantRepository
EntitlementResolving

// Privacy and reporting (server-side future module)
ConsentRepository
ReportingPolicyRepository
AggregateMetricRepository
SponsorReporting
```

永続化、認証、決済、LLM SDKの型を各Domainへ漏らさないようにします。
