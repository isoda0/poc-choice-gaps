# ADR 0004: Questを反復練習ブロックとして進行する

- Status: Accepted
- Date: 2026-08-15

## Context

当初のVertical Sliceは、Choice GapでQuestを確認してからYouTubeを開き、foreground復帰時に結果を入力し、1日の結果からDay 2に別のQuestを選ぶ構造だった。

しかし、Choice Gapの目的はアプリが利用直前に外部介入することではなく、本人が自動的な行動へ少しずつ早く気づけるようになることにある。CBTやマインドフルネス型再発予防では、平時にスキルを学び、日常で宿題として繰り返し、後のセッションで振り返る構造が使われる。1回の結果だけで能力やスキル適合を判断することも避ける必要がある。

## Decision

- YouTube起動直前の検知・アプリ表示をコアループの前提にしない。
- 同じInterventionTemplateを複数回練習する `PracticeBlock` を導入する。
- 毎日のQuestはPracticeBlockに属し、最低練習回数までは原則同じスキルを続ける。
- 日中のアプリ入力は必須にせず、本人が設定した時刻の `Daily Reflection` を標準の結果入力にする。
- 結果は成功／失敗ではなく、`practiceOutcome`、`awarenessTiming`、`observedTrigger`、対象行動を分離して保存する。
- `not_remembered` と `no_opportunity` は有効練習回数へ算入せず、`no_opportunity` はAwareness評価へ算入しない。
- 次のスキルは1日の結果ではなく、最低練習回数後のBlock Reviewで選ぶ。
- 本人は次のスキルへ進むか、同じスキルを続けるか選べる。

## Consequences

- Choice Gapの価値を「リアルタイム介入」ではなく「内的なChoice Pointの学習」として明確にできる。
- Screen Time APIやYouTube起動検知がなくてもPoCを成立させられる。
- 気づくのが遅かった日も有効な観察として扱える。
- 毎日の目新しさより、反復と習得を優先する推薦ロジックになる。
- `PracticeBlock`、進行ポリシー、定時通知、通知許可拒否時の導線が新たに必要になる。
- 定時の回想には記憶誤差があるため、結果を観測事実や臨床効果として断定できない。
- 将来、必要性が確認された場合は任意の即時ワンタップ記録を追加できるが、利用直前の強制介入にはしない。

## Evidence boundary

依存症治療プログラムの構造をYouTube習慣へ直接一般化はしない。根拠と翻訳上の境界は [10-behavior-change-program-design.md](../10-behavior-change-program-design.md) に記録し、製品効果の主張には対象領域での別の検証を要求する。
