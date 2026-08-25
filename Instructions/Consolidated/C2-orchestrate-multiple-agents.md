---
title: タスク 2 - 複数のエージェントを順番にオーケストレーションする
lab:
  title: タスク 2 - 複数のエージェントを順番にオーケストレーションする
  description: Microsoft Agent Framework を使用して複数のエージェントを順番にオーケストレーションします。サマライザー、分類子、アクション エージェント (それぞれ最後のエージェントに基づいて構築されます) によって、サイトのフィードバックがトリアージされます。
  type: task
  parent: C
  order: 2
  section: optional
  difficulty: 3
  duration: 30
  access: open
  level: 300
  concepts: 'Microsoft Agent Framework, multi-agent orchestration, sequential workflow'
  status: draft
---

# タスク 2 - 複数のエージェントを順番にオーケストレーションする

"これは、「**Agent Framework を使用してマルチエージェント ソリューションを構築する**」ラボの一部です。ここから始める場合は、まず「[作業の開始](C0-getting-started.md)」を完了してください。"**

> **設定 (ここから始める場合):** このタスクには Foundry プロジェクトとスタート コードが必要です。 「[作業の開始](C0-getting-started.md)」をまだ完了していない場合は完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定してください。
> 次に、VS Code で開いた `Python` フォルダーから、準備ができていることを確認します。

```
python ../setup/check_env.py --task 2
```

> **前のタスクから続ける場合**  同じ `Python` フォルダーで以前のタスクを終了したばかりで、プロジェクト、仮想環境、`.env` が既に設定されている場合、以下の「**エージェントを作成する**」に直接進んでください。

---

一部の仕事は、専門家からなる **チーム** によって最善の方法で実行されます。つまり、それぞれの専門家が 1 つのステップを処理し、その結果が次の専門家に渡されます。 Microsoft Agent Framework の **シーケンシャル オーケストレーション** はまさにそれを実現します。エージェントのリストが順番に実行され、各エージェントの出力が収集されます。 このタスクでは、Caldova の**フィードバックのトリアージ** パイプラインを構築します。"サマライザー" がサイトのコメントを要約し、"分類子" がそれにラベルを付け、"アクション" エージェントが次のステップを推奨します。******

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>シーケンシャル オーケストレーションとは</summary>
<div class="concept-body" markdown="1">

**シーケンシャル オーケストレーション** は、複数のエージェントを順番に実行し、実行中の会話を各エージェントから次のエージェントにフィードします。 これは、タスクを順序付けられた段階 (要約、分類、決定) に明確に分割し、各段階でその前の段階の出力を活用する場合に適しています。 Agent Framework では、`SequentialBuilder` を使用してシーケンシャル オーケストレーションを構築し、参加者エージェントを実行順に一覧表示します。

