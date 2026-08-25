---
title: タスク 1 – エージェントを作成して典拠する
lab:
  title: タスク 1 – エージェントを作成して典拠する
  description: Microsoft Foundry ポータルでエージェントを作成し、自分のデータから回答できるように Caldova サプライ チェーン ポリシーに典拠します。
  type: task
  parent: A
  order: 1
  section: core
  difficulty: 2
  duration: 15
  access: open
  level: 200
  concepts: 'agent creation, grounding, file search'
  status: draft
---

# タスク 1 — エージェントを作成して典拠する

これは、**AI エージェントを構築および拡張する**ラボの一部です。初めてご覧になる方は、「[はじめに](A0-getting-started.md)」から開始してください。**

> **必要なもの:** **展開済みのモデル付きの Microsoft Foundry プロジェクト**。 まだお持ちでない場合は、 まず「[はじめに](A0-getting-started.md)」を完了します (オプション A では、ポータルで作成します)。 このタスクはポータルで完全に完了します。ローカル コードや `.env` ファイルは必要ないため、他のタスクから引き継ぐものはありません。

---

典拠はエージェントに信頼できる情報源を提供し、推測せずに正確に答えられるようにします。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>典拠とは</summary>
<div class="concept-body" markdown="1">

Caldova エージェントの最も重要な機能の 1 つは、**典拠**です。
典拠は、サプライ チェーン ポリシー文書のような信頼された発行元の資料を添付し、エージェントは応答を作成するのではなく、"そのデータから" 回答します。**

[詳細情報 →](https://review.learn.microsoft.com/en-us/training/modules/build-extend-ai-agents/2-understand-agents-foundry?branch=pr-en-us-55509)

</div>
</details>

1. エージェントのプレイグラウンドで、**[指示]** を次のように設定します。

    ```prompt
    You are the Caldova supply chain assistant.
    You help planning and materials teams with questions about capacity, contract manufacturers, and materials.

    Guidelines:
    - Always be clear and concise
    - Use the supply chain policy documentation to answer questions accurately
    - If you don't know the answer, admit it and suggest contacting the planning desk directly
    ```

1. サンプルのサプライ チェーン ポリシー ドキュメントをダウンロードします。 新しいブラウザー タブを開き、次に移動します。

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/A-build-and-extend-ai-agents/Python/Supply_Chain_Policy.txt
    ```

    ファイルをローカル コンピューターに保存します。

1. プレイグラウンドに戻って、**[ツール]** セクションで **[追加]** を選び、**[ファイル検索]** を追加します。

1. **[追加]** の右側で **[ファイルをアップロード]** を選択し、ダウンロードした `Supply_Chain_Policy.txt` ファイルを参照してから [**添付**] を選択します。 ファイルのインデックスが作成されるまで待ちます。

1. エージェントを **[保存]** します。

### 典拠されたエージェントをテストする

1. チャットペインに次の内容を入力します。

    ```
    How long does review take for a standard capacity request?
    ```

    エージェントはこの回答でサプライ チェーン ポリシー文書を参照するはずです。

1. 典拠されたデータを使用していることを確認するために、2 つ目の質問を試してみましょう。

    ```
    How much is five weeks of premium contract capacity at expedited priority?
    ```

> ✅ **チェックポイント**: エージェントはアップロードされたポリシー文書を使って、サプライ チェーンの質問に答えます。
> あなたは完全にポータルの中で、エージェントを作成し典拠しました。

---

**次へ:** [タスク 2 — リモート MCP サーバーを接続する](A2-connect-a-remote-mcp-server.md)
