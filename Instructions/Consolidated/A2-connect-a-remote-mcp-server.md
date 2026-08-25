---
title: タスク 2 – リモート MCP サーバーを接続する
lab:
  title: タスク 2 – リモート MCP サーバーを接続する
  description: リモートのモデル コンテキスト プロトコル (MCP) サーバーに接続し、ツール承認要求をコードで処理して、エージェントを拡張します。
  type: task
  parent: A
  order: 2
  section: core
  difficulty: 3
  duration: 20
  access: open
  level: 300
  concepts: 'tools, Model Context Protocol (MCP), approvals'
  status: draft
---

# タスク 2 — リモート MCP サーバーを接続する

これは、**AI エージェントを構築および拡張する**ラボの一部です。初めてご覧になる方は、「[はじめに](A0-getting-started.md)」から開始してください。**

> **設定 (ここから始めます):** このタスクには Foundry プロジェクトとスタート コードが必要です。 まだ用意していない場合は、「[はじめに](A0-getting-started.md)」を完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定します。
> 次に、VS Code で開いた `Python` フォルダーから準備ができていることを確認します。

```
python ../setup/check_env.py --task 2
```

> **前のタスクから続けている場合** 同じ `Python` フォルダーで前のタスクを終了したばかりで、プロジェクト、仮想環境、`.env` が既に設定されている場合、下記の「**エージェントを MCP サーバーに接続する**」に直接進んでください。

---

**モデル コンテキスト プロトコル (MCP)** は、サーバーがホストするツールをエージェントが検出し呼び出すことができるようにします。 舞台裏では、Caldova プラットフォーム チームが Azure でサプライ チェーン プラットフォームを再構築しています。そのため、このタスクでは、エージェントを **Microsoft Learn Docs** リモート MCP サーバー に接続することで、信頼できる最新の Azure ドキュメントをオンデマンドで取得できるアシスタントをチームに提供します。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>MCP とは</summary>
<div class="concept-body" markdown="1">

**モデル コンテキスト プロトコル (MCP)** は、エージェントが実行時にツールを検出できるようにすることでこれを解決します。 MCP では、ツールはライブ カタログとして機能する**サーバー**上に存在します。 エージェントは (**クライアント**を通じて) サーバーに利用可能なツールを要求し、オンデマンドで呼び出します。

[詳細情報 →](https://review.learn.microsoft.com/en-us/training/modules/build-extend-ai-agents/5-connect-agents-to-mcp?branch=pr-en-us-55509)

</div>
</details>

`Python` フォルダーを開き、「[はじめに](A0-getting-started.md)」から仮想環境をアクティブ化し (`.\labenv\Scripts\Activate.ps1`)、下記に進みます。

### エージェントを MCP サーバーに接続する

**remote_mcp_agent.py** を開き、コメント付きのプレースホルダーごとにコードを追加します。

> **ヒント**: コードを追加する際は、コメントとインデントを揃えておきましょう。

1. **参照を追加します**。

    ```python
    # Add references
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.projects.models import PromptAgentDefinition, MCPTool
    from openai.types.responses.response_input_param import McpApprovalResponse, ResponseInputParam
    ```

1. **エージェントのクライアントに接続します**。

    ```python
    # Connect to the agents client
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
        project_client.get_openai_client() as openai_client,
    ):
    ```

1. **エージェントの MCP ツールを初期化します**。これが Microsoft Learn Docs MCP サーバーでエージェントをポイントします。

    ```python
    # Initialize agent MCP tool
    mcp_tool = MCPTool(
        server_label="api-specs",
        server_url="https://learn.microsoft.com/api/mcp",
        require_approval="always",
    )
    ```

1. **MCP ツールで新しいエージェントを作成します**。

    ```python
    # Create a new agent with the MCP tool
    agent = project_client.agents.create_version(
        agent_name="platform-docs-agent",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="You are a platform engineering assistant for Caldova. Use the available MCP tools to look up trusted Azure documentation and help the team build and operate the supply chain platform.",
            tools=[mcp_tool],
        ),
    )
    print(f"Agent created (id: {agent.id}, name: {agent.name}, version: {agent.version})")
    ```

1. **会話スレッドを作成します**。

    ```python
    # Create a conversation thread
    conversation = openai_client.conversations.create()
    print(f"Created conversation (id: {conversation.id})")
    ```

1. **MCP ツールをトリガーする初期要求を送信します**。

    ```python
    # Send initial request that will trigger the MCP tool
    response = openai_client.responses.create(
        conversation=conversation.id,
        input="Give me the Azure CLI commands to deploy our product catalog API to an Azure Container App with a managed identity.",
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    )
    ```

1. **MCP 承認要求を処理します**。ツールには承認が必要なため、エージェントは各呼び出しの前に一時停止しアクセス許可を要求します。 このループは各要求を自動承認します。

    ```python
    # Process any MCP approval requests that were generated
    while True:
        input_list: ResponseInputParam = []
        for item in response.output:
            if item.type == "mcp_approval_request":
                if item.server_label == "api-specs" and item.id:
                    input_list.append(
                        McpApprovalResponse(
                            type="mcp_approval_response",
                            approve=True,
                            approval_request_id=item.id,
                        )
                    )

        # No more approvals needed -> the agent has produced its final response
        if not input_list:
            break

        response = openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        )

    print(f"\nAgent response: {response.output_text}")
    ```

1. **エージェントのバージョンをクリーンアップします**。テスト エージェントを忘れないようにしましょう。

    ```python
    # Clean up resources by deleting the agent version
    project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
    print("Agent deleted")
    ```

1. ファイルを保存します (**CTRL + S**)。

### 実行してテストする

1. ターミナルでアプリにサインインして実行します。

    ```
    az login
    ```

    ```
    python remote_mcp_agent.py
    ```

1. エージェントが自身を作成し、MCP ツール (ループによって自動的に承認済み) を呼び出し、ライブ ドキュメントを使用して回答する様子を確認します。 次のような出力が表示されます。

    ```
    Agent created (id: platform-docs-agent:2, name: platform-docs-agent, version: 2)
    Created conversation (id: conv_...)

    Agent response: Here are Azure CLI commands to create an Azure Container App with a managed identity:
    ...
    Agent deleted
    ```

1. 別の Azure サービスについてたずねるように `input` 文字列を変更して、もう一度実行してみてください。

> ✅ **チェックポイント**: あなたは典拠されたエージェントを構築し、"さらに" リモート MCP サーバーを通じて外部ツールで承認処理も含めてエージェントを拡張しました。** これがこのラボの核であり、次の内容はすべて任意です。

完了したら、ターミナルに「`deactivate`」と入力して、仮想環境を終了します。

---

**次へ (任意):** [タスク 3 — クライアント アプリからエージェントを呼び出す](A3-call-your-agent-from-a-client-app.md) · [タスク 4 — カスタム機能ツールを追加する](A4-add-custom-function-tools.md)
