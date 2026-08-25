---
title: エージェントを観察し、評価し、安全に保つ
lab:
  title: エージェントを観察し、評価し、安全に保つ
  description: Caldova エージェントが実際に何をしているのかを見てみましょう。OpenTelemetry でトレースし、内蔵のエバリュエータでグラウンド トゥルースと照らし合わせて応答をスコアリングし、AI レッド チーミング エージェントで攻撃します。 モジュール式のラボは、最後まで通して進めることも、タスクを 1 つずつ進めることもできます。
  type: lab
  id: D
  order: 4
  difficulty: 3
  duration: 60
  access: open
  level: 300
  concepts: 'tracing, OpenTelemetry, evaluation, groundedness, AI red teaming'
  islab: true
  status: draft
---

<!--
PILOT NOTE (remove before publishing):
"Lab D" is new content: there was no observability, evaluation or safety-testing
material anywhere in this repo. It follows the same template as Labs A-C.
Starter code lives in a single folder — Labfiles/D-observe-evaluate-and-secure-agents/Python/ —
shared by every task (one virtual environment, one .env). The completed reference code is
in Labfiles/D-observe-evaluate-and-secure-agents/Solution/Python/.

This landing page is the lab overview. Setup lives in D0-getting-started.md and each task is
its own page (D1-D3) so it can be completed on its own. The azd template and Bicep are
generated from Labfiles/_shared/ — edit them there, not in the lab folder.
-->

# エージェントを観察し、評価し、安全に保つ

**レベル** ▰▰▰▱▱ **L300**  (**L100** 初級 → **L500** エキスパート)

エージェントは半日もあれば構築できます。 それが良いかどうか、そして、それを誰かが悪用しようとしたときに反応するかどうかを知るのは、別の仕事です。 このラボは、その仕事についての説明です。つまり、稼働中のエージェントの内部を見て、その応答の質を測定し、他の誰かより先にそれを攻撃することです。

