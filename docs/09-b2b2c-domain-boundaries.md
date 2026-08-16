# B2B2C・テナンシー・データ境界 v0.1

## 1. ビジネスモデル

```text
企業・大学等 ── 契約・料金 ──→ Choice Gap
                                   │
                                   └─ Premium利用権 ─→ 本人

本人 ── PersonalSpace内で行動変容 ─→ 本人だけが詳細を閲覧

Choice Gap ── 閾値を満たす集計のみ ─→ スポンサー
```

スポンサーは顧客・支払者ですが、個人の行動データの所有者ではありません。この非対称性をテナンシー設計の中心にします。

## 2. なぜ通常のorganization tenantに入れないか

一般的なB2B SaaSのように、すべてのレコードへ `organizationId` を付けると次の問題が起きます。

- 企業管理者が個人データへアクセスできる設計になりやすい。
- 退職・卒業時にデータの所有権と削除主体が曖昧になる。
- 複数組織のProgramへ参加した人の履歴が分断される。
- 個人契約から企業契約、またはその逆への移行でデータ移行が必要になる。
- 企業が本人のValueTargetやQuestへ影響できてしまう。

そのため、個人データは `PersonalSpace`、組織契約は `SponsoredProgram`、両者の接続は `ProgramEnrollment` と `EntitlementGrant` に限定します。

## 3. データ分類

| 分類 | 例 | 主な閲覧者 |
| --- | --- | --- |
| Personal Sensitive | BehaviorTarget、Trigger、State、QuestResult、ValueTarget、自由入力 | 本人、必要最小限のサービス処理 |
| Sponsor Operational | 招待、席割当、参加状態、契約期間 | 認可されたスポンサー管理者 |
| Aggregate Reporting | 閾値を満たした継続率、Reflection参加率、任意自己評価 | スポンサー管理者 |
| Commercial | 契約、請求、Plan、SeatPool | Billing担当者 |
| Security / Audit | 同意、アクセス判断、管理操作 | 限定された運用・監査担当 |

Personal SensitiveとSponsor Operationalを同じ管理画面用Query Modelに結合しません。

## 4. Sponsor管理者ができること

- Programを作成・終了する。
- 席数と契約状況を確認する。
- 招待を送信・失効する。
- 利用権の割当状態を管理する。
- 閾値を満たすProgram集計を閲覧する。
- 任意の教育コンテンツやテーマを提示する。

## 5. Sponsor管理者ができないこと

- 個人のYouTube/SNS利用状況を見る。
- 個人のBehaviorTarget、ValueTarget、Triggerを知る。
- 個人のQuest内容、Practice Outcome、Awareness Timing、自由入力を見る。
- 個人へ学習や業務上のValueTargetを設定する。
- 個人の推薦アルゴリズムを上書きする。
- 人事評価、学業評価、懲戒のための個人スコアを取得する。
- 同意撤回を妨げる。

## 6. アカウントと契約のライフサイクル

### 企業Programへ参加

```text
Invitation accepted
  → ProgramEnrollment active
  → Sponsored EntitlementGrant active
  → EffectiveEntitlement recalculated
```

既存の個人アカウントがあれば同じPersonalSpaceを使います。新しい企業専用PersonalSpaceを作りません。

### 退職・卒業・Program終了

```text
ProgramEnrollment left / ended
  → Sponsored EntitlementGrant expired
  → EffectiveEntitlement recalculated
  → PersonalSpace and history remain personal
```

本人はFreeへ戻る、個人サブスクへ移る、またはデータ削除を選びます。スポンサーはPersonalSpaceの削除を要求できません。ただし法令・契約上必要なスポンサー運用データの保持は別ポリシーで管理します。

### 複数スポンサー

複数のEntitlementGrantを許容し、最も有利な機能集合と有効期限をresolverで計算します。Personal dataをProgramごとに複製しません。

## 7. 組織向けレポート生成

```text
Personal Events
  → valid consent check
  → approved metric calculation
  → minimum cohort / dimension checks
  → AggregateMetricSnapshot
  → Sponsor Dashboard
```

ルール:

- MetricDefinitionをversion管理する。
- ProgramごとのReportingPolicyで許可metricとdimensionを限定する。
- 最小cohort未満は値を返さず `suppressed_small_cohort` とする。
- 自由入力を集計ソースにしない。
- 個人IDをAggregateMetricSnapshotへ入れない。
- 少数属性の組み合わせによる再識別を防ぐ。
- 集計であってもスポンサー利用目的を本人へ説明する。

## 8. ProgramとPersonalizationの境界

Programは「6週間のデジタルセルフコントロール教育」のような教材・テーマを提供できます。一方、個人のQuest選択は本人のBehaviorStateと履歴に基づきます。

```text
ProgramContent: 提示してよい候補・教材
Personal Recommendation: 本人に出すQuest
Sponsor Reporting: 集計されたProgram利用状況
```

この3つを別Entity・別Serviceとして実装します。ProgramContentをQuestAssignmentとして強制配信しません。

## 9. MVPで実装するもの／しないもの

### 今入れる

- Person / PersonalSpace
- 独立したValueTarget
- Personal entityのscope ID
- EntitlementResolving interface
- purpose別Consentを追加できる形
- version付きDomain Event

### 今は実装しない

- Organization管理画面
- SSO / SCIM
- Seat billing
- Sponsor Reporting pipeline
- 学校・未成年者フロー
- Program curriculum配信
- 個人と組織の実データ同期

境界を先に固定し、未検証のB2B機能は作り込みません。

## 10. 実装レビュー用チェックリスト

- 新しいPersonal entityにorganizationIdを追加していないか。
- `isPremium` をUserへ保存していないか。
- スポンサー管理者向けAPIがPersonal Repositoryを呼んでいないか。
- 招待メール等を行動データストアへ保存していないか。
- ValueTargetをProgramが作成・変更できないか。
- 集計値にmetric / policy versionとparticipantCountがあるか。
- cohortが小さい場合に値を返していないか。
- Program終了でPersonalSpaceを消していないか。
- LLMへスポンサー名・所属・個人行動を不必要に同時送信していないか。
