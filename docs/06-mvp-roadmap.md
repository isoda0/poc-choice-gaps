# MVPロードマップ v0.2

## 0. ゴール

MVPのゴールは「利用時間を減らせるアルゴリズムの完成」ではなく、次の練習ループが現実の生活で成立するかを確認することです。

```text
Skill Assignment
  → Repeated Practice
  → Scheduled Reflection
  → Awareness Observation
  → Block Review
  → Next Skill
```

## Phase 0: Foundation

### 成果物

- Xcodeプロジェクトと基本モジュール構成
- Domain型とRepository Protocol
- `PracticeBlock`、`Quest`、`QuestResult`、`AwarenessTiming`
- Person / PersonalSpaceによる個人データscope
- 独立したValueTarget型
- EntitlementResolving interface（当面はローカルFree Grant）
- `schemas/` とSwift型の対応表
- インメモリRepository
- 日時を注入できるClock

### 完了条件

- Domain層がSwiftUI、永続化、通知、外部AI SDKへ依存しない。
- Personal entityがorganizationIdや課金状態へ依存しない。
- サンプルプロフィールから最初のAwareness Blockを返す単体テストが通る。
- `not_remembered` と `no_opportunity` を除外して練習回数を計算できる。

## Phase 1: Static Practice Slice

### 成果物

- Welcome
- 6問オンボーディング
- Reflection時刻設定UI
- 固定ルールによる `notice_opening` Block
- Day 1 Daily Reflection
- Day 2も同じQuestを継続
- 3件のモック結果によるBlock Review

### 完了条件

- アプリ内のモックデータだけでVertical Sliceを完走できる。
- YouTube起動前のアプリ操作を要求しない。
- UIコピーに禁止・診断・失敗の断定がない。
- 1日の結果だけでスキルが切り替わらない。

## Phase 2: Local Persistence & Reminder

### 成果物

- Profile / State / PracticeBlock / Quest / Resultのローカル保存
- app lifecycleに応じた復元
- 1日1件のReflection結果制約
- ローカル通知のスケジュール、時刻変更、停止
- 通知許可がない場合のアプリ内導線
- 保存データの開発者向けリセット

### 完了条件

- アプリを終了してもPractice Blockと当日の状態を復元できる。
- 同じ日のQuestに複数Resultが作られない。
- 通知時刻を変えた場合、古い予定が重複しない。
- 通知を拒否してもReflectionを記録できる。

## Phase 3: Explainable Progression

### 成果物

- 初期8件のInterventionTemplate
- Curriculum Stage / Candidate generation / scoring
- Daily Progression Policy
- Block Progression Policy
- decision log
- 複数結果に応じたBehaviorState更新
- Block Review理由文の固定テンプレート生成

### 完了条件

- 主要分岐のgolden testがある。
- 同じBlockを継続・完了した理由を入力とルールから再現できる。
- `no_opportunity`、`not_remembered`、高負担を別々に扱える。
- safetyまたはinteraction costで候補を除外できる。

## Phase 4: Usability Test Build

### 成果物

- 開発者イベントログ
- オンボーディング所要時間
- Quest想起、Reflection入力、Practice Block継続イベント
- Awareness Timingの探索的集計
- TestFlight等で少人数が使えるビルド
- フィードバック導線
- 通知時刻・頻度を調整できる設定

### 完了条件

- 個人を特定する不要な自由入力を収集しない。
- 主要イベントが重複なく記録される。
- Awareness Timingが自己申告であることを分析上区別する。
- 少人数テストの質問票と中止基準が定義される。

## Phase 5: Optional AI Layer

### 成果物

- `AIService` interface
- 自由入力のenum構造化
- 選択済みQuestとBlock Reviewの文面個人化
- AI失敗時の固定テンプレートfallback
- 出力schema validation

### 完了条件

- AI出力がInterventionTemplate ID、最低練習回数、進行判断を変更できない。
- schema不正、timeout、offline時もコアループを完走できる。
- 機微情報の送信範囲が明文化され、ユーザーへ説明できる。

## Phase 6: Learn and Decide

### 判断項目

- Scheduled Reflection Completion RateとDay 2 / Day 3 Return Rateが継続投資に値するか。
- 日中のアプリ介入なしでQuestを思い出せるか。
- Awareness Timingが複数回観察でき、前方への変化が見られるか。
- Practice Blockの標準回数・日数を短く／長くすべきか。
- 定時の回想だけで十分か、任意の即時ワンタップ記録が必要か。
- どのIntervention Familyに反応があるか。
- AI個人化に明確な価値があるか。
- 次にSNS / スマホ全般へ広げるか、YouTubeを深掘りするか。

## 優先順位付きバックログ

### P0

- Domain型
- 6問オンボーディング
- Practice BlockとAwareness Timing
- Day 1 / Day 2同一Quest
- Daily Reflection
- ローカル通知
- PracticeBlock / Quest / Result保存
- lifecycle復元
- 安全スコープ表示
- 主要イベント

### P1

- 8件のテンプレート
- Block Progression Policyとdecision log
- 通知時刻・スヌーズの調整
- Block Review
- 任意の即時ワンタップ記録
- accessibility / Dynamic Type
- データ削除

### P2

- LLM個人化
- 週次振り返り
- 軽量な進捗可視化
- SNS / スマホ全般のTarget Pack

### 明示的に後回し

- YouTube起動直前の強制介入
- Screen Time API
- HealthKit
- Android / Apple Watch
- ソーシャル機能・ランキング
- 強化学習・Contextual Bandit
- アルコール・喫煙など医療安全要件の高い対象

## 実装前に決める短期課題

1. Deployment targetとSwift version。
2. SwiftDataか別のローカル保存方式か。
3. 通知の提案時刻、許可拒否時、タイムゾーン変更時の挙動。
4. Practice Blockの初期最低練習回数と最大継続日数。
5. 匿名イベントをどこまで端末外へ送るか。
6. 正式名称決定前のBundle ID。
