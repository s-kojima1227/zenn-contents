---
title: "Linearから学ぶ「スプリントしない」チーム開発"
emoji: "📚"
type: "scrap"
topics: ["スクラム", "Linear", "サイクル", "スプリント"]
---

- 著者: ts / コミューン株式会社（2025年12月12日）
- URL: https://zenn.dev/dev_commune/articles/edcd7ecf9b6f4a

## 書き起こし

### はじめに

本記事はCommune Developers Advent Calendar 2025の12日目の記事です。

チームのタスク管理ツールとしてLinearを使い始めたところ、Linearが「スプリント」ではなく「サイクル」という用語を使っていることに気づきました。公式ドキュメントを読むと、これは単なる言い換えではなく明確な思想の違いが反映されているのだとわかりました。

この記事では、Linear MethodやLinearの公式ドキュメントを引用しながら、スプリントとサイクルの思想的な違いを整理します。スクラムを否定する意図は全くありません。また、これは個人的な調査であり、チームや会社として「サイクルを採用する」と決めたわけでもありません。異なるアプローチを知ることで、自チームの運用を見直すきっかけになればと思い、まとめてみました。

### 1. Linearはなぜ「スプリント」と呼ばないのか

公式Changelogにはこう書かれています。

> "You might know them as sprints, but we decided to call them cycles to give them meaning outside of the agile methodology." — Linear Changelog (2019)

さらに、公式ドキュメントではこう明言されています。

> "Unlike sprints, cycles are not tied to releases." — Linear Docs

スクラムのスプリントは「スプリント終了時にリリースできる状態の成果物を作る」ことを前提とした期間です。一方、Linearのサイクルはリリースとは切り離されています。期間内に完結させることを前提とせず、作業を区切るリズムとして機能します。終わらなければ次のサイクルで続ける、というシンプルな設計です。

### 2. スプリントとサイクルの思想的な違い

整理すると、スプリントは「予測可能性」を重視した戦略、サイクルは「着実な前進」を重視した戦略と捉えることができます。

**目的の違い：リリース vs モメンタム**

> "The goal is to maintain a healthy momentum with your teams, not to rush towards the end." — Linear Method

「モメンタム」は「勢い」や「推進力」を意味します。止まらずに前に進み続ける状態、と捉えるとわかりやすいかもしれません。

「リリースではなくモメンタム」という表現には、開発チームの現実が反映されているように思います。機能開発だけでなく、技術負債の解消、バグ修正、差し込み対応など、やることは常にあります。特定のリリースに向けて走り切るのではなく、継続的にアウトプットを出し続けることを後押しする設計です。

**未完了の扱い：振り返り vs 自動ロールオーバー**

> "Any unfinished work rolls over to the next cycle automatically." — Linear Docs

スクラムでは、未完了は振り返りの対象です。「なぜ達成できなかったか」を分析し、見積もり精度を高め、次のスプリントの予測可能性を向上させます。

一方、Linearの自動ロールオーバーは異なる思想に基づいています。サイクルは期日が来たら自動的にクローズされますが、未完了のタスクをクローズしたサイクルに残す方法はありません。これは「目の前のタスクから順々に完了させていく」ことを促す設計と捉えられます。終わらなかったものは次へ持ち越し、引き続き取り組む。シンプルにそれだけです。

**計画の性質：コミットメント vs フォーカス**

> "Cycles should feel reasonable. Don't overload cycles with tasks." — Linear Method

スプリントでは「どれだけコミットできるか」を計画時に決め、その達成を目指します。サイクルでは「無理のない量」でフォーカスする範囲を決め、着実に完了を積み重ねることを優先します。

### 3. サイクルが求める規律

ここまで読むと、「コミットメントがないと緩くなるのでは」という疑問が浮かぶかもしれません。サイクルは計画遵守を緩める代わりに、別の厳しさに重きを置いています。

**毎日の前進:**

