---
lab:
  title: モデル コンテキスト プロトコル (MCP) ツールを使用してエージェントを拡張する
  description: モデル コンテキスト プロトコル (MCP) サーバー ツールを統合して、エージェント機能を拡張します。
---

# モデル コンテキスト プロトコル (MCP) ツールを使用して AI エージェントを開発する

この演習では、Microsoft Foundry VS Code 拡張機能を使用して、モデル コンテキスト プロトコル (MCP) サーバー ツールを使用して外部データ ソースと API にアクセスできるエージェントを作成します。 エージェントが、MCP ツールを使用して最新の情報を取得し、カスタム サービスと対話できるようになります。

この演習の所要時間は約 **60** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

この演習を開始するには、以下のものが必要です。
- Visual Studio Code がインストールされていること
- 有効な Azure サブスクリプション
- Python バージョン 3.10 以降がインストールされている

## Microsoft Foundry VS Code 拡張機能をインストールする

VS Code 拡張機能をインストールして設定することから始めましょう。

1. Visual Studio Code を開きます。

1. 左側のペインから **[拡張機能]** を選択します (または **Ctrl + Shift + X** キーを押します)。

1. 検索バーに「**Microsoft Foundry**」と入力し、Enter キーを押します。

1. Microsoft から **[Microsoft Foundry]** 拡張機能を選択し、**[インストール]** をクリックします。

1. インストールが完了したら、Visual Studio Code の左側のプライマリ ナビゲーション バーに拡張機能が表示されていることを確認します。

## Azure にサインインしてプロジェクトを作成する

次に、Azure リソースに接続し、新しい AI Foundry プロジェクトを作成します。

1. VS Code のサイド バーで、**[Microsoft Foundry]** 拡張機能のアイコンを選択します。

1. [リソース] ビューで、**[Azure にサインイン...]** を選択し、認証プロンプトに従います。

   > **注**:既にサインインしている場合、このオプションは表示されません。

1. Foundry 拡張機能ビューで **[リソース]** の横にある **[+]** (プラス) アイコンを選択して、新しい Foundry プロジェクトを作成します。

1. ドロップダウンから Azure サブスクリプションを選択します。

1. 新しいリソース グループを作成するか、既存のリソース グループを使用するかを選択します。
   
   **新しいリソース グループを作成する場合:**
   - **[新しいリソース グループの作成]** を選択し、Enter キーを押します
   - リソース グループの名前 (例: "rg-ai-agents-lab") を入力し、Enter キーを押します
   - 使用可能なオプションから場所を選択し、Enter キーを押します
   
   **既存のリソース グループを使用する場合:**
   - 使用するリソース グループを一覧から選択し、Enter キーを押します

1. テキストボックスに Foundry プロジェクトの名前 (例: "ai-agents-project") を入力し、Enter キーを押します。

1. プロジェクトのデプロイが完了するまで待ちます。 ポップアップが表示され、"プロジェクトが正常にデプロイされました" というメッセージが表示されます。

## モデルをデプロイする

このタスクでは、エージェントで使用するモデル カタログからモデルをデプロイします。

1. "プロジェクトが正常にデプロイされました" というポップアップが表示されたら、**[モデルのデプロイ]** ボタンを選択します。 これにより、モデル カタログが開きます。

   > **ヒント**: モデル カタログにアクセスするには、[リソース] セクションで **[モデル]** の横にある **[+]** アイコンを選択します。または、**F1** キーを押し、コマンド **Microsoft Foundry: Open Model Catalog**を実行して、モデル カタログにアクセスすることもできます。

1. モデル カタログで、**gpt-4** モデルを見つけます (検索バーを使用すると、すばやく見つけることができます)。

    ![Foundry VS Code 拡張機能の [モデル カタログ] のスクリーンショット。](Media/vs-code-model.png)

1. gpt-4 モデルの横にある **[デプロイ]** を選択します。

1. デプロイ設定を構成します。
   - **デプロイ名**:"gpt-4" のような名前を入力します
   - **デプロイの種類**:**[グローバル標準]** (グローバル標準を使用できない場合は **[標準]**) を選択します
   - **モデルのバージョン**: デフォルトのままにする
   - **1 分あたりのトークン数**:デフォルトのままにする

