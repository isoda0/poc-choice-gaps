# 介入・推薦エンジン v0.2

## 1. 基本方針

AIに心理的介入を自由に発明させません。管理された `InterventionTemplate` からシステムがPractice Blockを選び、LLMは必要に応じてユーザー入力の構造化や文面の個人化を担当します。

推薦単位は毎日のQuestではなくPractice Blockです。同じスキルを複数回練習し、Block Reviewで次のスキルを決めます。

```text
BehaviorProfile + BehaviorState + PracticeHistory
  → Safety Gate
  → Curriculum Stage
  → Bottleneck Detection
  → Candidate Generation
  → Candidate Scoring
  → Deterministic Block Selection
  → Optional LLM Personalization
  → Practice Block
  → Repeated Daily Quest + Scheduled Reflection
  → Progression Policy
```

## 2. 初期カリキュラム

| Stage | 狙い | 代表的な観察・スキル |
| --- | --- | --- |
| Awareness Timing | 気づいた位置を観察する | いつ気づいたかをラベリング |
| Trigger Awareness | 先行条件を知る | 疲労、退屈、時刻、直前行動 |
| Pause | 自動反応に短い間を作る | 一呼吸、10秒待つ |
| Intentional Choice | 行動の目的や終了点を選ぶ | 見るもの、終了点を決める |
| Alternative | 別の反応も選べるようにする | 水を飲む、立つ、後にする |
| Maintenance | 合う方法を再利用する | 高リスク場面の計画、振り返り |

全ユーザーへ完全に同じ順序を強制しませんが、`automatic_open` の初期ユーザーは低負担なAwareness Timingから開始します。事前質問だけでPauseやStop Ruleへ飛ばさず、まず実際の観察を得ます。

## 3. 初期Intervention Family

| Family | 狙い | 例 |
| --- | --- | --- |
| Awareness | 自動行動と気づいた位置を観察する | 気づいた時点で行動を言葉にする |
| Reflection | トリガーと結果を理解する | 代表的な場面を定時に振り返る |
| Pause | 衝動と行動の間を空ける | 一呼吸、10秒待つ |
| If-Then Plan | 状況と反応を結ぶ | アイコンに触れたら、行動を言葉にする |
| Intentional Choice | 目的や終了点を選ぶ | 見るものを1つ決める |
| Stop Rule | 継続行動に選択点を作る | 1本見たら一度閉じる |
| Alternative | トリガー後の別行動を試す | 水を飲む、短く歩く |
| Environment | 摩擦や合図を調整する | ホーム画面の配置を変える |

## 4. MVP用テンプレート候補

最初は以下の8件でAwarenessからChoiceまでの基本分岐を覆います。

| ID | Family | Stage | Quest概要 | 最低練習回数 | 負担 |
| --- | --- | --- | --- | ---: | --- |
| `notice_opening` | Awareness | Awareness Timing | 気づいた時点で「いま開こうとしている」と言う | 3 | 低 |
| `notice_the_trigger` | Reflection | Trigger Awareness | 一番印象に残った直前の状態を選ぶ | 3 | 低 |
| `pause_one_breath` | Pause | Pause | 気づいたら一呼吸してから選ぶ | 3 | 低 |
| `pause_10_seconds` | Pause | Pause | 10秒待ち、それでも見たければ見る | 3 | 低 |
| `intentional_opening` | Intentional Choice | Intentional Choice | 見るものを1つ決めてから開く | 3 | 低 |
| `choose_session_end` | Intentional Choice | Intentional Choice | 開く前に終了点を1つ決める | 3 | 中 |
| `stop_after_one_video` | Stop Rule | Intentional Choice | 1本後に一度閉じ、再び選ぶ | 3 | 低 |
| `tiny_alternative` | Alternative | Alternative | 見る前に水を飲む／立つ選択肢を試す | 3 | 低 |

最低練習回数の `3` はMVPの初期仮説であり、心理学的な確定値ではありません。日数ではなく、`practiced` と `remembered_afterward` の有効な気づきの練習で数えます。

