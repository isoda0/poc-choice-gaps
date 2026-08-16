# Choice Gap（仮称）

> 「やめさせる」のではなく、「自分で選べるようにする」。

Choice Gap は、望まないデジタル習慣に対して、本人が自動的な行動へ少しずつ早く気づき、自分で選べるようになるための行動変容支援アプリです。禁止や強制ブロックを中心にせず、毎日の小さな実験（Quest）と定時の振り返りを通して、「気づく位置」を行動の後から前へ移す練習を支援します。

アプリがYouTubeを開く直前に検知・介入することは、コアループの前提ではありません。平時にスキルを提示し、ユーザーが日常で試し、後から短く振り返る「スキル練習 + 宿題 + レビュー」の構造を取ります。

長期的には、本人が取り戻した時間を学習・休息・運動・家族との時間など、自分が価値を置く活動へ使えるよう支援します。収益モデルは個人サブスクに加え、企業・大学等がPremium利用権を提供するB2B2Cを想定します。ただし、スポンサーは個人の行動データを閲覧できません。

現在は企画・設計段階です。最初のPoCは **YouTubeを無目的に開く／見続ける行動** に対象を限定し、iOS / SwiftUI とローカル保存でコアループを検証します。

## MVPで検証すること

```text
最小限のヒアリング
  → 低負担な気づきのスキルを1つ提示
  → 同じスキルを日常で複数回練習
  → 毎日定時に短く振り返る
  → 気づいたタイミングとトリガーを更新
  → 練習ブロックの境界で次のスキルを決める
```

検証したい主な仮説は次の4つです。

1. 2分程度の初回ヒアリングなら完了してもらえる。
2. アプリが利用直前に介入しなくても、低負担なQuestを日常で思い出して試せる。
3. 定時の30秒程度の振り返りなら習慣化できる。
4. 同じ気づきのスキルを反復すると、YouTube利用に気づくタイミングが少しずつ早くなる。

## プロダクト原則

- 禁止より選択を支援する。
- 利用時間の削減より先に、気づきとChoice Pointの増加を評価する。
- 気づいたのが行動後でも、観察できたこととして扱う。
- 一度に練習するスキルは原則1つにし、1日の結果だけで切り替えない。
- 「介入を実行できたか」と「対象行動をしたか」を分けて評価する。
- 日中の即時記録は任意とし、定時の振り返りをコアループにする。
- 推薦は説明可能なルールで決め、LLMは主に入力の構造化と表現の個人化に使う。
- 医療行為・診断・重度依存の自己治療をMVPの対象にしない。
- 支払者と利用者を分け、企業契約でも個人の行動データを本人領域に保つ。

## ドキュメント

- [構想とプロダクト原則](docs/01-product-concept.md)
- [UX・初回ヒアリング仕様](docs/02-ux-and-onboarding.md)
- [ドメインモデル](docs/03-domain-model.md)
- [介入・推薦エンジン](docs/04-intervention-engine.md)
- [MVP Vertical Slice](docs/05-mvp-vertical-slice.md)
- [MVPロードマップ](docs/06-mvp-roadmap.md)
- [安全性とスコープ境界](docs/07-safety-and-scope.md)
- [指標と検証計画](docs/08-metrics-and-validation.md)
- [B2B2C・テナンシー・データ境界](docs/09-b2b2c-domain-boundaries.md)
- [行動変容プログラム設計の根拠と翻訳](docs/10-behavior-change-program-design.md)
- [設計判断記録](docs/decisions/)
- [実装用JSON Schema](schemas/)

## 想定する初期アーキテクチャ

```text
SwiftUI
  ├─ OnboardingFlow
  ├─ PracticeFlow
  └─ DailyReflection

Application
  ├─ OnboardingService
  ├─ BehaviorStateEstimator
  ├─ PracticeBlockService
  ├─ InterventionSelector
  ├─ ProgressionPolicy
  └─ QuestResultService

Persistence
  └─ Local Storage

System Boundary
  └─ Local Notification Scheduler

Optional AI boundary
  └─ AIService
```

最初のPoCではバックエンド、YouTube利用検知、Screen Time APIを必須にしません。推薦ロジック、練習継続ルール、介入テンプレートはアプリ内で再現可能・テスト可能に保ちます。

## 実装開始時の推奨順序

1. `schemas/` をSwiftの型へ落とす。
2. `PracticeBlock` と `ProgressionPolicy` を純粋なDomainロジックとして実装する。
3. モックデータで Onboarding → Day 1 Quest → 定時振り返り → Day 2の同一Quest → Block Review を通す。
4. ローカル永続化と定時通知を追加する。
5. 実機でQuest想起率、振り返り入力率、気づきのタイミングを測る。
6. LLMは固定テンプレート版が動いた後に接続する。

## 現時点の非目標

- YouTube起動直前の強制介入
- Screen Time API / HealthKit 連携
- Android / Apple Watch対応
- SNS、ランキング、複雑なXPシステム
- 長期分析ダッシュボード
- 強化学習、Contextual Bandit
- 完全な音声AI会話
- アルコール、薬物、重度の依存を対象にした介入

## ステータス

`Concept / Pre-implementation`

このリポジトリの構想は2026-08-15に、気づきの段階的学習、反復練習、定時振り返りを中心とする形へ更新しました。依存症治療の構造は設計上の参考とし、YouTube習慣への有効性、医療安全、法務・プライバシー要件は、製品化前に対象領域の一次情報と専門家レビューで別途検証します。
