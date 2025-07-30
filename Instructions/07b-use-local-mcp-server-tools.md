# モデル コンテキスト プロトコル (MCP) を使用して AI エージェントをツールに接続する

この演習では、MCP サーバーに接続し、呼び出し可能な関数を自動的に検出できるエージェントを作成します。

化粧品小売業者向けの簡単な在庫評価エージェントを構築します。 MCP サーバーを使用すると、エージェントは在庫に関する情報を取得し、在庫の補充または一掃の提案を行うことができます。

> **ヒント**: この演習で使用するコードは、Python 用の Azure AI Foundry SDK と MCP SDK に基づいています。 Microsoft .NET 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、「[Azure AI Foundry SDK クライアント ライブラリ](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)」と [MCP C# SDK](https://modelcontextprotocol.github.io/csharp-sdk/api/ModelContextProtocol.html) を参照してください。

この演習の所要時間は約 **30** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Azure AI Foundry プロジェクトを作成する

まず、Azure AI Foundry プロジェクトを作成します。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./Media/ai-foundry-home.png)

1. ホーム ページで、**[エージェントを作成する]** を選択します。
1. プロジェクトの作成を求められたら、プロジェクトの有効な名前を入力し、**[詳細]** オプションを展開します。
1. プロジェクトについて次の設定を確認します。
    - **Azure AI Foundry リソース**: *Azure AI Foundry リソースの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **リージョン**: ***AI サービスでサポートされている場所を選択します***\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。
1. メッセージが表示されたら、(クォータの可用性に応じて) *[Global Standard]* または *[Standard]* デプロイ オプションを使用して、**gpt-4o** モデルをデプロイします。

    >**注**:クォータが使用可能な場合は、エージェントとプロジェクトの作成時に GPT-4o 基本モデルが自動的にデプロイされることがあります。

1. プロジェクトが作成されると、エージェント プレイグラウンドが開きます。

1. 左側のナビゲーション ウィンドウで **[概要]** を選択すると、プロジェクトのメイン ページが表示されます。次のようになります。

    ![Azure AI Foundry プロジェクトの概要ページのスクリーンショット。](./Media/ai-foundry-project.png)

1. **Azure AI Foundry プロジェクト エンドポイント**の値をメモ帳にコピーします。後でこれを使用して、クライアント アプリケーション内でプロジェクトに接続します。

## MCP 関数ツールを使用するエージェントを開発する

AI Foundry でプロジェクトを作成したので、AI エージェントと MCP サーバーを統合するアプリを開発しましょう。

### アプリケーション コードを含むリポジトリを複製する

