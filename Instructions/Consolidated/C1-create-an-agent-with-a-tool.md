---
title: タスク 1 - ツールを使用するエージェントを構築する
lab:
  title: タスク 1 - ツールを使用するエージェントを構築する
  description: Microsoft Agent Framework を使用して、カスタム ツールを呼び出す単一のエージェントを構築します。Python 関数を @tool で修飾し、それをエージェントにアタッチし、agent.run() でツール呼び出しループを駆動します。
  type: task
  parent: C
  order: 1
  section: core
  difficulty: 3
  duration: 30
  access: open
  level: 300
  concepts: 'Microsoft Agent Framework, tools, agents'
  status: draft
---

# タスク 1 - ツールを使用するエージェントを構築する

"これは、「**Agent Framework を使用してマルチエージェント ソリューションを構築する**」ラボの一部です。ここから始める場合は、まず「[作業の開始](C0-getting-started.md)」を完了してください。"**

> **設定 (ここから始める場合):** このタスクには Foundry プロジェクトとスタート コードが必要です。 「[作業の開始](C0-getting-started.md)」をまだ完了していない場合は完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定してください。
> 次に、VS Code で開いた `Python` フォルダーから、準備ができていることを確認します。

```
python ../setup/check_env.py --task 1
```

> **前のタスクから続ける場合**  同じ `Python` フォルダーで以前のタスクを終了したばかりで、プロジェクト、仮想環境、`.env` が既に設定されている場合、以下の「**カスタム ツールを使用するエージェントを構築する**」に直接進んでください。

---

有用なエージェントはすべて、チャット以外のことも "実行" できます。** **Microsoft Agent Framework (MAF)** で、通常の Python 関数を記述し、それを `@tool` で修飾し、エージェントに渡して、エージェントに機能を付与します。フレームワークによって、ツールのスキーマが生成され、ツール呼び出しループ全体が自動的に実行されます。 このタスクでは、Caldova の**サイト訪問費エージェント**を構築します。これは、エンジニアのサイト訪問費データを読み取り、それを明細化し、ツールを呼び出して払い戻し請求を財務担当者に "メールで送信" します。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>ツールとは</summary>
<div class="concept-body" markdown="1">

**ツール**とは、エージェントがモデル自身の知識を超えてアクションを実行したり、情報を取得したりできるようにするためにエージェントに付与する関数です。 Agent Framework で、通常の Python 関数を記述し、`@tool` デコレーターを追加します。フレームワークによって、関数シグネチャ (パラメーターの説明を含む) が読み取られ、モデルに必要なスキーマが構築されます。 ツールが必要であるとモデルが判断すると、`agent.run()` が、関数を呼び出し、その結果をモデルにフィードバックして処理を続行します。これらはすべて自動的に実行されます。