![Anton](../Media/anton-avatar.png)<br /><strong>あなたの AI ガイド、Anton をご紹介します。</strong><br />このラボでは **Ask Anton** のヒントを見つけることができます。 より対話型の、実践的なヘルプが必要ですか? *[Ask Anton](https://aka.ms/choose-anton)* アプリで Anton と会話しましょう。

<details>
<summary><strong><i>Ask Anton アプリについて</i></strong></summary>

<strong><i><a href="https://aka.ms/choose-anton" target="_blank">Ask Anton</a></i></strong> は、AI の概念や Microsoft Foundry の技術に関する質問に答えることができる生成 AI エージェントです。 <code>https://aka.ms/choose-anton</code> から、2 つのバージョンで使用できます。
<ul>
<li><strong>Azure ベースの</strong>: 最適なエクスペリエンスです (Azure サブスクリプションと Foundry プロジェクト内のモデルのデプロイが必要です)。<i></i></li>
<li><strong>ブラウザーベース</strong>: ブラウザーで小さな言語モデルを使用します (機能は制限されています。古い、または低スペックのデバイスでは動作が遅くなるか "ベーシック" モードでしか動作しないことがあります)。<i></i></li>
</ul>
<blockquote><i>Ask Anton は、サポートされている Microsoft 製品、Microsoft Learn、AI スキル ナビゲーターのコンポーネントのいずれでも<u>ありません</u>。</i></blockquote>
</details>

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>なぜ、出力を読むだけではいけないのですか?</summary>
<div class="concept-body" markdown="1">

出力はエージェントの中で見栄えが良い部分で、他のすべてはそうでないからです。
応答は、流暢で自信満々に間違えている場合もあれば、正解しているものの 3 回再試行され、ツール呼び出しがタイムアウトしていた場合もあります。**トレース**には、応答に至るまでの過程で何が起こったかが示されます。
**[評価]** は、応答を既に真実だと知っていることと比べてスコアリングします。
**[レッド チーミング]** は、敵対的な質問の場合にエージェントが何をするかを示します。

</div>
</details>

**あなたのシナリオ:** あなたは製薬会社 **Caldova** で働いており、製品の発売に向けて急いで準備をしています。 以前のラボで構築したサプライ チェーン アシスタントは、計画チームからの実際の質問に答えるようになり、IT コンプライアンス リードはそれについてより厳しい質問を投げかけています。 *なぜその応答は遅かったのでしょうか? 容量ポリシーについて、作り話をしていませんか? 誰かが本来言ってはいけないことを言わせようとしたらどうなるのですか?* このラボで、あなたはこれら 3 つすべてを意見ではなく証拠を使用して答えます。

まずは、**コア** タスクから始めます。これで、"動く" から "どれだけうまく動くかを証明できる" ようになります。 次の**オプション** タスクは、安全性の次に進みます。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 学習内容

この演習の**コア** タスクを完了することで、次のことができるようになります。

- OpenTelemetry で**エージェントをトレース**し、そのトレースを Azure Monitor にエクスポートし、Foundry ポータルで読み取る。これには、自分のコードの周りに追加したカスタム スパンも含まれます。
- 組み込みのエバリュエータ (根拠性、関連性、類似性) と JSONL データセットを使用し、グラウンド トゥルースと比較して**応答の質を評価する**。

**オプション** タスクでは、さらに次のようなことができます。

- AI レッド チーミング エージェントを使用して、**エージェントをレッドチーミングする**。デプロイしたエージェントに対して敵対的な攻撃戦略と独自のシード プロンプトを実行し、攻撃成功率を確認します。

## このラボの構成

このラボは**モジュール式**です。 各タスクは**単独で完了し、新たに始まる**ように書かれているため、1 つのタスクを選んでそれだけをこなすことができます。 すべてのタスクが 1 つのスタート フォルダー、1 つの仮想環境、1 つの `.env` を共有しているので、一気に進めることもできます。

1. **まずは、「[はじめに](D0-getting-started.md)」から開始しましょう** — Microsoft Foundry プロジェクトを作成し、Application Insights を接続し、スタート コードを入手し、`.env` を設定します。 すべてのタスクはここから始まります。一度にラボ全体を進める場合、これは一度だけで済みます。
2. **好きなタスクをやってみましょう。** 各タスクには必要なセットアップが記載されているので、別々に開始できます。 前のタスクから直接進む場合、短い *[前のタスクから続けていますか?]*
   というメモを上部で選択すれば、繰り返しのセットアップをスキップしてそのまま進められます。

## ラボの概要

まずは、**コア** タスクを完了してください。最後に、内部が見えるエージェントと、その応答のスコアカードが表示されます。 次に、ご希望に応じて**オプション** タスクを追加して、攻撃にどれだけ耐えているかを試してみるとよいでしょう。

<!-- BEGIN GENERATED: task-table - do not edit by hand; run: python tools/generate_lab_blocks.py -->
| セクション | タスク | Level | 時間 |
| --- | --- | --- | --- |
| **コア** | [タスク 1 – エージェントをトレースする](D1-trace-your-agent.md) | ▰▰▰▱▱ L300 | 最大 25 分 |
| **コア** | [タスク 2 – 応答の質を評価する](D2-evaluate-answer-quality.md) | ▰▰▰▱▱ L300 | 最大 35 分 |
| *オプション* | [タスク 3 – エージェントをレッド チーミングする](D3-red-team-your-agent.md) | ▰▰▰▰▱ L400 | 最大 35 分 |

**コア タスク:** 約 **60 分**。 **ラボ全体** (オプションのタスクをすべて含める): 約 **1 時間 35 分**。
<!-- END GENERATED: task-table -->

**自分に合ったパスを選ぶ** — 自分の時間に合ったタスクを選びましょう。

- **コアのみ (最大 1 時間):** タスク 1 〜 2 を行います。
- **すべて (最大 1 時間 35 分):** **タスク 3** (レッド チームのスキャン) を追加します。

> **1 つのエージェント、3 つの質問**: タスク 1 はコード内で作成したエージェントをトレースします。 タスク 2 と 3 はどちらも、[ラボ B](B-integrate-agents-with-enterprise-knowledge-and-m365.md) の**グラウンディングされたナレッジ エージェント**を指定しています。ラボ B をまだ終えていない場合、1 つのコマンドで同等のエージェントが作成されるため、このラボは独立した存在になります — 「[はじめに](D0-getting-started.md)」を参照してください。

## 推測せずに、測定する

このラボの 3 つの技術はそれぞれ異なる疑問に答えるものなので、どれがどれかを明確にしておくとよいでしょう。

- **トレース**は、*「何が起きたか」* に答えます どのスパンにどれくらい時間がかかり、どのツールが呼び出され、どのモデルが送信されたかについての、1 回の実行の記録です。 動作が遅い時や故障している時に使用します。
- **評価**は、*「平均してどれくらい良いか」* に答えます これはデータセットに対するスコアなので、3 つのうち唯一、変更によって状況が良くなったのか悪くなったのかを示します。
- **レッド チーミング**は、*「何をさせることができるか」* に答えます これは敵対的なプローブであり、正常な結果が最低限であって、保証ではありません。

どれも他のものに代わるものではなく、3 つとも運用環境で発見するよりも安く済みます。

## まとめ

このラボでは、次の作業を行いました。

- OpenTelemetry を使用して**エージェントをインストルメント化**し、トレースを Application Insights にエクスポートし、Foundry ポータルで独自のカスタム スパンも含めて読み取りました。
- 組み込みの根拠性、関連性、および類似性のエバリュエータを使用して、グラウンド トゥルースのデータセットに対して**グラウンディングされたエージェントを評価**し、変更間で比較できるスコアを得ました。
- (オプション) 敵対的な攻撃戦略と独自のシード プロンプトを使用して**エージェントをレッド チーミング**し、その結果の攻撃成功率を読み取りました。

これらを合わせて "デモが成功した" ということを、誰かに見せられる証拠にしましょう。

## クリーンアップ

Azure の不要なコストを避けるため、終了後は作成したリソースを削除してください。

1. [Azure portal](https://portal.azure.com) で、Foundry リソースが含まれるリソース グループに移動します。
1. ツール バーで **[リソース グループの削除]** を選択し、リソース グループの名前を入力して確認します。

> タスク 1 で実行するコードによって、作成したエージェント バージョンは削除されます。 エージェントのタスク 2 と 3 の測定は、リソース グループを削除すると削除されます。 `azd` を使用してプロビジョニングした場合は、代わりに `azd down` を実行してください。ただし、Foundry ポータルから作成した Application Insights は別のリソースであり、`azd` では削除されず、リソース グループで削除されることに注意してください。