[詳細情報 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

`Python` フォルダーを開き、「[作業の開始](C0-getting-started.md)」 (`.\labenv\Scripts\Activate.ps1`) で仮想環境をアクティブ化し、下記に進みます。

### エージェントを作成する

**feedback_agents.py** を開き、コメント付きの各プレースホルダーにコードを追加します。

1. ファイルに既に含まれているコードを確認します。 少し時間を取って、`main` 関数内のエージェント**命令**の 3 つのセット (サマライザー、分類子、アクション) を読みます。これらは、各エージェントの実行内容を定義します。

    > **ヒント**: コードを追加する際は、インデントをコメントと揃えてください。

1. ファイルの上部で **Add references** というコメントを見つけて、必要な名前空間を追加します。

    ```python
    # Add references
    from agent_framework import Message
    from agent_framework.foundry import FoundryChatClient
    from agent_framework.orchestrations import SequentialBuilder
    from azure.identity import AzureCliCredential
    ```

1. **Create the chat client** というコメントを見つけて、以下のコードを追加します (インデント レベルを維持してください)。

    ```python
    # Create the chat client
    credential = AzureCliCredential()
    chat_client = FoundryChatClient(
        credential=credential,
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),
    )
    ```

    **AzureCliCredential** を使用すると、コードから `az login` セッションを使用して Azure に対する認証を行い、**FoundryChatClient** で Foundry プロジェクトに接続できます。 3 つのエージェントすべてがこの 1 つのクライアントを共有します。

1. "**Create agents**" というコメントを見つけて、共有クライアントから 3 つのエージェントを作成する次のコードを追加します。

    ```python
    # Create agents
    summarizer_agent = chat_client.as_agent(
        name="summarizer",
        instructions=summarizer_instructions,
    )

    classifier_agent = chat_client.as_agent(
        name="classifier",
        instructions=classifier_instructions,
    )

    action_agent = chat_client.as_agent(
        name="action",
        instructions=action_instructions,
    )
    ```

1. **Initialize the current feedback** というコメントを見つけて、パイプラインでトリアージするためのサンプルのサイト フィードバックを追加します。

    ```python
    # Initialize the current feedback
    feedback="""
    I use the line-scheduling app before every changeover, and it works well overall.
    But when I'm checking the schedule at night on the floor, the bright screen is really harsh on my eyes.
    If you added a dark mode option, it would make it much more comfortable to use in low light.
    """
    ```

### 順次オーケストレーションを作成する

1. **Build sequential orchestration**というコメントを見つけて、パイプラインを定義する次のコードを追加します。

    ```python
    # Build sequential orchestration
    workflow = SequentialBuilder(
        participants=[summarizer_agent, classifier_agent, action_agent],
        output_from="all",
    ).build()
    ```

    エージェントはリストされた順にフィードバックを処理します。 `output_from="all"` により、最後のエージェントだけでなく "すべて" のエージェントからの出力が確実に収集されます。**

1. **Run and collect outputs** というコメントを見つけて、次のコードを追加します。

    ```python
    # Run and collect outputs
    result = await workflow.run(f"Site feedback: {feedback}")
    outputs = result.get_outputs()
    ```

    このコードはオーケストレーションを実行し、参加している各エージェントからの出力を収集します。

1. **Display output** というコメントを見つけて、次のコードを追加します。

    ```python
    # Display outputs
    i = 1
    for response in outputs:
        for msg in cast(list[Message], response.messages):
            name = msg.author_name or ("assistant" if msg.role == "assistant" else "user")
            print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
            i += 1
    ```

    これは、オーケストレーションによって収集された各メッセージを書式設定し、それを作成したエージェントのラベルを付けて出力します。

1. ファイルを保存します (**Ctrl + S** キー)。

### 実行してテストする

1. ターミナルで、アプリにサインインして実行します。

    ```
    az login
    ```

    ```
    python feedback_agents.py
    ```

1. 出力結果を確認します。 各エージェントでステップが 1 つずつ処理され、次のような出力が表示されます。

    ```
    Site team requests a dark mode option for comfortable night-shift use.
    Feature request
    Log as an enhancement request to add a dark mode for night-shift use.
    ------------------------------------------------------------
    01 [summarizer]
    Site team requests a dark mode option for comfortable night-shift use.
    ------------------------------------------------------------
    02 [classifier]
    Feature request
    ------------------------------------------------------------
    03 [action]
    Log as an enhancement request to add a dark mode for night-shift use.
    ```

    > **ヒント**: レート制限を超過したためにアプリが失敗した場合は、数秒待ってからもう一度試してください。 `feedback` 文字列を苦情または称賛の内容に書き換えてもう一度実行し、分類と推奨されるアクションがどのように変更されるかを確認してみてください。

> ✅**チェックポイント**: Microsoft Agent Framework を使用して 3 つのエージェントを順番にオーケストレーションし、作業をある専門エージェントから次の専門エージェントに渡し、すべてのエージェントの出力を収集しました。

完了したら、「`deactivate`」と入力して、仮想環境を終了します。

---

**次 (省略可能):** [タスク 3 - リモート エージェントを A2A で接続する](C3-connect-remote-agents-with-a2a.md)
