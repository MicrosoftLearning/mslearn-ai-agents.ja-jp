---
title: タスク 4 - サポート チケットを分類してルーティングする
lab:
  title: タスク 4 - サポート チケットを分類してルーティングする
  description: Microsoft Agent Framework を使用して、トリアージ エージェントで Caldova のサポート チケットを分類し、カテゴリと信頼度に基づいてコードで各チケットをルーティングします。
  type: task
  parent: C
  order: 4
  section: optional
  difficulty: 3
  duration: 30
  access: open
  level: 300
  concepts: 'Microsoft Agent Framework, structured output, classification, conditional routing'
  status: draft
---

# タスク 4 - サポート チケットを分類してルーティングする

"これは、「**Agent Framework を使用してマルチエージェント ソリューションを構築する**」ラボの一部です。ここから始める場合は、まず「[作業の開始](C0-getting-started.md)」を完了してください。"**

> **設定 (ここから始める場合):** このタスクには Foundry プロジェクトとスタート コードが必要です。 「[作業の開始](C0-getting-started.md)」をまだ完了していない場合は完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定してください。
> 次に、VS Code で開いた `Python` フォルダーから、準備ができていることを確認します。

```
python ../setup/check_env.py --task 4
```

> **前のタスクから続ける場合**  同じ `Python` フォルダーで前のタスクを終了したばかりで、プロジェクト、仮想環境、`.env` が既に設定されている場合、下記の「** トリアージ エージェントを構築する**」に直接進んでください。

---

すべてのマルチエージェント ジョブにパイプラインが必要なわけではありません。 あるエージェントが "思考" を行い (メッセージを読み取り、決定を行う)、**コード**で、その決定に基づいてアクションを実行する場合もあります。** このタスクでは、Caldova の**サポートデスク トリアージ**を構築します。単一のエージェントで各サポート チケットを分類して信頼度スコアを設定し、それに応じて Python コードでチケットをルーティングします。請求の問題がエスカレーションされ、信頼度の低いチケットは詳細を得るために返送され、残りのチケットは自動的に処理されます。

この作業を行うコツは **構造化出力**です。エージェントに対して、PROSE ではなく小さな JSON オブジェクトで回答するように依頼します。そうすることで、コードでは、その回答に基づいて確実に分岐することができます。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>大きなプロンプトではなくコードでルーティングする理由</summary>
<div class="concept-body" markdown="1">

単一のエージェントに対して、分類と実行内容の決定の**両方**を実行するように依頼することは "できます" が、**決定** (エージェントによる判断) と**ルーティング** (ビジネス ルール) を分離しておくと、システムのテスト、監査、変更が容易になります。** エージェントは小さく予測可能な分類を返します。次に何が起こるかはコードによって決まります。 モデルに**構造化出力** (ここでは `category` と `confidence` を含む JSON オブジェクト) を求めると、コードでその結果に基づいて確定的に分岐することができます。

[詳細情報 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

`Python` フォルダーを開き、「[作業の開始](C0-getting-started.md)」 (`.\labenv\Scripts\Activate.ps1`) で仮想環境をアクティブ化し、下記に進みます。

### トリアージ エージェントを構築する

**ticket_triage.py** を開き、コメント付きの各プレースホルダーにコードを追加します。

1. ファイルに既に含まれているコードを確認します。 `TRIAGE_INSTRUCTIONS` (エージェントに対して、`customer_issue`、`category`、`confidence` を含む JSON オブジェクトを返すように指示する)、`parse_classification` ヘルパー (応答からその JSON を読み取る)、`route_ticket` (ビジネス ルール) に注意してください。 サンプル チケットは `sample_tickets.json` から読み込まれます。

    > **ヒント**: コードを追加する際は、インデントをコメントと揃えてください。

1. ファイルの上部で **Add references** というコメントを見つけて、必要な名前空間を追加します。

    ```python
    # Add references
    from agent_framework import Agent
    from agent_framework.foundry import FoundryChatClient
    from azure.identity import AzureCliCredential
    ```

1. **Create a foundry chat client** というコメントを見つけて、以下のコードを追加します (インデント レベルを維持してください)。

    ```python
    # Create a foundry chat client
    client = FoundryChatClient(
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),
        credential=AzureCliCredential(),
    )
    ```

    **AzureCliCredential** を使用すると、コードから `az login` セッションを使用して Azure に対する認証を行い、**FoundryChatClient** で Foundry プロジェクトに接続できます。

1. **Create the triage agent** というコメントを見つけて、次のコードを追加します。

    ```python
    # Create the triage agent
    agent = Agent(
        client=client,
        name="TicketTriageAgent",
        instructions=TRIAGE_INSTRUCTIONS,
    )
    ```

    共有クライアントによってサポートされる単一のエージェントが、すべての分類を行います。 その動作は完全に `TRIAGE_INSTRUCTIONS` から発生します。

### 各チケットを分類してルーティングする

1. `for` ループ内で、**Create a session, classify the ticket, then parse and route the result** というコメントを見つけて、次のコードを追加します (プレースホルダー `pass` を置き換えてください)。

    ```python
        # Create a session, classify the ticket, then parse and route the result
        session = agent.create_session()
        response = await agent.run(ticket, session=session)

        try:
            classification = parse_classification(response.text)
        except (ValueError, json.JSONDecodeError):
            print("  [review] Could not parse the classification. Send for manual review.")
            continue

        category = classification.get("category", "unknown")
        confidence = float(classification.get("confidence", 0))
        print(f"  Category:   {category} (confidence {confidence:.2f})")
        print(f"  Decision:   {route_ticket(classification)}")
    ```

    チケットごとに新しいセッションを作成し、エージェントを実行して分類を取得し、JSON を解析して、その結果を `route_ticket` に渡します。これは、より大規模なワークフローでノードを 1 つずつ構築していく場合と同じ**分類してから分岐**パターンです。

1. ファイルを保存します (**Ctrl + S** キー)。

### 実行してテストする

1. ターミナルで、アプリにサインインして実行します。

    ```
    az login
    ```

    ```
    python ticket_triage.py
    ```

1. 出力結果を確認します。 各チケットは分類され、ルーティングされます。 次のような出力が表示されます。

    ```
    Ticket 1: The batch record terminal on packaging line B keeps losing its connection even after a full restart.
      Category:   Equipment (confidence 0.95)
      Decision:   [auto] Equipment issue: send troubleshooting steps and raise a maintenance job.

    Ticket 2: Is there a way to see all of our past capacity requests and export them as a report?
      Category:   General (confidence 0.90)
      Decision:   [auto] General question: reply with a help-center answer.

    Ticket 3: We were invoiced twice for the same transfer week last Friday and the statement shows two payments. Can someone fix this?
      Category:   Billing (confidence 0.97)
      Decision:   [escalated] Billing issue routed to the Caldova orders team.
    ```

    > **ヒント**: 不明瞭なチケット (たとえば `"It's not working"`) を `sample_tickets.json` に追加してみてください。 信頼度スコアが低い場合は、`CONFIDENCE_THRESHOLD` に違反し、推測せずに、詳細情報を得るために戻されます。

> ✅**チェックポイント**: コードで単一のエージェントの**構造化出力**を使用して**条件付きルーティング**を駆動しました。つまり、ビジュアル ワークフロー デザイナーを使用せずに、各チケットを分類し、カテゴリと信頼度に基づいて分岐します。

完了したら、「`deactivate`」と入力して、仮想環境を終了します。

---

**次:** オプションのタスクを完了しました。 概要とクリーンアップ手順については、[ラボの概要](C-build-multi-agent-solutions-with-agent-framework.md)に戻ってください。
