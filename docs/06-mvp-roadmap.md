# MVPロードマップ

## 0. ゴール

MVPのゴールは「利用時間を減らせるアルゴリズムの完成」ではなく、次の適応ループが現実の生活で成立するかを確認することです。

```text
Observation → Small Intervention → Behavior → Feedback → Adaptation
```

## Phase 0: Foundation

### 成果物

- Xcodeプロジェクトと基本モジュール構成
- Domain型とRepository Protocol
- Person / PersonalSpaceによる個人データscope
- 独立したValueTarget型
- EntitlementResolving interface（当面はローカルFree Grant）
- `schemas/` とSwift型の対応表
- インメモリRepository
- 日時を注入できるClock

### 完了条件

- Domain層がSwiftUI、永続化、外部AI SDKへ依存しない。
- Personal entityがorganizationIdや課金状態へ依存しない。
- サンプルプロフィールを読み、Day 1介入を返す単体テストが通る。

## Phase 1: Static Vertical Slice

### 成果物

- Welcome
- 6問オンボーディング
- 固定ルールによるDay 1 Quest
- Quest実行とResult Check-in
- 固定ルールによるDay 2 Quest

### 完了条件

- アプリ内のモックデータだけでVertical Sliceを完走できる。
- UIコピーに禁止・診断・失敗の断定がない。

## Phase 2: Local Persistence

### 成果物

- Profile / State / Quest / Resultのローカル保存
- app lifecycleに応じた復元
- 重複Resultの防止
- 保存データの開発者向けリセット

### 完了条件

- アプリを終了しても進行状態を復元できる。
- 同じQuestに複数Resultが作られない。

## Phase 3: Explainable Adaptation

### 成果物

- 初期8件のInterventionTemplate
- Candidate generation / scoring
- decision log
- Resultに応じたBehaviorState更新
- Day 2理由文の固定テンプレート生成

### 完了条件

- 主要分岐のgolden testがある。
- 選択結果を入力・候補・スコアから再現できる。
- safetyまたはinteraction costで候補を除外できる。

## Phase 4: Usability Test Build

### 成果物

- 開発者イベントログ
- オンボーディング所要時間
- Quest acceptance / execution / result入力イベント
- TestFlight等で少人数が使えるビルド
- フィードバック導線

### 完了条件

- 個人を特定する不要な自由入力を収集しない。
- 主要イベントが重複なく記録される。
- 少人数テストの質問票と中止基準が定義される。

## Phase 5: Optional AI Layer

### 成果物

- `AIService` interface
- 自由入力のenum構造化
- 選択済みQuestの文面個人化
- AI失敗時の固定テンプレートfallback
- 出力schema validation

### 完了条件

- AI出力が推薦IDを変更できない。
- schema不正、timeout、offline時もコアループを完走できる。
- 機微情報の送信範囲が明文化され、ユーザーへ説明できる。

## Phase 6: Learn and Decide

### 判断項目

- Result Input RateとDay 2 Return Rateが継続投資に値するか。
- onboardingを短くすべきか。
- どのIntervention Familyに反応があるか。
- AI個人化に明確な価値があるか。
- 次にSNS / スマホ全般へ広げるか、YouTubeを深掘りするか。

## 優先順位付きバックログ

### P0

- Domain型
- 6問オンボーディング
- Day 1 / Day 2ルール
- Quest / Result保存
- lifecycle復元
- 安全スコープ表示
- 主要イベント

### P1

- 8件のテンプレート
- decision log
- 追加質問ルール
- accessibility / Dynamic Type
- 通知の最小実験
- データ削除

### P2

- LLM個人化
- 週次振り返り
- 軽量な進捗可視化
- SNS / スマホ全般のTarget Pack

### 明示的に後回し

- Screen Time API
- HealthKit
- Android / Apple Watch
- ソーシャル機能・ランキング
- 強化学習・Contextual Bandit
- アルコール・喫煙など医療安全要件の高い対象

## 実装前に決める短期課題

1. Deployment targetとSwift version。
2. SwiftDataか別のローカル保存方式か。
3. 匿名イベントをどこまで端末外へ送るか。
4. 通知なしでも最初のテストを成立させるか。
5. 正式名称決定前のBundle ID。
