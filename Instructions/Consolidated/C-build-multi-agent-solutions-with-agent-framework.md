---
title: Agent Framework を使用してマルチエージェント ソリューションを構築する
lab:
  title: Agent Framework を使用してマルチエージェント ソリューションを構築する
  description: 'Microsoft Agent Framework を使用して Caldova オペレーション エージェントを構築する: まず、ツールを使用する単一のエージェントから始め、次に複数のエージェントを順にオーケストレーションし、さらに A2A プロトコルを使用してプロセス間でリモート エージェントを接続します。 最初から最後まで通して行うことも、タスクを一度に 1 つずつ進めることもできるモジュール式のラボ。'
  type: lab
  id: C
  order: 3
  difficulty: 3
  duration: 30
  access: open
  level: 300
  concepts: 'Microsoft Agent Framework, tools, multi-agent orchestration, A2A protocol'
  islab: true
  status: draft
---

# Agent Framework を使用してマルチエージェント ソリューションを構築する

**レベル** ▰▰▰▱▱ **L300**  (**L100** 初心者 → **L500** エキスパート)

単独のエージェントも有用ですが、 各エージェントが特定の作業に集中し、それを他のエージェントに引き渡すことができるエージェントの "チーム" を構築すれば、本当の業務運営を構築できます。** このラボでは、**Microsoft Agent Framework (MAF)** を使用して Caldova マルチエージェント システムを構築します。ツールを使用する 1 つのエージェントから始めて、プロトコルを介して相互に呼び出し合う一連のリモート エージェントに拡張します。

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
<summary>Microsoft Agent Framework とは</summary>
<div class="concept-body" markdown="1">

**Microsoft Agent Framework(MAF)** は、Microsoft Foundry 上でエージェントを構築するための上位レベルの SDK です。 プレーンな Python 関数に `@tool` を修飾し (スキーマが自動的に生成されます)、`await agent.run(...)` を呼び出します。これにより、ツール呼び出しループ全体が自動的に実行されます。 また、**マルチエージェント** ソリューション (複数のエージェントを同時に動かすオーケストレーション) の構成要素も提供されるため、手作業でプラミングを接続する必要はありません。

