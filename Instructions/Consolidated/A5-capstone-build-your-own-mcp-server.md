---
title: 'タスク 5 – キャップストーン: 独自の MCP サーバーを構築する'
lab:
  title: 'タスク 5 – キャップストーン: 独自の MCP サーバーを構築する'
  description: 'キャップストーン: 独自の MCP サーバーを構築し、関数ツールと組み合わせて 1 つの Caldova アシスタントにします。'
  type: task
  parent: A
  order: 5
  section: optional
  difficulty: 4
  duration: 35
  access: open
  level: 400
  concepts: 'MCP server, tool orchestration, Microsoft Agent Framework'
  status: draft
---

# タスク 5 — キャップストーン: 独自の MCP サーバーを構築する

これは、**AI エージェントを構築および拡張する**ラボの一部です。初めてご覧になる方は、「[はじめに](A0-getting-started.md)」から開始してください。**

> **セットアップ (ここから始めます):** これが **キャップストーン**です。 Foundry プロジェクトとスタート コードが必要です。 まだ用意していない場合は、「[はじめに](A0-getting-started.md)」を完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定します。 [タスク 4](A4-add-custom-function-tools.md) の `functions.py` を再利用します — スターター フォルダーに既に存在するため、タスク 4 を終えている必要はありません。 次に、VS Code で開いた `Python` フォルダーから下記を確認します。

```
python ../setup/check_env.py --task 5
```

> **前のタスクから続けている場合** 同じ `Python` フォルダーで以前のタスクを終了したばかりで、プロジェクト、仮想環境、`.env` が既に設定されている場合、下記の「**設定**」に直接進み `server.py` と `client.py` の編集を開始します。

---

**目標**: MCP サーバー上で**独自**のツールをホストし、ラボを 1 つの **Caldova サプライ チェーン アシスタント**にまとめます。これは、**キャパシティの計画と転送の見積もり** (タスク 4 の関数ツール) の両方を行い、"さらに" **実際の材料の在庫と消費量** (ここでホストするツール) をチェックします。**

**強調された概念**: MCP でのサーバーとクライアントの分割 (サーバーがツールを "登録" し、クライアントが、それらを "検出して呼び出す" こと)、および 1 つのエージェントが**複数の種類のツール**を同時に保持できる方法です。**** `respond()` で、各コールを正しい場所に "ルーティング" します。ローカルの Python 関数はインプロセスで動作し、MCP ツールはサーバーセッション上で動作します。**

> **タスク 4 での構築方法**: このキャップストーンは、タスク 4 のキャパシティ プランナー ツールを新しい MCP サーバーと "組み合わせた" ものです。** タスク 4 を完了する必要はありません。これらのツール (`next_available_slot`、`calculate_transfer_cost`、`generate_capacity_report`) は、`client.py` に既製で用意されています。そのため、新しい作業 (MCP サーバーのホスト、および 1 つのエージェントでの両方のツール セットを "組み合わせる"こと) に集中できます。** (タスク 4 を終えましたか? それは話が早いです。見覚えのあるツールが登場します。)

**設定:**

1. `Labfiles/A-build-and-extend-ai-agents/Python` フォルダーで仮想環境 (`.\labenv\Scripts\Activate.ps1`) をアクティブ化し、**.env** に `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` があることを確認します (「[はじめに](A0-getting-started.md)」を参照)。
    **server.py** と **client.py** を編集します。

> **まず試してみてください**: 各ファイルのコメントを使って **server.py** と **client.py** を接続してください。
> 進めながら考えてみてください: なぜ診断出力は `stdout` ではなく、`stderr` に出力される (または抑制される) 必要があるのでしょう。 (ヒント: MCP は stdio 経由で JSON-RPC を読み上げるので、stdout に出力された内容はすべてプロトコル メッセージとして解析され、不要なバナーは、ストリームを破損します。そのため、サーバーは `show_banner=False`で始まります。) また、エージェントが**両方の**ツール セットを取得した場合、コードは、与えられた `function_call` に対し、ローカル関数と MCP ツールのどちらを実行する必要があるのかをどのように判断するでしょうか?**

<details markdown="1">
<summary>ソリューションを表示する</summary>

** `server.py`** で、サーバーを作成し、提供された 2 つの関数をツールとして公開します。

```python
# Add references
from fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP(name="Inventory")

@mcp.tool()
def get_inventory_levels() -> dict:
    ...  # returns the sample inventory dict already in the file

@mcp.tool()
def get_weekly_consumption() -> dict:
    ...  # returns the sample consumption dict already in the file

# Run the MCP server
mcp.run(show_banner=False)
```

** `client.py`** でサーバーに接続し、そのツールを検出し、1 つのエージェントでキャパシティ プランナー ツールと**ともに**登録し、次に、各呼び出しを `respond()` でルーティングします。 チャット UI は非同期イベント ループ上で動作するため、接続コードは最初のメッセージで 1 回だけ実行される非同期 `setup()` に存在します。

1. ファイルの先頭に MCP 参照を追加します。

    ```python
    from mcp import ClientSession, StdioServerParameters
    from mcp.client.stdio import stdio_client
    ```

    `capacity_planner_tools` リストと `local_functions` ディスパッチ ディクショナリー (タスク 4 のツール) はファイルの上部付近で既に提供されており、書き換える必要はありません。

