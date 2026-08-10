# Choice Gap（仮称）

> 「やめさせる」のではなく、「自分で選べるようにする」。

Choice Gap は、望まないデジタル習慣に対して、衝動と行動の間に小さな「選択の余白」を作る行動変容支援アプリです。禁止や強制ブロックを中心にせず、毎日の小さな実験（Quest）を通して、自動的な行動を意図的な選択へ変えることを目指します。

現在は企画・設計段階です。最初のPoCは **YouTubeを無目的に開く／見続ける行動** に対象を限定し、iOS / SwiftUI とローカル保存でコアループを検証します。

## MVPで検証すること

```text
最小限のヒアリング
  → 状態を推定
  → 小さなQuestを1つ提示
  → 実行結果を最小入力で記録
  → 状態を更新
  → 翌日のQuestを変える
```

検証したい主な仮説は次の3つです。

1. 2分程度の初回ヒアリングなら完了してもらえる。
2. 10秒程度のQuestなら日常の中で試してもらえる。
3. 前日の結果に応じてQuestを変えると、個人化されていると感じてもらえる。

## プロダクト原則

- 禁止より選択を支援する。
- 完全な理解より、次の意思決定に必要な最小情報を集める。
- 一度に提示するQuestは原則1つ。
- 「介入を実行できたか」と「対象行動をしなかったか」を分けて評価する。
- 推薦は説明可能なルールで決め、LLMは主に入力の構造化と表現の個人化に使う。
- 医療行為・診断・重度依存の自己治療をMVPの対象にしない。

## ドキュメント

- [構想とプロダクト原則](docs/01-product-concept.md)
- [UX・初回ヒアリング仕様](docs/02-ux-and-onboarding.md)
- [ドメインモデル](docs/03-domain-model.md)
- [介入・推薦エンジン](docs/04-intervention-engine.md)
- [MVP Vertical Slice](docs/05-mvp-vertical-slice.md)
- [MVPロードマップ](docs/06-mvp-roadmap.md)
- [安全性とスコープ境界](docs/07-safety-and-scope.md)
- [指標と検証計画](docs/08-metrics-and-validation.md)
- [設計判断記録](docs/decisions/)
- [実装用JSON Schema](schemas/)

## 想定する初期アーキテクチャ

```text
SwiftUI
  ├─ OnboardingFlow
  ├─ QuestFlow
  └─ ResultCheckIn

Application
  ├─ OnboardingService
  ├─ BehaviorStateEstimator
  ├─ InteractionPlanner
  ├─ InterventionSelector
  ├─ QuestGenerator
  └─ QuestResultService

Persistence
  └─ Local Storage

Optional AI boundary
  └─ AIService
```

最初のPoCではバックエンドを必須にしません。推薦ロジックと介入テンプレートはアプリ内で再現可能・テスト可能に保ちます。

## 実装開始時の推奨順序

1. `schemas/` をSwiftの型へ落とす。
2. ルールベースの `InterventionSelector` を純粋関数として実装する。
3. モックデータで Onboarding → Day 1 → Result → Day 2 を通す。
4. ローカル永続化を追加する。
5. 実機でヒアリング所要時間と結果入力率を測る。
6. LLMは固定テンプレート版が動いた後に接続する。

## 現時点の非目標

- Screen Time API / HealthKit 連携
- Android / Apple Watch対応
- SNS、ランキング、複雑なXPシステム
- 長期分析ダッシュボード
- 強化学習、Contextual Bandit
- 完全な音声AI会話
- アルコール、薬物、重度の依存を対象にした介入

## ステータス

`Concept / Pre-implementation`

このリポジトリの文書は、2026-08-10時点の構想会話を、後続実装に使える形へ再構成したものです。心理学的エビデンス、医療安全、法務・プライバシー要件は、リリース前に専門家と一次情報で別途検証します。