[詳細情報 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

**シナリオ:** あなたは、製薬会社 **Caldova** で働いており、会社は、前倒しされた製品発売の準備をしているところです。 このラボでは、Caldova の業務を支援する自動化を構築します。まず、サイト訪問費の請求を提出する単一のエージェントの構築から始め、次に、サイトのフィードバックをトリアージするエージェントのパイプラインを構築し、最後に、個別のプロセスで動作し、プロトコルを介して共同作業を行う一連の専門の転送計画エージェントを構築します。

まず、**コア** タスクから始めて、ツールを使用するエージェントを可能な限り迅速に機能できるようにします。 その後、一連の**オプション**のタスクを実行して、マルチエージェントのパターンをさらに詳しく調べることができます。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 学習内容

この演習の**コア** タスクを完了すると、次のことができるようになります。

- Microsoft Agent Framework を使用して、**カスタム ツールを使用するエージェントを構築する**。Python 関数を `@tool` で修飾し、それを `Agent` に渡して、`agent.run()` を使用してツール呼び出しループを駆動します。

**オプション**のタスクを実行すると、さらに次のことができるようになります。

- 順番に**複数のエージェントをオーケストレーション**し、作業をある専門エージェントから次のエージェントに渡し、すべてのエージェントの出力を収集する。
- 別々のプロセスで実行される**リモート エージェントを接続する**。これらのエージェントは、**エージェント間 (A2A)** プロトコルを使用して相互に呼び出し、ルーティング エージェントによって調整されます。
- 1 つのエージェントの**構造化出力**を独自のコード内の条件付きルーティングに変換して、サポート チケットを**分類してルーティングする**。

## このラボの構成

このラボは**モジュール式**です。 各タスクは、**単独で完了し、やり直すことができる**ように作成されています。そのため、1 つのタスクを選んで、それだけを実行することができます。 さらに、すべてのタスクで 1 つのスターター フォルダー、1 つの仮想環境、1 つの `.env` が共有されているため、一気に進めたい場合は、次の手順で進めることができます。

1. **「[作業の開始](C0-getting-started.md)」から始める** - Microsoft Foundry プロジェクトを作成し (ポータル内で作成するか、1 つの `azd up` コマンドを使用)、スタート コードを入手し、`.env` を設定します。 すべてのタスクはここから始まります。ラボ全体を一気に進める場合は、この作業は 1 回だけで済みます。
2. **任意のタスクを実行する。** 各タスクには必要なセットアップが記載されているため、単独で開始できます。 前のタスクから続けて作業を行う場合、上部の "前のタスクから続ける場合" という**
   注記を参照すると、繰り返しのセットアップをスキップして続行できます。

## ラボの概要

まず、**コア** タスクを完了します。これらを完了すると、実際に機能し、ツールを使用するエージェントが完成します。 次に、関心のある任意の**オプション**のタスクを展開します。

<!-- BEGIN GENERATED: task-table - do not edit by hand; run: python tools/generate_lab_blocks.py -->
| セクション | タスク | Level | 時間 |
| --- | --- | --- | --- |
| **コア** | [タスク 1 - ツールを使用するエージェントを構築する](C1-create-an-agent-with-a-tool.md) | ▰▰▰▱▱ L300 | 最長 30 分 |
| *オプション* | [タスク 2 - 複数のエージェントを順番にオーケストレーションする](C2-orchestrate-multiple-agents.md) | ▰▰▰▱▱ L300 | 最長 30 分 |
| *オプション* | [タスク 3 - リモート エージェントを A2A で接続する](C3-connect-remote-agents-with-a2a.md) | ▰▰▰▰▱ L400 | 最長 30 分 |
| *オプション* | [タスク 4 - サポート チケットを分類してルーティングする](C4-classify-and-route-a-ticket.md) | ▰▰▰▱▱ L300 | 最長 30 分 |

**コア タスク:** 約 **30 分**。 **ラボ全体** (オプションのタスクをすべて含める): 約 **2 時間**。
<!-- END GENERATED: task-table -->

**自分に合ったパスを選ぶ** - 持ち時間に収まるタスクを選びます。

- **コアのみ (最長 99 分):** タスク 1 を実行します。
- **コアと 1 つのパターン (最長 1 時間):** **タスク 2** (シーケンシャル オーケストレーション) または**タスク 4** (分類とルーティング) を追加します。
- **すべて (最長 2 時間):** **タスク 2**、**タスク 3** (A2A でリモート エージェントを接続)、**タスク 4** を追加します。

## 1 つのフレームワーク、1 つのエージェントから多数のエージェントに拡張

このラボのすべてのタスクは **Microsoft Agent Framework** に基づいて構築されているため、ソリューションの規模が拡大しても、コードの形は使い慣れた形のままです。

- **タスク 1** では、**単一**のエージェントを作成します。 `@tool` を使用してツールを説明し、`FoundryChatClient` を使用して `Agent` にアタッチし、`agent.run(...)` を呼び出します。ツール呼び出しループはフレームワークによって自動的に実行されます。
- **Task 2** では、同じクライアントを保持しますが、**複数**のエージェントを作成し、それらを `SequentialBuilder` オーケストレーションに渡します。オーケストレーションはエージェントを順番に実行し、それぞれの出力を収集します。
- **タスク 3** では、エージェントを**個別のプロセス**に分割し、ルーティング エージェントで **A2A プロトコル**を使用してそれらのプロセスを検出して呼び出します。これは同じコラボレーションのアイデアですが、今度はネットワーク上で行います。
- **タスク 4** では、**単一**のエージェントに戻ります。ただし、その**構造化出力** (JSON 分類) がコード内の**条件付きルーティング**を駆動し、各サポート チケットをエスカレーションまたは自動処理します。

最初にシングルエージェントのしくみを理解すると、後でマルチエージェントのパターンが意味を持つようになります。

## まとめ

このラボでは、次の作業を行いました。

- Microsoft Agent Framework を使用して**カスタム ツールを使用するエージェント**を構築しました。
- (省略可能) **複数のエージェントを順番にオーケストレーションして**、作業を段階的にトリアージしました。
- (省略可能) **A2A プロトコル**を使用してプロセス間で**リモート エージェント**を接続し、調整エージェントによってルーティングしました。
- (省略可能) エージェントの**構造化分類** をコード内の**条件付きルーティング**に変換しました。

これらを合わせると、Agent Framework によって、特定のタスクを担う単一のエージェントから、調整されたエージェントのチームにどのように規模を拡張する方法が明らかになります。

## クリーンアップ

完了したら、不要な Azure コストを回避するために、作成したリソースを削除します。

1. [Azure portal](https://portal.azure.com) で、Foundry リソースが含まれているリソース グループに移動します。
1. ツール バーで、**[リソース グループの削除]** を選択してリソース グループの名前を入力し、確認します。

> `azd` を使用してプロビジョニングした場合は、代わりに `azd down` を実行すると、作成したすべてのものが削除されます。