心理学的ラベルやエビデンス強度は、[10-behavior-change-program-design.md](10-behavior-change-program-design.md) を出発点とし、専門家・一次資料によるレビュー後に確定します。

## 5. 初回ルール

```text
if problemPattern in [automatic_open, urge_control, unknown]:
    notice_opening

if problemPattern == stopping_control:
    notice_opening with copy focused on "次の動画へ移る瞬間"

if problemPattern == lack_alternative:
    notice_opening
    remember tiny_alternative as a later candidate
```

最初のBlockで高度な個人化を演出しません。`interactionPreference == low` の場合は、日中の入力や自由入力を必須にするテンプレートを除外します。

## 6. Daily Progression Policy

日々のReflection後は、原則として同じBlockを継続します。

```text
result.no_opportunity:
    練習回数を増やさず、同じQuestを継続

result.not_remembered:
    練習回数を増やさず、同じスキルを継続し、通知時刻または合図の調整を提案

result.practiced / remembered_afterward:
    観察を追加し、最低練習回数までは同じスキルを継続

burden is high:
    スキルは変えず、入力や実行条件を軽くする
```

1日の `not_remembered` や遅いAwareness Timingから能力不足を断定しません。

## 7. Block Progression Policy

Block終了または次の候補選択は、次を満たしたときだけ評価します。

```text
eligibleForReview =
  practiceCount >= minimumPracticeCount
  or userRequestsChange
  or burdenRequiresEarlyReview
```

初期ルール例:

```text
Block = notice_opening

Awareness Timingが複数回観察できた:
    notice_the_trigger または pause_one_breath

思い出せない結果が多い:
    notice_openingを継続し、合図・コピー・通知を調整

気づけるが自動的に開く:
    pause_one_breath

気づけるが見続けることが主課題:
    stop_after_one_video または choose_session_end

負担が高い:
    同Stage内のより低負担な候補
```

## 8. スコアリング

MVPは候補をカリキュラムとルールで絞った後、説明可能な重み付きスコアを使います。

```text
score =
  stageFit           * 0.35
+ bottleneckFit      * 0.25
+ userFit            * 0.15
+ practiceEvidence   * 0.15
+ lowBurden          * 0.10
- safetyPenalty
- prematureChangePenalty
```

`novelty` は加点しません。目新しさのために習得途中のスキルを変えないためです。スコアと各要素はdecision logへ保存します。

## 9. 状態更新

- `Awareness Timing` はQuestResultの観察値として保存する。
- BehaviorStateの `awareness.value` は複数観察から段階的に更新する。
- 1件の結果は主にconfidenceを上げ、単独で大きくstate値を変えない。
- `not_remembered` と `no_opportunity` を有効練習回数へ算入しない。
- `no_opportunity` から能力や実行率を推定しない。
- 定時の回想であることを考慮し、自己申告を観測事実と断定しない。

## 10. LLMの責務

### 任せる

- 自由入力を既知enumと短い補足へ構造化する。
- 選択済みテンプレートを自然で非難しないQuest文面にする。
- Block内の観察を「分かったこと」として1〜2文で要約する。
- 任意の振り返り会話を短く支援する。

### 任せない

- Safety Gateの最終判断。
- カタログ外の心理介入の自由生成。
- 診断、治療判断、緊急性判断をLLM単独で行うこと。
- 最低練習回数や進行条件を無視したBlock変更。
- 保存済みの構造化データを、会話要約だけで上書きすること。

## 11. AIなしで動く要件

PoCのコアループは、AIServiceが利用不可でも次を満たします。

- ボタン回答だけでプロフィールを作れる。
- 固定ルールで最初のPractice Blockを選べる。
- 同じQuestを複数日にわたり提示できる。
- 定時ReflectionからAwareness Timingを保存できる。
- 固定ルールでBlock継続・完了・次候補を決められる。

AI接続は価値を高める機能であり、推薦や練習継続の唯一の実行経路にしません。