[詳細情報 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

`Python` フォルダーを開き、「[作業の開始](C0-getting-started.md)」 (`.\labenv\Scripts\Activate.ps1`) で仮想環境をアクティブ化し、下記に進みます。

### カスタム ツールを使用するエージェントを構築する

**expense_agent.py** を開き、コメント付きの各プレースホルダーにコードを追加します。

1. ファイルに既に含まれているコードを確認します。 その構成要素を次に示します。
    - いくつかの **import** ステートメント。
    - `data.txt` (サイト訪問費データ) を読み込み、そのデータの処理方法を質問し、呼び出しを行う `main` 関数。
    - エージェントを作成して実行する `process_expenses_data` 関数。

    > **ヒント**: コードを追加する際は、インデントをコメントと揃えてください。

1. ファイルの上部で **Add references** というコメントを見つけて、必要な名前空間を追加します。

    ```python
    # Add references
    from agent_framework import tool, Agent
    from agent_framework.foundry import FoundryChatClient
    from azure.identity import AzureCliCredential
    from pydantic import Field
    ```

1. ファイルの下部近くで **Create a tool function for the email functionality** というコメントを見つけて、エージェントが請求を送信するために使用するツールを追加します。

    ```python
    # Create a tool function for the email functionality
    @tool(approval_mode="never_require")
    def submit_claim(
        to: Annotated[str, Field(description="Who to send the email to")],
        subject: Annotated[str, Field(description="The subject of the email.")],
        body: Annotated[str, Field(description="The text body of the email.")],
    ):
        """Submit a Caldova site-visit expense claim by sending an email."""
        print("\nTo:", to)
        print("Subject:", subject)
        print(body, "\n")
    ```

    > **注**: この関数は、コンソールにメールを出力することで電子メールの送信を*シミュレート*します。 実際のアプリケーションでは、SMTP サービスなどを使用して、実際に電子メールを送信します。 `approval_mode="never_require"` を使用すると、エージェントは毎回一時停止して承認を求めることなく、ツールを呼び出せるようになります。

1. `process_expenses_data` 関数に戻り、**Create a foundry chat client** というコメントを見つけて、以下のコードを追加します (インデント レベルを維持してください)。

    ```python
    # Create a foundry chat client
    client = FoundryChatClient(
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),
        credential=AzureCliCredential(),
    )
    ```

    **AzureCliCredential** オブジェクトを使用すると、コードから `az login` セッションを使用して Azure に対する認証を行うことができます。 **FoundryChatClient** オブジェクトは、`.env` のエンドポイントとモデル デプロイ名を使用して Foundry プロジェクトに接続します。

1. **Initialize an agent with the tool and instructions** というコメントを見つけて、以下のコードを追加します。

    ```python
    # Initialize an agent with the tool and instructions
    agent = Agent(
        client=client,
        name="SiteVisitExpenseAgent",
        instructions="""You are an AI assistant for Caldova site-visit expense claims.
                    At the user's request, create an expense claim and use the submit_claim tool to send an email to expenses@caldova.example with the subject 'Site Visit Expense Claim' and a body that contains the itemized expenses with a total.
                    Then confirm to the user that you've done so. Don't ask for any more information from the user, just use the data provided to create the email.""",
        tools=[submit_claim],
    )
    ```

    **Agent** オブジェクトは、クライアント、動作方法を指示する命令、呼び出しが許可されている `submit_claim` ツールを使用して初期化されます。

1. エージェントに続くコード (既に提供済み) を確認します。 会話を保持する**セッション**が作成され、`await agent.run(...)` が呼び出されます。これにより、ツール呼び出しループ全体が実行され、最終応答が `response.text` として返されます。

    ```python
    # Create a session and use the agent to process the expenses data
    try:
        # A session keeps the conversation history across the agent run
        session = agent.create_session()
        # Invoke the agent with the prompt and the site-visit expenses data
        response = await agent.run(f"{prompt}: {expenses_data}", session=session)
        # Display the response
        print(f"\n# Agent:\n{response.text}")
    except Exception as e:
        # Something went wrong
        print(e)
    ```

1. ファイルを保存します (**Ctrl + S** キー)。

### 実行してテストする

1. ターミナルで、アプリにサインインして実行します。

    ```
    az login
    ```

    ```
    python expense_agent.py
    ```

    `az login` を使用すると、`AzureCliCredential` でお使いの Azure アカウントに対して認証できるようになります。

1. 経費データの処理方法を質問されたら、次のように入力します。

    ```
    Submit an expense claim
    ```

1. 出力結果を確認します。 エージェントは、明細化された経費請求メールを作成し (`submit_claim` ツールで印刷)、完了したことを確認します。 次のような出力が表示されます。

    ```
    To: expenses@caldova.example
    Subject: Site Visit Expense Claim
    ...itemized expenses with a total...

    # Agent:
    I've submitted your site-visit expense claim to expenses@caldova.example.
    ```

    > **ヒント**: レート制限を超過したためにアプリが失敗した場合は、数秒待ってからもう一度試してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

> ✅**チェックポイント**: Microsoft Agent Framework を使用して、カスタム ツールを使用する単一のエージェントを構築しました。モデルがツールを呼び出すタイミングを決定し、`agent.run()` ループを処理しました。
> それがこのラボのコアです。 以下のオプションのタスクを使用すると、マルチエージェント ソリューションに拡張されます。

完了したら、「`deactivate`」と入力して、仮想環境を終了します。

---

**次 (省略可能):** [タスク 2 - 複数のエージェントを順番にオーケストレーションする](C2-orchestrate-multiple-agents.md) · [タスク 3 - リモート エージェントを A2A で接続する](C3-connect-remote-agents-with-a2a.md)