2. `setup()` 内で、stdio 経由でサーバーを起動してセッションを開き、利用可能なツールを一覧にして、それぞれを呼び出し可能なツールとしてラップします:

    ```python
    stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
    stdio, write = stdio_transport
    session = await exit_stack.enter_async_context(ClientSession(stdio, write))
    await session.initialize()
    tools = (await session.list_tools()).tools

    def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            return await session.call_tool(tool_name, kwargs)
        tool_func.__name__ = tool_name
        return tool_func

    functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}

    mcp_function_tools = [
        FunctionTool(
            name=tool.name,
            description=tool.description,
            parameters={"type": "object", "properties": {}, "additionalProperties": False},
            strict=True,
        )
        for tool in tools
    ]
    ```

3. エージェントを**両方**のツールセット、つまりキャパシティ プランナー "および" 材料ツールで作成します。**

    ```python
    agent = project_client.agents.create_version(
        agent_name="caldova-assistant",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="""
            You are the Caldova supply chain assistant. You help planners find open
            production capacity and estimate contract manufacturing costs, and you help
            the materials team check live stock and consumption.

            Capacity planning and transfers:
            - Use the slot and transfer tools to find open capacity, estimate cost, and draft capacity requests.

            Material inventory:
            - Recommend reorder if material inventory < 10 and weekly consumption > 15
            - Flag for review if material inventory > 20 and weekly consumption < 5
            """,
            tools=[*capacity_planner_tools, *mcp_function_tools],
        ),
    )
    ```

4. `respond()`で、各 `function_call` を適切なエグゼキューターにルーティングします。ローカル関数は直接実行され (文字列を返します)、MCP ツールはセッションを介して待機されます。

    ```python
    for item in response.output:
        if item.type == "function_call":
            kwargs = json.loads(item.arguments)

            if item.name in local_functions:
                output_text = local_functions[item.name](**kwargs)          # Task 4 function
            else:
                result = await functions_dict[item.name](**kwargs)          # your MCP tool
                output_text = result.content[0].text

            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=output_text,
                )
            )

    # ...send outputs back, then:
    return AgentReply(text=response.output_text)
    ```

`python client.py` を実行します。 ブラウザーでチャット ウィンドウが開き、最初のメッセージで stdio 経由でサーバーが起動します。 次に、1 回の会話でアシスタントの**両方**の機能を演習するプロンプトを試してみましょう。

```
Plan capacity: find the next open slot at Brightwater and price 5 weeks of premium contract capacity at expedited priority.
```
```
Now check materials — are there any we should reorder?
```

最初のプロンプトはタスク 4 のキャパシティ プランナー機能を呼び出し、2 つ目は MCP の在庫ツールを呼び出します。すべて**同じ**エージェントの、**同じ**チャットで行います。 ブラウザーのタブを閉じて、ターミナルで **Ctrl + C** を押してアプリを停止します。

</details>

**応用**: 3 つ目の MCP ツール (`get_reorder_threshold` など) を追加すると、他のクライアントを変更せずにエージェントがそれを検出します。ルーティング機能は、ローカル関数として認識されないツールを既に処理しているためです。

<details markdown="1">
<summary>比較: Microsoft Agent Framework を使用した同じキャップストーン</summary>

`client.py` では、MCP クライアント (`ClientSession`、`stdio_client`) を手動で接続し、検出された各ツールをラップし、`FunctionTool` スキーマを構築してから、すべての `function_call` (ローカル関数または MCP ツール) を "ルーティング" しました。** **Microsoft Agent Framework** によりそれらのすべてが不要になります。 **client_maf.py** (完成版を提供済み) を開いて `python client_maf.py` で実行します。同じキャップストーン、同じ 2 つのツールセットを 1 つのエージェントに使う動作です。

あなたの `server.py` は変わらず、MCP サーバーの作成者はあなた自身です。 なくなるのは、クライアントの接続とルーティング ループです。 `MCPStdioTool` がサーバーを起動してツールを公開し、あなたがこれをエージェント自身の `@tool` 関数とともに `agent.run()` に渡します。

```python
from agent_framework import tool, Agent, MCPStdioTool

agent = Agent(
    client=FoundryChatClient(...),
    name="caldova-assistant",
    instructions="You are the Caldova supply chain assistant...",
    tools=[next_available_slot, calculate_transfer_cost, generate_capacity_report],
)

async with MCPStdioTool(name="Inventory", command="python", args=["server.py"]) as mcp_tool:
    # One call handles either tool set — no manual "local vs MCP" routing
    result = await agent.run(user_message, tools=mcp_tool, session=session)
```

`if item.name in local_functions ... else ...` ブランチがないことに注目してください。`agent.run()` がモデルで選ばれたツールを呼び出します。それが Python 関数か、MCP サーバーにホストされているツールかは問いません。 最初に手動でルーティングを構築すると、フレームワークが引き継ぐステップを正確に確認できます。

</details>

---

**次へ (任意):** [タスク 6 — アシスタントをホステッド エージェントに昇格させるか、](A6-promote-your-assistant-to-a-hosted-agent.md)または[ラボの概要](A-build-and-extend-ai-agents.md)に**戻る**。