1. 左下隅にある **[Microsoft Foundry にデプロイ]** を選択します。

1. 確認ダイアログで **[デプロイ]** を選択して、モデルをデプロイします。

1. デプロイが完了するまで待ちます。 デプロイしたモデルが、[リソース] ビューの **[モデル]** セクションに表示されます。

1. プロジェクトのデプロイ名を右クリックし、**[プロジェクト エンドポイントのコピー]** を選択します。 次のステップでエージェントを Foundry プロジェクトに接続するために、この URL が必要になります。

   <img src="Media/vs-code-endpoint.png" alt="Screenshot of copying the project endpoint in the Foundry VS Code extension." width="550">

## スタート コード リポジトリをクローンする

この演習では、Foundry プロジェクトに接続し、MCP サーバー ツールを使用するエージェントを作成するのに役立つスタート コードを使用します。

1. VS Code の **[ようこそ]** タブに移動します (メニュー バーから **[ヘルプ] > [ようこそ]** を選択すると開きます)。

1. **[Git リポジトリの複製]** を選択し、スタート コード リポジトリの URL `https://github.com/MicrosoftLearning/mslearn-ai-agents.git` を入力します。

1. 新しいフォルダーを作成し、**[リポジトリの宛先として選択]** を選択し、プロンプトが表示されたらクローンされたリポジトリを開きます。

1. エクスプローラー ビューで、**[Labfiles/03-mcp-integration/Python]** フォルダーに移動して、この演習のスタート コードを見つけます。

1. **requirements.txt** ファイルを右クリックし、**[統合ターミナルで開く]** を選択します。

1. ターミナルで、次のコマンドを入力して、必要な Python パッケージをインストールします。

    ```
    pip install -r requirements.txt
    ```

