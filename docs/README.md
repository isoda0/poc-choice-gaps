# Documentation Index

このディレクトリは、構想を実装可能な判断へ落とすための正本です。READMEは入口、ここにある各文書は設計の詳細、`decisions/` は判断理由の履歴として扱います。

| 文書 | 用途 |
| --- | --- |
| [01-product-concept.md](01-product-concept.md) | プロダクトの目的、対象、原則、用語 |
| [02-ux-and-onboarding.md](02-ux-and-onboarding.md) | 初回ヒアリング、入力方法、質問予算、結果入力 |
| [03-domain-model.md](03-domain-model.md) | Entity、値、状態遷移、不変条件 |
| [04-intervention-engine.md](04-intervention-engine.md) | 介入カタログ、推薦、LLM境界 |
| [05-mvp-vertical-slice.md](05-mvp-vertical-slice.md) | Day 1からDay 2までの完成条件 |
| [06-mvp-roadmap.md](06-mvp-roadmap.md) | 実装フェーズ、順序、完了条件 |
| [07-safety-and-scope.md](07-safety-and-scope.md) | 対象外、安全ゲート、表現上の境界 |
| [08-metrics-and-validation.md](08-metrics-and-validation.md) | MVP指標、イベント、検証方法 |
| [09-b2b2c-domain-boundaries.md](09-b2b2c-domain-boundaries.md) | B2B2C、テナンシー、利用権、個人データ境界 |

## 出典と扱い

初期内容は、ChatGPT会話「依存症改善アプリ構想」（conversation ID: `6a799d59-5280-83ee-8577-ccfde6e21cf6`）で合意された構想を編集・統合したものです。会話中に言及された研究やガイドラインの引用は、このリポジトリでは検証済み参考文献として扱いません。研究・医療に関する主張は、製品化前に一次情報を確認し、必要に応じて専門家レビューを受けます。

## 更新ルール

- 大きな設計変更は `docs/decisions/` にADRを追加する。
- 「決定済み」「仮説」「未決定」を混同しない。
- UXコピーと内部ドメイン用語を分離する。
- 医療・安全に関わる対象範囲は、実装都合だけで拡張しない。
