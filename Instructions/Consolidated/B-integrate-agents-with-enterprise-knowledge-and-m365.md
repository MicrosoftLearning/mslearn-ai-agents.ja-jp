---
title: エージェントをエンタープライズ ナレッジおよび Microsoft 365 と統合する
lab:
  title: エージェントをエンタープライズ ナレッジおよび Microsoft 365 と統合する
  description: 'Caldova のスタッフ ナレッジ アシスタントを構築する: Foundry IQ を使用してエンタープライズ ドキュメントをグラウンディングし、Microsoft Teams、Microsoft 365 Copilot、Work IQ を介して提供します。 最初から最後まで通して行うことも、タスクを一度に 1 つずつ進めることもできるモジュール式のラボ。'
  type: lab
  id: B
  order: 2
  difficulty: 3
  duration: 35
  access: open
  level: 300
  concepts: 'enterprise knowledge grounding, Foundry IQ, Microsoft 365, Model Context Protocol (MCP)'
  islab: true
  status: draft
---

# エージェントをエンタープライズ ナレッジおよび Microsoft 365 と統合する

**レベル** ▰▰▰▱▱ **L300**  (**L100** 初心者 → **L500** エキスパート)

エージェントは、会社 "独自" の知識に基づいて回答し、従業員が既に行っている業務で役割を果たせるようになると、企業にとって本当に有用なものになります。** このラボでは、**グラウンディングされたエンタープライズ ナレッジ エージェント**を構築し、**Microsoft 365 を介して提供**します。

