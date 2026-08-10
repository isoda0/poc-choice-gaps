# 介入・推薦エンジン v0.1

## 1. 基本方針

AIに心理的介入を自由に発明させません。管理された `InterventionTemplate` からシステムが候補を選び、LLMは必要に応じてユーザー入力の構造化や文面の個人化を担当します。

```text
BehaviorProfile + BehaviorState + QuestHistory
  → Safety Gate
  → Bottleneck Detection
  → Candidate Generation
  → Candidate Scoring
  → Deterministic Selection
  → Optional LLM Personalization
  → Quest
```

## 2. 初期Intervention Family

| Family | 狙い | 例 |
| --- | --- | --- |
| Awareness | 自動行動に気づく | 開く目的を一言決める |
| Pause | 衝動と行動の間を空ける | 30秒待つ |
| Stop Rule | 継続行動に選択点を作る | 1本見たら一度閉じる |
| Alternative | トリガー後の別行動を試す | 水を飲む、短く歩く |
| If-Then Plan | 状況と代替反応を結ぶ | 暇なら、まず立ち上がる |
| Reflection | パターンを理解する | うまくいった状況を選ぶ |
| Environment | 摩擦や合図を調整する | ホーム画面から外す |

## 3. MVP用テンプレート候補

最初は以下の8件でVertical Sliceと基本分岐を覆います。

| ID | Family | Mechanism | Quest概要 | 負担 |
| --- | --- | --- | --- | --- |
| `intentional_opening` | Awareness | awareness | 見るものを決めてから開く | 低 |
| `name_the_reason` | Awareness | awareness | 開く理由を1つ選ぶ | 低 |
| `pause_30_seconds` | Pause | urge_control | 30秒待ち、それでも見たければ見る | 低 |
| `pause_1_minute` | Pause | urge_control | 1分待ち、衝動の変化を見る | 中 |
| `stop_after_one_video` | Stop Rule | stopping_control | 1本後に一度閉じる | 低 |
| `choose_session_end` | Stop Rule | stopping_control | 開く前に終了点を1つ決める | 中 |
| `tiny_alternative` | Alternative | alternative_available | 先に水を飲む／立つ | 低 |
| `trigger_check` | Reflection | trigger_awareness | 直前の状態を1つ選ぶ | 低 |

心理学的ラベルやエビデンス強度は、専門家・一次資料によるレビュー後に確定します。

## 4. 初回ルール

```text
if problemPattern == automatic_open:
    intentional_opening

if problemPattern == urge_control:
    pause_30_seconds

if problemPattern == stopping_control:
    stop_after_one_video

if problemPattern == lack_alternative:
    tiny_alternative

if problemPattern == unknown:
    trigger_check
```

`interactionPreference == low` の場合は、自由入力を必須にするテンプレートを除外します。

## 5. Day 2ルール例

```text
Day 1 = intentional_opening

result.success:
    intentional_openingをもう1日試す、または軽いstop ruleへ進む

result.partial + continued_browsing:
    stop_after_one_video

result.failed + forgot:
    同じ介入をより目立つ合図付きで再提示

result.failed + too_burdensome:
    interactionCostがより低い候補

result.failed + automatic_open:
    入力不要のpause / visual cue候補
```

## 6. スコアリング

MVPは候補をルールで絞った後、説明可能な重み付きスコアを使います。

```text
score =
  mechanismFit       * 0.45
+ userFit            * 0.20
+ previousSuccess    * 0.15
+ novelty            * 0.10
+ lowBurden          * 0.10
- safetyPenalty
- repetitionPenalty
```

初回は `previousSuccess` を中立値にするか、残りの重みを正規化します。スコアと各要素はdecision logへ保存します。

## 7. 負担の扱い

同じ介入でも、ユーザー状態に応じてQuest化を変えます。

```text
receptivityが低い / 疲労が強い
  → タップだけ、30秒、自由入力なし

receptivityが高い / 週次振り返り
  → 一言入力、数分のReflectionも候補
```

負担を下げることは単なるUI調整ではなく、推薦条件の一部です。

## 8. LLMの責務

### 任せる

- 自由入力を既知enumと短い補足へ構造化する。
- 選択済みテンプレートを自然で非難しないQuest文面にする。
- 履歴から「昨日分かったこと」を1〜2文で要約する。
- 任意の振り返り会話を短く支援する。

### 任せない

- Safety Gateの最終判断。
- カタログ外の心理介入の自由生成。
- 診断、治療判断、緊急性判断をLLM単独で行うこと。
- スコアや履歴を無視した介入選択。
- 保存済みの構造化データを、会話要約だけで上書きすること。

## 9. AIなしで動く要件

PoCのコアループは、AIServiceが利用不可でも次を満たします。

- ボタン回答だけでプロフィールを作れる。
- 固定ルールでDay 1 / Day 2を選べる。
- 固定テンプレートでQuestを表示できる。
- Resultから状態を更新できる。

AI接続は価値を高める機能であり、推薦の唯一の実行経路にしません。