1. **.env** ファイルを開き、**[your_project_endpoint]** プレースホルダーをプロジェクトのエンドポイント (Microsoft Foundry 拡張機能のプロジェクト デプロイ リソースからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名に設定されていることを確認します。 これらの変更を行った後、**Ctrl + S** を使用してファイルを保存します。

これで、MCP サーバー ツールを使用して外部データ ソースと API にアクセスする AI エージェントを作成する準備ができました。

## Azure AI エージェントをリモート MCP サーバーに接続する

このタスクでは、リモート MCP サーバーに接続し、AI エージェントを準備し、ユーザー プロンプトを実行します。

1. コード エディターで **agent.py** ファイルを開きます。

   > **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 コメントのインデント レベルをガイドとして利用します。

1. コメント **Add references** を見つけて、クラスをインポートする次のコードを追加します。

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import PromptAgentDefinition, MCPTool
   from openai.types.responses.response_input_param import McpApprovalResponse, ResponseInputParam
    ```

1. コメント **Connect to the agents client** を見つけて、現在の Azure 資格情報を使用して Azure AI プロジェクトに接続する次のコードを追加します。

    ```python
   # Connect to the agents client
   with (
       DefaultAzureCredential() as credential,
       AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
       project_client.get_openai_client() as openai_client,
   ):
    ```

1. コメント **Initialize agent MCP tool** の下に、次のコードを追加します。

    ```python
   # Initialize agent MCP tool
   mcp_tool = MCPTool(
       server_label="api-specs",
       server_url="https://learn.microsoft.com/api/mcp",
       require_approval="always",
   )
    ```

    このコードは、Microsft Learn Docs リモート MCP サーバーに接続します。 これは、クライアントが Microsoft の公式ドキュメントから信頼できる最新の情報に直接アクセスできるようにするクラウド ホスト サービスです。

1. コメント **Create a new agent with the MCP tool** の下に、次のコードを追加します。

    ```python
   # Create a new agent with the MCP tool
   agent = project_client.agents.create_version(
       agent_name="MyAgent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions="You are a helpful agent that can use MCP tools to assist users. Use the available MCP tools to answer questions and perform tasks.",
           tools=[mcp_tool],
       ),
   )
   print(f"Agent created (id: {agent.id}, name: {agent.name}, version: {agent.version})")
    ```

    このコードで、エージェントに指示を与え、MCP ツール定義を提供します。

1. コメント **Create a conversation thread** を見つけて、次のコードを追加します。

    ```python
   # Create a conversation thread
   conversation = openai_client.conversations.create()
   print(f"Created conversation (id: {conversation.id})")
    ```

1. コメント **Send initial request that will trigger the MCP tool** を見つけて、次のコードを追加します。

    ```python
   # Send initial request that will trigger the MCP tool
   response = openai_client.responses.create(
       conversation=conversation.id,
       input="Give me the Azure CLI commands to create an Azure Container App with a managed identity.",
       extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
   )
    ```

1. コメント **Process any MCP approval requests that were generated** を見つけて、次のコードを追加します。

    ```python
   # Process any MCP approval requests that were generated
   input_list: ResponseInputParam = []
   for item in response.output:
       if item.type == "mcp_approval_request":
           if item.server_label == "api-specs" and item.id:
               # Automatically approve the MCP request to allow the agent to proceed
               input_list.append(
                   McpApprovalResponse(
                       type="mcp_approval_response",
                       approve=True,
                       approval_request_id=item.id,
                   )
               )

   print("Final input:")
   print(input_list)
    ```

    このコードは、エージェントの応答で MCP 承認要求をリッスンし、自動的に承認します。

1. コメント **Send the approval response back and retrieve a response** を見つけて、次のコードを追加します。

    ```python
   # Send the approval response back and retrieve a response
   response = openai_client.responses.create(
       input=input_list,
       previous_response_id=response.id,
       extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
   )

   print(f"\nAgent response: {response.output_text}")
    ```

1. コメント **Clean up resources by deleting the agent version** を見つけて、次のコードを追加します。

    ```python
   # Clean up resources by deleting the agent version
   project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
   print("Agent deleted")
    ```

1. 完了したら、コード ファイルを保存 (*CTRL + S*) します。

### アプリケーションの実行

1. 統合ターミナルで、次のコマンドを入力してアプリケーションを実行します。

    ```
   python agent.py
    ```

1. エージェントがプロンプトを処理するまで待ち、MCP サーバーを使用して、要求された情報を取得するための適切なツールを見つけます。 次のような出力が表示されるはずです。

    ```
    Agent created (id: MyAgent:2, name: MyAgent, version: 2)
    Created conversation (id: conv_086911ecabcbc05700BBHIeNRoPSO5tKPHiXRkgHuStYzy27BS)
    Final input:
    [{'type': 'mcp_approval_response', 'approve': True, 'approval_request_id': '{approval_request_id}'}]

    Agent response: Here are Azure CLI commands to create an Azure Container App with a managed identity:

    **1. For a System-assigned Managed Identity**
    ```sh
    az containerapp create \
    --name <CONTAINERAPP_NAME> \
    --resource-group <RESOURCE_GROUP> \
    --environment <CONTAINERAPPS_ENVIRONMENT> \
    --image <CONTAINER_IMAGE> \
    --identity 'system'
    ```

    [続き...]

    エージェントが削除されました
    ```

    Notice that the agent was able to invoke the MCP tool to automatically fulfill the request.

1. You can update the input in the request to ask for different information. In each case, the agent will attempt to find technical documentation by using the MCP tool.

## Connect an Azure AI Agent to custom MCP server tools

In addition to connecting to remote MCP servers, you can also create your own custom MCP server tools and connect them to your agent. A Model Context Protocol (MCP) Server is a component that hosts callable tools. These tools are Python functions that can be exposed to AI agents. When tools are annotated with `@mcp.tool()`, they become discoverable to the client, allowing an AI agent to call them autonomously during a conversation or task. In this task, you'll add tools that will allow an agent to perform inventory inquiries and recommendations.

### Create an MCP server with custom tools

1. Open the **server.py** file in the code editor.

    In this code file, you'll define the tools the agent can use to simulate a backend service for the retail store. Notice the server setup code at the top of the file. It uses `FastMCP` to quickly spin up an MCP server instance named "Inventory". This server will host the tools you define and make them accessible to the agent during the lab.

1. Under the comment **Add references**, add the following code:

    ```python
   # Add references
   from mcp.server.fastmcp import FastMCP
    ```

1. コメント **Create an MCP server** の下に、次のコードを追加して新しい MCP サーバー インスタンスを作成します。

    ```python
   # Create an MCP server
   mcp = FastMCP(server_label="Inventory")
    ```

    このコードにより、"Inventory" というラベルで新しい MCP サーバーが初期化されます。

1. コメント **Add an inventory check mcp tool** を見つけ、関数定義の上に次のデコレーターを追加します。

    ```python
   # Add an inventory check mcp tool
   @mcp.tool()
   def get_inventory_levels() -> dict:
      # continued...
    ```

    このディクショナリは、サンプル インベントリを表します。 `@mcp.tool()` デコレーターは、MCP サーバー上にツールとして関数を登録し、LLM が関数を検出できるようにします。 

1. コメント **Add a weekly sales mcp tool** を見つけ、関数定義の上に次のデコレーターを追加します。

    ```python
   # Add a weekly sales mcp tool
   @mcp.tool()
   def get_weekly_sales() -> dict:
      # continued...
    ```

1. コメント **Run the MCP server** を見つけ、次のコードを追加してサーバーを起動します。

    ```python
   # Run the MCP server
   mcp.run()
    ```

    このコードは MCP サーバーを起動し、エージェントがツールを検出して使用できるようにします。

1. ファイルを保存します (*CTRL+S*)。

### MCP クライアントを実装する

MCP クライアントは、MCP サーバーに接続してツールの検出と呼び出しを行うコンポーネントです。 エージェントとサーバーでホストされる関数の橋渡し役のように考えてください。ユーザー プロンプトへの応答で動的ツールを使用できるようになります。

1. **client.py** ファイルに移動します。

1. コメント **Add references** を見つけて、クラスをインポートする次のコードを追加します。

    ```python
   # Add references
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
    ```

1. **connect_to_server** メソッドで、コメント **Start the MCP server** を見つけ、次のコードを追加します。

    ```python
   # Start the MCP server
   stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
   stdio, write = stdio_transport
    ```

    標準の実稼働セットアップでは、サーバーはクライアントとは別に実行されます。 ただし、このラボのため、クライアントは標準の入出力転送を使用してサーバーの起動を担当します。 これにより、2 つのコンポーネント間に軽量な通信チャネルが作成され、ローカル開発のセットアップが簡略化されます。

1. コメント **Create an MCP client session** を見つけて、次のコードを追加します。

    ```python
   # Create an MCP client session
   session = await exit_stack.enter_async_context(ClientSession(stdio, write))
   await session.initialize()
    ```

    これにより、前のステップからの入出力ストリームを使用して新しいクライアント セッションが作成されます。 `session.initialize` を呼び出すと、MCP サーバーに登録されているツールの検出と呼び出しを行うセッションが準備されます。

1. コメント **List available tools** の下に、次のコードを追加して、クライアントがサーバーに接続されていることを確認します。

    ```python
   # List available tools
   response = await session.list_tools()
   tools = response.tools
   print("\nConnected to server with tools:", [tool.name for tool in tools]) 
    ```

    これで、クライアント セッションを Azure AI エージェントで使用する準備ができました。

### MCP ツールをエージェントに接続する

このタスクでは、MCP サーバー ツールをエージェントに接続して、ユーザーのプロンプトに応じて呼び出すことができます。

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 コメントのインデント レベルをガイドとして利用します。

1. **chat_loop** メソッドで、コメント **Build a function for each tool** を見つけ、次のコードを追加します。

    ```python
    # Build a function for each tool
    def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            result = await session.call_tool(tool_name, kwargs)
            return result
        
        tool_func.__name__ = tool_name
        return tool_func

    # Store the functions in a dictionary for easy access when processing function calls
    functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
    ```

    このコードは、AI エージェントから呼び出すことができるように、MCP サーバーで使用可能なツールを動的にラップします。 各ツールは、エージェントで呼び出しができる非同期関数に変換されます。
    
1. コメント **Create FunctionTool definitions for the agent** を見つけ、次のコードを追加します。

    ```python
   # Create FunctionTool definitions for the agent
   mcp_function_tools: FunctionTool = []
   for tool in tools:
       function_tool = FunctionTool(
           name=tool.name,
           description=tool.description,
           parameters={
               "type": "object",
               "properties": {},
               "additionalProperties": False,
           },
           strict=True
       )
       mcp_function_tools.append(function_tool)
    ```

1. コメント **Create the agent** を見つけて、次のコードを追加します。

    ```python
   # Create the agent
   agent = project_client.agents.create_version(
       agent_name="inventory-agent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions="""
           You are an inventory assistant. Here are some general guidelines:
           - Recommend restock if item inventory < 10  and weekly sales > 15
           - Recommend clearance if item inventory > 20 and weekly sales < 5
           """,
           tools=mcp_function_tools
       ),
   )
    ```

   これらの指示とツールを使用すると、エージェントが在庫と売上のデータを取得するツールを呼び出し、その情報を使用してユーザーに役に立つ応答を提供できます。

1. コメント **Process function calls** を見つけ、次のコードを追加します。

    ```python
   # Process function calls
   for item in response.output:
       if item.type == "function_call":
           # Retrieve the matching function tool
           function_name = item.name
           kwargs = json.loads(item.arguments)
           required_function = functions_dict.get(function_name)

           # Invoke the function
           output = await required_function(**kwargs)

           # Append the output text
           input_list.append(
              FunctionCallOutput(
                 type="function_call_output",
                 call_id=item.call_id,
                 output=output.content[0].text,
              )
           )
    ```

    このコードは、エージェントの応答で関数呼び出しをリッスンし、対応するツール関数を呼び出して、エージェントに送り返される出力を準備します。

1. コメント **Send function call outputs back to the model and retrieve a response** を見つけ、次のコードを追加します。

    ```python
   # Send function call outputs back to the model and retrieve a response
   if input_list:
      response = openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
      )
   print(f"Agent response: {response.output_text}")
    ```

1. 完了したら、コード ファイルを保存します (*CTRL + S*)。

### アプリケーションの実行

1. 統合ターミナルで、次のコマンドを入力してアプリケーションを実行します。

    ```
   python client.py
    ```

1. メッセージが表示されたら、次のようなプロンプトを入力します。

    ```
   Show me the current inventory levels for all products.
    ```

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

    次のような出力が表示されます。

    ```
    MessageRole.AGENT:
    Agent response: Here are the current inventory levels for all items:

   - Moisturizer: 6
   - Shampoo: 8
   - Body Spray: 28
   [continued ...]

   Would you like recommendations for restocking or clearance? If so, I can check the weekly sales to advise accordingly.
    ```

    エージェントが MCP ツールを呼び出して在庫と売上のデータを取得し、その情報を使用してユーザーに役に立つ応答を提供できたことがわかります。

1. 必要に応じて会話を続けることができます。 スレッドは*ステートフル*であるため、会話履歴を保持します。つまり、エージェントには各応答の完全なコンテキストが保持されます。 

    次のようなプロンプトを入力してみてください。

    ```
   Are there any products that should be restocked?
    ```

    ```
   Which products would you recommend for clearance?
    ```

    ```
   What are the best sellers this week?
    ```

    完了したら、「`quit`」と入力します。

## まとめ

この演習では、モデル コンテキスト プロトコル (MCP) サーバー ツールを使用して外部データ ソースと API にアクセスできる AI エージェントを作成しました。 エージェントを、Microsoft Learn Docs によってホストされているリモート MCP サーバーと、自分で実装したカスタム MCP サーバーに接続しました。 これらのツールを統合することで、エージェントは最新の情報を取得し、ユーザー プロンプトに対して情報に基づいた応答を提供できました。 これで、MCP ツールがどのようにして AI エージェントの機能を大幅に強化し、外部サービスとデータを活用して幅広いタスクを実行できるようになるかがわかります。 上出来

## クリーンアップ

Foundry VS Code 拡張機能を調べ終えたら、不要な Azure コストが発生しないようにリソースをクリーンアップする必要があります。

### モデルの削除

1. VS Code で、**[Azure リソース]** ビューを更新します。

1. **[モデル]** サブセクションを展開します。

1. デプロイしたモデルを右クリックし、**[削除]** を選択します。

### リソース グループを削除します

1. [Azure Portal](https://portal.azure.com)を開きます。

1. AI Foundry リソースを含むリソース グループに移動します。

1. **[リソース グループの削除]** を選択し、削除を確認します。