![Anton](../Media/anton-avatar.png)<br /><strong>AI ガイドの Anton をご紹介します。</strong><br />このラボの随所に **Ask Anton** のヒントがあります。 よりインタラクティブで実践的なサポートが必要な場合、 *[Ask Anton](https://aka.ms/choose-anton)* アプリで Anton とチャットできます。

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
<summary>エンタープライズ ナレッジ グラウンディングとは何か?</summary>
<div class="concept-body" markdown="1">

エージェントを**エンタープライズ ナレッジ**でグラウンディングするとは、エージェントを組織独自のドキュメント (ポリシー、カタログ、手順) に接続し、推測ではなく信頼できる資料に基づいて回答できるようにすることを意味します。 **Foundry IQ** はこれを大規模に行います。ナレッジ ベースのインデックスを作成して*エージェント取得*を実行します。また、各検索の前に**承認**ステップを要求できるため、アプリで制御を維持できます。

[詳細情報 →](https://learn.microsoft.com/azure/ai-foundry/)

</div>
</details>

**シナリオ:** あなたは、製薬会社 **Caldova** で働いており、会社は、前倒しされた製品発売の準備をしているところです。 計画チームと材料チームは、現場のキャパシティ、契約メーカー、技術移転、仕入先に関する質問に絶えず対応していますが、その回答はすべて内部ドキュメントに記載されています。 このラボでは、**Caldova のスタッフ ナレッジ アシスタント**を構築します。まず Foundry IQ を使用してアシスタントをエンタープライズ ドキュメントでグラウンディングし、次にそれを Microsoft Teams と Microsoft 365 Copilot に公開してスタッフが既に行っている業務で利用できるようにし、最後に **Work IQ** を調べて、Microsoft 365 のライブ シグナルをエージェントに取り込みます。

まず、**コア** タスクから始めて、グラウンディングされたエンタープライズ ナレッジ エージェントを可能な限り迅速に機能できるようにします。 その後、一連の**オプション**のタスクを実行して、エージェントを提供および拡張します。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 学習内容

この演習の**コア** タスクを完了すると、次のことができるようになります。

- Microsoft Foundry ポータルで **Foundry IQ** を使用して**エンタープライズ ナレッジ エージェントを作成してグラウンディング**し、エージェントがナレッジ ベースを検索する前に**承認**を要求する。
- **コードからエージェントに接続**し、ナレッジ ツール承認フローを自分で処理する。

**オプション**のタスクを実行すると、さらに次のことができるようになります。

- **エージェントを Microsoft Teams に公開**して、スタッフが Teams でエージェントとチャットできるようにする。
- Copilot エージェントとして**エージェントを Microsoft 365 Copilot に公開する**。
- MCP 経由で **Work IQ を使用して、Microsoft 365 ワークプレースのシグナルをエージェントに取り込む**。

## このラボの構成

このラボは**モジュール式**です。 各タスクは、**単独で完了し、やり直すことができる**ように作成されています。そのため、1 つのタスクを選んで、それだけを実行することができます。 さらに、すべてのコード タスクで 1 つのスターター フォルダー、1 つの仮想環境、1 つの `.env` が共有されているため、一気に進めたい場合は、次の手順で進めることができます。

1. **「[作業の開始](B0-getting-started.md)」から始める** - Microsoft Foundry プロジェクトを作成し (ポータル内で作成するか、1 つの `azd up` コマンドを使用)、スタート コードを入手し、`.env` を設定します。 すべてのタスクはここから始まります。ラボ全体を一気に進める場合は、この作業は 1 回だけで済みます。
2. **任意のタスクを実行する。** 各タスクには必要なセットアップが記載されているため、単独で開始できます。 前のタスクから続けて作業を行う場合、上部の "前のタスクから続ける場合" という**
   注記を参照すると、繰り返しのセットアップをスキップして続行できます。

## ラボの概要

まず、**コア** タスクを完了します。これを完了すると、実際に機能するグラウンディング済みのエンタープライズ ナレッジ エージェントが完成し、コードから呼び出すことができるようになります。 次に、関心のある任意の**オプション**のタスクを展開します。

<!-- BEGIN GENERATED: task-table - do not edit by hand; run: python tools/generate_lab_blocks.py -->
| セクション | タスク | Level | 時間 |
| --- | --- | --- | --- |
| **コア** | [タスク 1 - Foundry IQ ナレッジエージェントを作成し、コードから接続する](B1-create-a-foundry-iq-knowledge-agent.md) | ▰▰▰▱▱ L300 | 最長 35 分 |
| *オプション* | [タスク 2 - エージェントを Microsoft Teams に公開する](B2-publish-to-microsoft-teams.md) 🔒 | ▰▰▱▱▱ L200 | 最長 20 分 |
| *オプション* | [タスク 3 - エージェントを Microsoft 365 Copilot に公開する](B3-publish-to-microsoft-365-copilot.md) 🔒 | ▰▰▱▱▱ L200 | 最長 15 分 |
| *オプション* | [タスク 4 - Work IQ: Microsoft 365 のシグナルをエージェントに取り込む](B4-work-iq-workplace-intelligence.md) 🔒 | ▰▰▰▰▱ L400 | 最長 40 分 |

**コア タスク:** 約 **35 分**。 **ラボ全体** (オプションのタスクをすべて含める): 約 **1 時間 50 分**。

> 🔒 南京錠アイコンが付いたタスクには、お使いのアカウントにはない可能性があるアクセス権を必要とします。 各タスクを開くと簡単なチェックが行われ、アクセス権がない場合の対処方法が指示されます。このラボの他のタスクは、それらのタスクに依存しません。
> <!-- END GENERATED: task-table -->

**自分に合ったパスを選ぶ** - 持ち時間に収まるタスクを選びます。

- **コアのみ (最長 35 分):** タスク 1 を実行します。
- **Core と提供 (最長 1 時間 10 分):****タスク 2** と**タスク 3** も実行して、エージェントを M365 に公開します。
- **すべて (最長 1 時間 50 分):** ライブ ワークプレース インテリジェンスを活用するために **タスク 4** (Work IQ) を追加します。

> **1 つのアシスタント、どこにでも提供**: タスク 1 から 3 はすべて、**同じ**グラウンディングされたエージェント (`caldova-knowledge-agent`) を対象としています。 一度エージェントを構築してグラウンディングすれば (タスク 1)、タスク 2 と 3 では、その同じエージェントを Teams と Copilot に*公開*するだけで済み、新しいコードは必要ありません。 タスク 4 では、独自のエージェントを持つ別の Microsoft 365 機能 (Work IQ) を調べます。

## まとめ

このラボでは、次の作業を行いました。

- Foundry ポータルで **Foundry IQ** を使用して、各ナレッジ検索の前に承認を要求するエンタープライズ ナレッジ エージェントを作成し、**グラウンディング**しました。
- **コードからエージェントに接続**し、承認フローを自分で処理しました。
- (省略可能) エージェントを **Microsoft Teams** と **Microsoft 365 Copilot** に**公開**し、**Work IQ** を調べて、Microsoft 365 のライブ信号をエージェントに取り込みました。

これらを合わせて、エージェントを、グラウンディングされたナレッジ ベースから、組織が日常的に使用する Microsoft 365 サーフェイスへと移行する方法を示します。

## クリーンアップ

完了したら、不要な Azure コストを回避するために、作成したリソースを削除します。

1. [Azure portal](https://portal.azure.com) で、Foundry および Azure AI 検索 のリソースを含むリソース グループに移動します。
1. ツール バーで、**[リソース グループの削除]** を選択してリソース グループの名前を入力し、確認します。

> タスク 4 で実行したコードで作成されたエージェント バージョンは、そのコードによって既に削除されています。 ポータル エージェントは、リソース グループを削除すると削除されます。 `azd` を使用してプロビジョニングした場合は、代わりに `azd down` を実行すると、作成したすべてのものが削除されます。