1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。

    ウェルカム通知を閉じて、Azure portal のホーム ページを表示します。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、サブスクリプションにストレージがない ***PowerShell*** 環境を選択します。

    Azure portal の下部にあるペインに Cloud Shell のコマンド ライン インターフェイスが表示されます。 作業しやすくするために、このウィンドウのサイズを変更したり最大化したりすることができます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. Cloud Shell 画面で、次のコマンドを入力して、この演習のコード ファイルを含む GitHub リポジトリをクローンします (コマンドを入力するか、クリップボードにコピーしてから、コマンド ラインで右クリックし、プレーンテキストとして貼り付けます)。

    ```
   rm -r ai-agents -f
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents ai-agents
    ```

    > **ヒント**: Cloudshell にコマンドを入力すると、出力が大量のスクリーン バッファーを占有し、現在のライン上のカーソルが隠れてしまう可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. 次のコマンドを入力して、作業ディレクトリをコード ファイルを含むフォルダーに変更し、すべてを一覧表示します。

    ```
   cd ai-agents/Labfiles/07-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

    提供されるファイルには、クライアントとサーバーのアプリケーション コードが含まれます。 モデル コンテキスト プロトコルは、AI モデルをさまざまなデータ ソースやツールに接続するための標準化された方法を提供します。 `client.py` と `server.py` を分離して、エージェントロジックとツール定義をモジュール化し、実際のアーキテクチャをシミュレートします。 
    
    `server.py` は、エージェントが使用できるツールを定義し、バックエンド サービスまたはビジネス ロジックをシミュレートします。 
    `client.py` は、AI エージェントのセットアップ、ユーザー プロンプト、必要に応じたツールの呼び出しを処理します。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects mcp
    ```

    >**注:** ライブラリのインストール中に表示される警告やエラー メッセージは無視しても構いません。

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_endpoint** プレースホルダーをプロジェクトのエンドポイント (Azure AI Foundry ポータルのプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名 (*gpt-4o*) に設定されていることを確認します。

1. プレースホルダーを置き換えたら、**Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### MCP サーバーを実装する

モデル コンテキスト プロトコル (MCP) サーバーは、呼び出し可能なツールをホストするコンポーネントです。 これらのツールは、AI エージェントに公開可能な Python 関数です。 ツールに `@mcp.tool()` の注釈が付いている場合は、クライアントから検出可能になり、AI エージェントが会話やタスクの最中に動的に呼び出すことができます。 このタスクでは、エージェントが在庫チェックを実行できるようにするいくつかのツールを追加します。

1. 次のコマンドを入力して、使用する関数コード向けに提供されているコード ファイルを編集します。

    ```
   code server.py
    ```

    このコード ファイルでは、エージェントが小売店のバックエンド サービスをシミュレートするために使用できるツールを定義します。 ファイル上部のサーバー セットアップ コードに注目してください。 "Inventory" という名前の MCP サーバー インスタンスがすぐに起動するように `FastMCP` を使用しています。 このサーバーが、定義したツールをホストし、ラボ中にエージェントがアクセスできるようにします。

1. コメント **Add an inventory check tool** を見つけて、次のコードを追加します。

    ```python
   # Add an inventory check tool
   @mcp.tool()
   def get_inventory_levels() -> dict:
        """Returns current inventory for all products."""
        return {
            "Moisturizer": 6,
            "Shampoo": 8,
            "Body Spray": 28,
            "Hair Gel": 5, 
            "Lip Balm": 12,
            "Skin Serum": 9,
            "Cleanser": 30,
            "Conditioner": 3,
            "Setting Powder": 17,
            "Dry Shampoo": 45
        }
    ```

    このディクショナリは、サンプル インベントリを表します。 `@mcp.tool()` 注釈を使用すると、LLM で関数を検出できます。 

1. コメント **Add a weekly sales tool** を見つけて、次のコードを追加します。

    ```python
   # Add a weekly sales tool
   @mcp.tool()
   def get_weekly_sales() -> dict:
        """Returns number of units sold last week."""
        return {
            "Moisturizer": 22,
            "Shampoo": 18,
            "Body Spray": 3,
            "Hair Gel": 2,
            "Lip Balm": 14,
            "Skin Serum": 19,
            "Cleanser": 4,
            "Conditioner": 1,
            "Setting Powder": 13,
            "Dry Shampoo": 17
        }
    ```

1. ファイルを保存します (*CTRL+S*)。

### MCP クライアントを実装する

MCP クライアントは、MCP サーバーに接続してツールの検出と呼び出しを行うコンポーネントです。 エージェントとサーバーでホストされる関数の橋渡し役のように考えてください。ユーザー プロンプトへの応答で動的ツールを使用できるようになります。

1. 次のコマンドを入力して、クライアント コードの編集を開始します。

    ```
   code client.py
    ```

    > **ヒント**: コード ファイルにコードを追加する際は、必ず正しいインデントを維持してください。

1. コメント **Add references** を見つけて、クラスをインポートする次のコードを追加します。

    ```python
   # Add references
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, MessageRole, ListSortOrder
   from azure.identity import DefaultAzureCredential
    ```

1. コメント **Start the MCP server** を見つけて、次のコードを追加します。

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

このタスクでは、AI エージェントを準備し、ユーザー プロンプトを受け入れ、関数ツールを呼び出します。

1. コメント **Connect to the agents client** を見つけて、現在の Azure 資格情報を使用して Azure AI プロジェクトに接続する次のコードを追加します。

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True
        )
   )
    ```

1. コメント **List tools available on the server** の下に、次のコードを追加します。

    ```python
   # List tools available on the server
   response = await session.list_tools()
   tools = response.tools
    ```

1. コメント **Build a function for each tool** の下に、次のコードを追加します。

    ```python
   # Build a function for each tool
   def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            result = await session.call_tool(tool_name, kwargs)
            return result
        
        tool_func.__name__ = tool_name
        return tool_func

   functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
   mcp_function_tool = FunctionTool(functions=list(functions_dict.values()))
    ```

    このコードは、AI エージェントから呼び出すことができるように、MCP サーバーで使用可能なツールを動的にラップします。 各ツールは非同期関数に変換され、エージェントで使用できるように `FunctionTool` にバンドルされます。

1. コメント **Create the agent** を見つけて、次のコードを追加します。

    ```python
   # Create the agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="inventory-agent",
        instructions="""
        You are an inventory assistant. Here are some general guidelines:
        - Recommend restock if item inventory < 10  and weekly sales > 15
        - Recommend clearance if item inventory > 20 and weekly sales < 5
        """,
        tools=mcp_function_tool.definitions
   )
    ```

1. コメント **Enable auto function calling** を見つけて、次のコードを追加します。

    ```python
   # Enable auto function calling
   agents_client.enable_auto_function_calls(tools=mcp_function_tool)
    ```

1. コメント **Create a thread for the chat session** の下に、次のコードを追加します。

    ```python
   # Create a thread for the chat session
   thread = agents_client.threads.create()
    ```

1. コメント **Invoke the prompt** を見つけて、次のコードを追加します。

    ```python
   # Invoke the prompt
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=user_input,
   )
   run = agents_client.runs.create(thread_id=thread.id, agent_id=agent.id)
    ```

1. コメント **Retrieve the matching function tool** を見つけて、次のコードを追加します。

    ```python
   # Retrieve the matching function tool
   function_name = tool_call.function.name
   args_json = tool_call.function.arguments
   kwargs = json.loads(args_json)
   required_function = functions_dict.get(function_name)

   # Invoke the function
   output = await required_function(**kwargs)
    ```

    このコードでは、エージェント スレッドのツール呼び出しからの情報を使用します。 関数名と引数が取得され、一致する関数を呼び出すために使用されます。

1. コメント **Append the output text** の下に、次のコードを追加します。

    ```python
   # Append the output text
   tool_outputs.append({
        "tool_call_id": tool_call.id,
        "output": output.content[0].text,
   })
    ```

1. コメント **Submit the tool call output** の下に、次のコードを追加します。

    ```python
   # Submit the tool call output
   agents_client.runs.submit_tool_outputs(thread_id=thread.id, run_id=run.id, tool_outputs=tool_outputs)
    ```

    このコードは、必要なアクションが完了したことをエージェント スレッドに通知し、ツール呼び出しの出力を更新します。

1. コメント **Display the response** を見つけて、次のコードを追加します。

    ```python
   # Display the response
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
    ```

1. 完了したら、コード ファイルを保存します (*CTRL + S*)。 コード エディターを閉じても構いません (*CTRL + Q*) が、追加したコードを編集する必要がある場合に備えて開いたままにしておくこともできます。 どちらの場合も、Cloud Shell のコマンド ライン ペインは開いたままにします。

### Azure にサインインしてアプリを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Azure にサインインします。

    ```
   az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。
    
1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、Azure AI Foundry ハブを含むサブスクリプションを選択します。

1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```
   python client.py
    ```

1. メッセージが表示されたら、次のようなクエリを入力します。

    ```
   What are the current inventory levels?
    ```

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

    次のような出力が表示されます。

    ```
    MessageRole.AGENT:
    Here are the current inventory levels:

    - Moisturizer: 6
    - Shampoo: 8
    - Body Spray: 28
    - Hair Gel: 5
    - Lip Balm: 12
    - Skin Serum: 9
    - Cleanser: 30
    - Conditioner: 3
    - Setting Powder: 17
    - Dry Shampoo: 45
    ```

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

## クリーンアップ

これで演習が完了したので、不要なリソース使用を避けるために、作成したクラウド リソースを削除してください。

1. `https://portal.azure.com` で [Azure ポータル](https://portal.azure.com) を開き、この演習で使用したハブ リソースをデプロイしたリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
