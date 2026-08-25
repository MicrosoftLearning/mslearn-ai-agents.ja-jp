---
title: タスク 4 - カスタム関数ツールを追加する
lab:
  title: タスク 4 - カスタム関数ツールを追加する
  description: 独自の Python 関数で支援されるツールをエージェントに提供し、関数呼び出しのループを処理します。
  type: task
  parent: A
  order: 4
  section: optional
  difficulty: 3
  duration: 25
  access: open
  level: 300
  concepts: 'function tools, function calling, Microsoft Agent Framework'
  status: draft
---

# タスク 4 - カスタム関数ツールを追加する

"「**AI エージェントを構築および拡張する**」ラボの一部です。ここから始める場合は、まず「[作業の開始](A0-getting-started.md)」を完了してください。"**

> **設定 (ここから始める場合):** このタスクには Foundry プロジェクトとスタート コードが必要です。 「[作業の開始](A0-getting-started.md)」をまだ完了していない場合は完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定してください。 ヘルパー ファイル `functions.py` は既にスターター フォルダー内にあります。 次に、VS Code で開いた `Python` フォルダーから、準備ができていることを確認します。

```
python ../setup/check_env.py --task 4
```

> **前のタスクから続ける場合**  同じ `Python` フォルダーで以前のタスクを終了したばかりで、プロジェクト、仮想環境、`.env` が既に設定されている場合、以下の「**設定**」で **functions.py** のレビューに直接進んでください。

---

**目標**: **独自の Python 関数**で支援されるツールをエージェントに提供し、それが行う関数呼び出しを処理します。

**概念の強化**: 関数呼び出しループ - エージェントは、"どの" ツールが呼び出しを行い、"どの" 引数を使用するかを決定します。コードは、それを実行して結果を返します。****

**設定:**

1. `Labfiles/A-build-and-extend-ai-agents/Python` フォルダーで、仮想環境 (`.\labenv\Scripts\Activate.ps1`) をアクティブにし、`PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` が **.env** で設定されていることを確認します (「[作業の開始](A0-getting-started.md)」を参照)。
    次に、キャパシティ プランナーのヘルパー関数が含まれている **functions.py** を確認します。

> **まず試してみる**: **functions.py** の `next_available_slot(site)` を確認します。 モデルを呼び出すタイミングと方法をモデルが認識できるように、モデルに対する単一の `site` パラメーターをどのように記述すればよいでしょうか?  解決策を表示する前に JSON スキーマを記述してみましょう。

<details markdown="1">
<summary>ソリューションを表示する</summary>

**functions_agent.py** 内のコメントに対処します。 参照を追加し、プロジェクトに接続します (タスク 2 と同じパターンです)。 このファイルは構造化されているため、エージェントのセットアップは 1 回実行され、その後 `respond()` 関数が各チャット メッセージを処理し、返信を `run_chat_app()` に渡します。

1. **3 つの関数ツールを定義します。** 各スキーマは、Python 関数の 1 つ (たとえば、スロット検索ツール) を呼び出す方法をモデルに指示します。

    ```python
    # Define the slot lookup function tool
    slot_tool = FunctionTool(
        name="next_available_slot",
        description="Get the next open production slot at a given site.",
        parameters={
            "type": "object",
            "properties": {
                "site": {
                    "type": "string",
                    "description": "site to find the next open production slot at (e.g. 'ashford', 'brightwater', 'calderwood')",
                },
            },
            "required": ["site"],
            "additionalProperties": False,
        },
        strict=True,
    )
    ```

    `cost_tool` (`calculate_transfer_cost`) と `report_tool` (`generate_capacity_report`) を同じ方法で定義し、それぞれの関数のパラメーターを一致させます。

2. **3 つのツールすべてを備えたエージェントを作成します。**

    ```python
    agent = project_client.agents.create_version(
        agent_name="capacity-planner-agent",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="""You are a capacity planning assistant for Caldova that helps
                planners find open production slots and estimate contract manufacturing costs.
                Use the available tools to assist users with their inquiries.""",
            tools=[slot_tool, cost_tool, report_tool],
        ),
    )
    ```

