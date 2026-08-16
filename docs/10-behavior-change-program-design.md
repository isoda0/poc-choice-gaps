# 行動変容プログラム設計の根拠と翻訳 v0.1

## 1. 目的

Choice Gapは依存症の診断・治療を行う製品ではありません。本書は、心理学をもとに構築された依存症改善プログラムの共通構造を、低リスクなYouTube習慣のセルフケアへどう翻訳するかを記録します。

ここで参照する知見は設計根拠の出発点であり、YouTube利用に対する有効性の証明ではありません。

## 2. 参照したプログラムと研究

### Combined Behavioral Intervention（CBI）

NIAAAのCBIは、動機づけ、機能分析、個別化したスキルトレーニング、維持という段階で構成されます。先行条件と結果、行動が本人に与えていた利益、代替となる対処方法を整理し、必要なモジュールを絞って宿題として練習します。

- Source: [NIAAA Combined Behavioral Intervention Manual](https://www.niaaa.nih.gov/sites/default/files/NIAAA-combined-behavioral-intervention-manual.pdf)
- Choice Gapへの翻訳: Onboarding → Awareness/Trigger観察 → Practice Block → Block Review → Maintenance
- 境界: アルコール依存の臨床プログラムと、YouTube習慣のセルフケアは対象・リスク・治療強度が異なる。

CBIのUrge Monitoringでは、衝動の日時、状況、強さ、反応を記録し、通常2〜3週間の宿題として扱います。記録は可能な限り場面の直後が望ましく、一日の終わりの再構成は精度が低いとされています。

Choice Gapは日中の入力を必須にしないため、定時Reflectionを正確な行動ログとみなしません。代表的な1場面の自己申告として扱い、必要性が確認された場合だけ任意の即時記録を追加します。

### Cognitive-Behavioral Therapy for Addiction

NIDAのCBTマニュアルでは、機能分析を初回だけで終えず、利用や渇望の前に何があり、どう対処したかを各セッションで繰り返し確認します。スキルは説明を読むだけでは身につかず、試行、間違いの発見、再試行を通じて習得するものとして、セッション間の宿題が使われます。

- Source: [NIDA Cognitive-Behavioral Therapy Manual](https://archives.nida.nih.gov/sites/default/files/cbt.pdf)
- Choice Gapへの翻訳: 毎日別の助言を出さず、同じスキルをPractice Blockで複数回試す。
- 境界: 臨床家との治療関係、診断、併存症対応、危機対応をアプリで代替しない。

### Mindfulness-Based Relapse Prevention（MBRP）

MBRPは、環境的な手がかり、感情、身体感覚、衝動へ気づき、自動反応せず別の関わり方を練習するアプローチです。物質使用障害の治療後ケアを対象にしたランダム化試験では、8回の週次セッションによるMBRP、標準的な再発予防、通常治療が比較されました。

- Source: [Bowen et al., JAMA Psychiatry 2014](https://jamanetwork.com/journals/jamapsychiatry/fullarticle/1839290)
- Choice Gapへの翻訳: 衝動を消すことより、気づいた位置と選択可能性を観察する。
- 境界: 試験は治療を完了した物質使用障害の成人を対象としており、YouTube習慣への効果は示していない。

### 行動嗜癖に対するCBT

ギャンブル関連害のNICEガイドラインは、動機づけ面接と、再発予防を含む複数回のCBTを推奨しています。再発予防にはトリガーへの対処と再発時の対応が含まれます。

- Source: [NICE NG248: Gambling-related harms](https://www.nice.org.uk/guidance/ng248/chapter/recommendations)
- Choice Gapへの翻訳: 単発の介入ではなく、段階的なコースと維持計画として設計する。
- 境界: YouTubeの過剰利用をギャンブル障害と同一視しない。

### Implementation Intention

Implementation Intentionは、特定の手がかりと小さな反応を `when/if X, then Y` の形で結ぶ自己調整方法です。喫煙者を対象にしたランダム化試験では、この形式の計画が習慣的な喫煙行動の変化を支援する可能性が示されました。

- Source: [Armitage, Health Psychology 2016](https://pubmed.ncbi.nlm.nih.gov/27054302/)
- Choice Gapへの翻訳: 「YouTubeアイコンに触れたことに気づいたら、『いま開こうとしている』と言う」のように、観察可能な手がかりと低負担な反応を結ぶ。
- 境界: 喫煙中止の効果量をYouTube利用へ転用せず、Quest形式の仮説として検証する。

## 3. Choice Gapの設計原則への翻訳

| 心理学的な構造 | Choice Gapの設計 | 避けること |
| --- | --- | --- |
| 動機づけ | 本人が望む使い方と価値を確認する | 禁欲を外部から目標にする |
| 機能分析 | Trigger → Action → ConsequenceをReflectionで観察する | 初回質問だけで原因を断定する |
| 自己モニタリング | Awareness Timingと代表的トリガーを記録する | 利用時間だけを成功指標にする |
| スキル練習 | 1つのPractice Blockを複数回繰り返す | 毎日新しい助言へ切り替える |
| Implementation Intention | 明確な手がかりと10秒以内の反応を結ぶ | 抽象的な「意識しよう」で終える |
| マインドフルな気づき | 遅く気づいても観察として扱う | 衝動を消すことを成功条件にする |
| レビュー | 定時に代表的な1場面を短く振り返る | 回想データを正確な全件ログとみなす |
| 再発予防 | 忘れた場面・戻った行動から次の計画を作る | 1日を失敗としてリセットする |

## 4. 初期カリキュラム仮説

```text
Stage 1: Awareness Timing
  気づいた時点を観察し、行動をラベリングする

Stage 2: Trigger Awareness
  疲労、退屈、時刻、直前行動などの先行条件を知る

Stage 3: Pause
  一呼吸または短い待機を入れ、衝動と行動を分ける

Stage 4: Intentional Choice
  見る目的、終了点、見る／見ない／後にするを選ぶ

Stage 5: Alternative and Maintenance
  自分に合う代替行動と高リスク場面の計画を再利用する
```

各Stageの日数や最低練習回数は研究から直接決めず、MVPで負担、想起、習得感を検証します。

## 5. 定時Reflectionの位置づけ

定時通知は、依存行動の直前に外部から介入するためではなく、宿題を振り返るセッションの代替です。

- 日中: 本人がアプリなしでスキルを試す。
- 定時: 代表的な1場面を30秒程度で振り返る。
- Block Review: 複数回の観察から次の練習を決める。

定時通知を送れば自動的に習慣化できるとは仮定しません。通知時刻、開封、回答負担、想起精度、継続率をMVPで検証します。

## 6. 製品効果を主張する前に必要なこと

- YouTube習慣を対象にしたユーザビリティと受容性の検証。
- Awareness Timing尺度の理解可能性と再現性の確認。
- Practice Blockの回数・期間・進行条件の事前定義。
- 比較可能なデザインによる行動アウトカムの検証。
- 行動科学・臨床・医療安全の専門家レビュー。
- 「依存症改善」「治療」「予防」等の表現に関する法務・規制レビュー。

## 7. 参照状況

上記リンクは2026-08-15に参照しました。一次資料・公的ガイドラインとして内容を確認していますが、Choice Gapの設計全体が専門家により検証済みであることを意味しません。