> "You and your whole team should always try to take swift action and make progress each day." — Linear Method

「計画を守ったか」ではなく「毎日前進しているか」を重視しています。

**完了の積み重ね:**

> "Ideally you can complete several concrete tasks each week." — Linear Method

> "The clearest way to see whether something is complete or not is to show the diff in the code or design file." — Linear Method

見積もりの精度ではなく、実際に完了したものの数と質で進捗を測ります。

**見積もりの位置づけ:**

Linearにも見積もり機能はありますが、その使い方には特徴があります。

> "When estimates are too large, refine issue scope. Larger estimates usually mean that there is uncertainty about the issue's complexity." — Linear Docs

見積もり精度を上げることが目的ではなく、見積もりが大きい＝不確実性が高いと考え、Issueを分割するきっかけにします。

**ベロシティとCycle capacity:**

スクラムのベロシティと、LinearのCycle capacityは、似た計算方法ですが位置づけが異なります。

| | ベロシティ（スプリント） | Cycle capacity |
|---|---|---|
| 位置づけ | 計画の根拠 | 参考値 |
| 超えた/下回った場合 | 振り返りで原因分析 | 特に問題にしない |
| 安定性 | 安定させることが目標 | 重視しない |

振り返りの主役は、ベロシティではなく「実際に完了したものの数」になります。

**振り返りのアプローチ:**

スプリントとサイクルでは、振り返りで見るものが異なります。

| | スプリント | サイクル |
|---|---|---|
| 振り返りの目的 | 予測可能性を高める | 完了数の増加と質の向上 |
| 主な計測対象 | 計画達成率 | 完了したものの数と質 |
| 典型的な改善アクション | 見積もり精度の向上 | Issue分割・ブロッカー除去 |

### 4. 実践：スプリントしないチーム開発のポイント

サイクルの思想を取り入れる場合、以下のようなポイントが考えられます。

**計画達成率を追わない:** サイクルに積んだものを100%消化することを目標にしない。代わりに「何を完了させたか」で振り返ります。

**消化チケット数で振り返る:** ストーリーポイントは人や状況によってブレやすい指標です。完了したチケット数というシンプルな指標をメインに据え、ポイントは補助的に使う方法があります。

**差し込みを敵視しない:** 差し込みタスクは「計画を乱すもの」ではなく、「完了させたもの」としてカウントします。差し込みを一度受け止めて判断し、対応したものは成果として扱います。（LinearのTriage機能を使うことで実践できるようになっています）

**Cooldownを設ける:**

> "Including a cooldown period after each cycle to give your team a break and bandwidth to work through technical debt..." — Linear Docs

サイクルの間に意図的な余白を設けることで、持続可能なペースを維持できます（必須ではありません）。

### 5. どんなチームに向いているか

**向いていそうなケース:**
- 計画変更が頻繁に起きる環境（スタートアップなど）
- 機能開発以外（負債解消、差し込み対応等）の比率が高いチーム
- リリースタイミングに柔軟性がある環境

**向いていなさそうなケース:**
- 契約上のコミットメントがある開発（受託開発など）
- 計画精度が経営判断に直結する大規模組織
- 外部ステークホルダーへの予測提示が求められる環境

### おわりに

スプリントは「予測可能性」を高める戦略です。約束を守るように全力で走りきり、守れなければ振り返って精度を上げていきます。

サイクルは「着実な前進」を促す戦略です。目の前から順々に完了させ、アウトプットの総量を最大化することを目指します。

どちらが正しいかではなく、チームの状況と目的に応じて選択する問題です（私たちのチームも、Linearに移行したばかりでまだ手探りの状態です）。「約束を守れなかった」という振り返りがもし続いているなら、別のアプローチを試してみる価値はあるかもしれません。

### 参考

- Linear Method
- Linear Method - Principles & Practices
- Linear Method - Generate momentum
- Linear Docs - Cycles
- Linear Changelog - Cycles (2019)