3. `respond()` 内の**ツール呼び出しループを入力**します。応答から各 `function_call` を読み取り、一致する Python 関数を実行し、`FunctionCallOutput` を収集します。

    ```python
    # Process function calls
    for item in response.output:
        if item.type == "function_call":
            result = None
            if item.name == "next_available_slot":
                result = next_available_slot(**json.loads(item.arguments))
            elif item.name == "calculate_transfer_cost":
                result = calculate_transfer_cost(**json.loads(item.arguments))
            elif item.name == "generate_capacity_report":
                result = generate_capacity_report(**json.loads(item.arguments))
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=result,
                )
            )
    ```

    `respond()` の残りの部分 (既に提供済み) では、出力を返送し、最終的な回答をチャット ウィンドウに返します。 出力が同じ **会話**に添付されるため、ツール呼び出しは会話状態で解決されることに注意してください。代わりに `previous_response_id` で返送すると、"次の" メッセージが、"関数呼び出しのツール出力が見つかりません" で失敗します。****

    ```python
    # Send function call outputs back to the model and retrieve a response
    if input_list:
        response = openai_client.responses.create(
            conversation=conversation.id,
            input=input_list,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        )

    return AgentReply(text=response.output_text)
    ```

`python functions_agent.py` を実行します。 ブラウザーでチャット ウィンドウが開きます。一度に **2 つ**のツールを必要とするプロンプトを試してみてください。

```
Find me the next open slot at Brightwater and give me the cost for 5 weeks of premium contract capacity at expedited priority.
```

エージェントは両方の関数を 1 ターンで呼び出し、その結果を結合します。たとえば、次のようになります。

```
The next open slot at Brightwater is the Line 3 Changeover on March 3rd.
The cost for 5 weeks of premium contract capacity at expedited priority is $1,875K.
```

ブラウザー タブを閉じ、ターミナルで **Ctrl + C** キーを押してアプリを停止します (エージェントは終了時に自動的に削除されます)。

</details>

**拡張**: 独自の 4 つ目の関数ツールを追加し、それに言及するように指示を更新します。

<details markdown="1">
<summary>比較: Microsoft Agent Framework を使用した同じエージェント</summary>

ツールごとに 2 つのスキーマと、各 `function_call` と Python 関数を照合するディスパッチ ループを記述しました。 **Microsoft Agent Framework** は両方を削除します。 **functions_agent_maf.py** (完全版を提供済み) を開き、`python functions_agent_maf.py` と共に実行すると、"同じ" キャパシティプランナー アシスタントが生成されます。**

違いは、ツールの定義とループです。 手書きの `FunctionTool` スキーマの代わりに、関数に `@tool` を修飾し、各パラメーターをインラインで説明します。

```python
from agent_framework import tool, Agent
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential
from pydantic import Field
from typing import Annotated

@tool(approval_mode="never_require")
def next_available_slot(
    site: Annotated[str, Field(description="Site to find the next open production slot at (e.g. 'ashford', 'brightwater', 'calderwood')")],
) -> str:
    """Get the next open production slot at a given site."""
    return functions.next_available_slot(site)
```

その後、修飾された関数を使用してエージェントを作成し、ツール呼び出しループ全体を `agent.run()` に処理させます。`response.output` の読み込みも、名前の一致も、出力の返送も必要ありません。

```python
agent = Agent(
    client=FoundryChatClient(
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),
        credential=AzureCliCredential(),
    ),
    name="capacity-planner-agent",
    instructions="You are a capacity planning assistant for Caldova...",
    tools=[next_available_slot, calculate_transfer_cost, generate_capacity_report],
)

# agent.run() decides which tools to call, runs them, and returns the final answer
result = await agent.run(user_message, session=session)
```

結果は同じですが、コードの量は大幅に少なくなります。なぜなら、先ほど手動で記述したプラミングはフレームワークで行われるためです。
まず自分で記述すると、`agent.run()` で "何が" 実行されているかが明確になります。**

</details>

---

**次:** [タスク 5 - キャップストーン: 独自の MCP サーバーを構築する](A5-capstone-build-your-own-mcp-server.md)
