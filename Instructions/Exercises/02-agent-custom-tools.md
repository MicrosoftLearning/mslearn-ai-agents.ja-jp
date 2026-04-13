---
lab:
  title: AI エージェントでカスタム関数を使用する
  description: 関数を使用してエージェントにカスタム機能を追加する方法について説明します。
  level: 300
  duration: 50
  islab: true
---

# AI エージェントでカスタム関数を使用する

この演習では、タスクを完了するためのツールとしてカスタム関数を使用できるエージェントの作成について説明します。 このエージェントは天文学アシスタントとなり、天文現象についての情報を提供することと、ユーザーの入力に基づいて望遠鏡レンタル料を計算することができます。 あなたは関数ツールを定義し、エージェントからの関数呼び出しを処理するロジックを実装します。

> **ヒント**: この演習で使用するコードは、Microsoft Foundry SDK for Python に基づいています。 Microsoft .NET、JavaScript、Java 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、[Microsoft Foundry SDK クライアント ライブラリ](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)に関するページを参照してください。

この演習の所要時間は約 **50** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

この演習を開始するには、以下のものが必要です。

- ローカル コンピューターに [Visual Studio Code](https://code.visualstudio.com/) がインストールされている
- 有効な [Azure サブスクリプション](https://azure.microsoft.com/free/)
- [Python 3.13](https://www.python.org/downloads/) 以降がインストールされている
- ローカル コンピューターに [Git](https://git-scm.com/downloads) がインストールされている

## Microsoft Foundry VS Code 拡張機能をインストールする

VS Code 拡張機能をインストールして設定することから始めましょう。

1. Visual Studio Code を開きます。

1. 左側のペインから **[拡張機能]** を選択します (または **Ctrl + Shift + X** キーを押します)。

1. 検索バーに「**Microsoft Foundry**」と入力し、Enter キーを押します。

1. Microsoft から **[Microsoft Foundry]** 拡張機能を選択し、**[インストール]** をクリックします。

1. インストールが完了したら、Visual Studio Code の左側のプライマリ ナビゲーション バーに拡張機能が表示されていることを確認します。

## Azure にサインインしてプロジェクトを作成する

次に、Azure リソースに接続し、新しい Microsoft Foundry プロジェクトを作成します。

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

1. モデル カタログで、**[gpt-4.1]** モデルを見つけます (検索バーを使用するとすばやく見つけることができます)。

    ![Foundry VS Code 拡張機能の [モデル カタログ] のスクリーンショット。](../Media/vs-code-model.png)

1. gpt-4.1 モデルの横にある **[デプロイ]** を選択します。

1. デプロイ設定を構成します。
   - **デプロイ名**:"gpt-4.1" のような名前を入力します
   - **デプロイの種類**:**[グローバル標準]** (グローバル標準を使用できない場合は **[標準]**) を選択します
   - **モデルのバージョン**: デフォルトのままにする
   - **1 分あたりのトークン数**:デフォルトのままにする

1. 左下隅にある **[Microsoft Foundry にデプロイ]** を選択します。

1. 確認ダイアログで **[デプロイ]** を選択して、モデルをデプロイします。

1. デプロイが完了するまで待ちます。 デプロイしたモデルが、[リソース] ビューの **[モデル]** セクションに表示されます。

1. プロジェクトのデプロイ名を右クリックし、**[プロジェクト エンドポイントのコピー]** を選択します。 次のステップでエージェントを Foundry プロジェクトに接続するために、この URL が必要になります。

   <img src="../Media/vs-code-endpoint.png" alt="Screenshot of copying the project endpoint in the Foundry VS Code extension." width="550">

## スタート コード リポジトリをクローンする

この演習では、スタート コードを利用して Foundry プロジェクトに接続し、カスタム関数ツールを使用するエージェントを作成します。

1. VS Code の **[ようこそ]** タブに移動します (メニュー バーから **[ヘルプ] > [ようこそ]** を選択すると開きます)。

1. **[Git リポジトリの複製]** を選択し、スタート コード リポジトリの URL `https://github.com/MicrosoftLearning/mslearn-ai-agents.git` を入力します。

1. 新しいフォルダーを作成し、**[リポジトリの宛先として選択]** を選択し、プロンプトが表示されたらクローンされたリポジトリを開きます。

1. エクスプローラー ビューで **Labfiles/02-agent-custom-tools/Python** フォルダーに移動します。ここに、この演習用のスタート コードがあります。

1. **requirements.txt** ファイルを右クリックし、**[統合ターミナルで開く]** を選択します。

1. ターミナルで、次のコマンドを入力して、仮想環境に必要な Python パッケージをインストールします。

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

1. **.env** ファイルを開き、**[your_project_endpoint]** プレースホルダーをプロジェクトのエンドポイント (Microsoft Foundry 拡張機能のプロジェクト デプロイ リソースからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名に設定されていることを確認します。 これらの変更を行った後、**Ctrl + S** を使用してファイルを保存します。

これで、MCP サーバー ツールを使用して外部データ ソースと API にアクセスする AI エージェントを作成する準備ができました。

## エージェントが使用する関数を作成する

1. **functions.py** ファイルを開いて既存のコードを確認します。

    このファイルには、エージェント用のツールとして使用できるさまざまな関数が含まれています。 これらの関数は、**data** フォルダーにあるサンプル ファイルを使用して天文現象と場所に関する情報を取得します。

1. **Determine the next visible astronomical event for a given location** というコメントを見つけて、次に示すコードを追加します。

    ```python
   # Determine the next visible astronomical event for a given location
   def next_visible_event(location: str) -> str:
       """Returns the next visible astronomical event for a location."""
       today = int(datetime.now().strftime("%m%d"))
       loc = location.lower().replace(" ", "_")

       # Retrieve the next event visible from the location, starting with events later this year
       for name, event_type, date, date_str, locs in EVENTS:
           if loc in locs and date >= today:
               return json.dumps({"event": name, "type": event_type, "date": date_str, "visible_from": sorted(locs)})

       return json.dumps({"message": f"No upcoming events found for {location}."})
    ```

    この関数はサンプル現象データを調べて、指定された場所から見ることができる次の天文現象を見つけて、その現象の詳細を JSON 文字列として返します。 次に、この関数を使用できるエージェントを作成しましょう。

## Foundry プロジェクトに接続する

1. **agent.py** ファイルを開きます。

   > **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 コメントのインデント レベルをガイドとして利用します。

1. 「**Add references (参照を追加する)**」コメントを見つけて以下のコードを追加し、関数ツールを使用する Azure AI エージェントの構築に必要なクラスをインポートします。

    ```python
   # Add references
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FunctionTool
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects.models import PromptAgentDefinition, FunctionTool
   from openai.types.responses.response_input_param import FunctionCallOutput, ResponseInputParam
   from functions import next_visible_event, calculate_observation_cost, generate_observation_report
    ```

    **functions.py** ファイルの中で定義した関数がインポートされているため、エージェント用のツールとして使用できることに注目してください。

1. **Connect to the project client** というコメントを見つけて、次に示すコードを追加します。

    ```python
    # Connect to the project client
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
        project_client.get_openai_client() as openai_client,
    ):
    ```

## 関数ツールを定義する

このタスクでは、エージェントでの使用が可能な関数ツールのそれぞれを定義します。 各関数ツールのパラメーターは JSON スキーマを使用して定義され、この中で関数の各パラメーターの名前、型、説明、その他の属性が指定されます。

1. **Define the event function tool** というコメントを見つけて、次に示すコードを追加します。

    ```python
   # Define the event function tool
   event_tool = FunctionTool(
       name="next_visible_event",
       description="Get the next visible event in a given location.",
       parameters={
           "type": "object",
           "properties": {
               "location": {
                   "type": "string",
                   "description": "continent to find the next visible event in (e.g. 'north_america', 'south_america', 'australia')",
               },
           },
           "required": ["location"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

1. **Define the observation cost function tool** というコメントを見つけて、次に示すコードを追加します。

    ```python
   # Define the observation cost function tool
   cost_tool = FunctionTool(
       name="calculate_observation_cost",
       description="Calculate the cost of an observation based on the telescope tier, number of hours, and priority level.",
       parameters={
           "type": "object",
           "properties": {
               "telescope_tier": {
                   "type": "string",
                   "description": "the tier of the telescope (e.g. 'standard', 'advanced', 'premium')",
               },
               "hours": {
                   "type": "number",
                   "description": "the number of hours for the observation",
               },
               "priority": {
                   "type": "string",
                   "description": "the priority level of the observation (e.g. 'low', 'normal', 'high')",
               },
           },
           "required": ["telescope_tier", "hours", "priority"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

1. **Define the observation report generation function tool** というコメントを見つけて、次に示すコードを追加します。

    ```python
   # Define the observation report generation function tool
   report_tool = FunctionTool(
       name="generate_observation_report",
       description="Generate a report summarizing an astronomical observation",
       parameters={
           "type": "object",
           "properties": {
               "event_name": {
                   "type": "string",
                   "description": "the name of the astronomical event being observed",
               },
               "location": {
                   "type": "string",
                   "description": "the location of the observer",
               },
               "telescope_tier": {
                   "type": "string",
                   "description": "the tier of the telescope used for the observation (e.g. 'standard', 'advanced', 'premium')",
               },
               "hours": {
                   "type": "number",
                   "description": "the number of hours the telescope was used for the observation",
               },
               "priority": {
                   "type": "string",
                   "description": "the priority level of the observation (e.g. 'low', 'normal', 'high')",
               },
               "observer_name": {
                   "type": "string",
                   "description": "the name of the person who conducted the observation",
               },                   
           },
           "required": ["event_name", "location", "telescope_tier", "hours", "priority", "observer_name"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

## 関数ツールを使用するエージェントを作成する

関数ツールの定義が完了したので、タスクを完了するためにこれらのツールを使用できるエージェントを作成できます。

1. **Create a new agent with the function tools** というコメントを見つけて、次に示すコードを追加します。

    ```python
   # Create a new agent with the function tools
   agent = project_client.agents.create_version(
       agent_name="astronomy-agent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions=
               """You are an astronomy observations assistant that helps users find 
               information about astronomical events and calculate telescope rental costs. 
               Use the available tools to assist users with their inquiries.""",
           tools=[event_tool, cost_tool, report_tool],
       ),
   )
    ```

## メッセージをエージェントに送信し、応答を処理する

関数ツールを使用するエージェントの作成が完了したので、エージェントにメッセージを送信してその応答を処理することができます。

1. 「**Create a thread for the chat session (チャット セッションのスレッドを作成する)**」というコメントを見つけて、次のコードを追加します。

    ```python
   # Create a thread for the chat session
   conversation = openai_client.conversations.create()
    ```

    このコードで、エージェントとのチャット セッションを作成します。

1. **Create a list to hold function call outputs that will be sent back as input to the agent (エージェントへの入力として返される関数呼び出し出力を保持するリストを作成する)** というコメントを検索して、次のコードを追加します。

    ```python
   # Create a list to hold function call outputs that will be sent back as input to the agent
   input_list: ResponseInputParam = []
   ```

1. **Send a prompt to the agent** というコメントを見つけて、次に示すコードを追加します。

    ```python
   # Send a prompt to the agent
   openai_client.conversations.items.create(
       conversation_id=conversation.id,
       items=[{"type": "message", "role": "user", "content": user_input}],
   )
    ```

1. **Retrieve the agent's response, which may include function calls** というコメントを見つけて、次に示すコードを追加します。

    ```python
   # Retrieve the agent's response, which may include function calls
   response = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
       input=input_list,
   )

   # Check the run status for failures
   if response.status == "failed":
       print(f"Response failed: {response.error}")
    ```

    このコードで、ユーザー プロンプトをエージェントに送信して応答を取得します。 また、応答が失敗を示しているかどうかを調べて、示している場合はエラーを出力します。

## 関数呼び出しを処理し、エージェントの応答を表示する

1. 「**Process function calls (関数呼び出しを処理する)**」コメントを見つけて、エージェントによって行われた関数呼び出しを処理する次のコードを追加します。

    ```python
   # Process function calls
   for item in response.output:
       if item.type == "function_call":
           # Retrieve the matching function tool
           function_name = item.name
           result = None
           if item.name == "next_visible_event":
               result = next_visible_event(**json.loads(item.arguments))
           elif item.name == "calculate_observation_cost":
               result = calculate_observation_cost(**json.loads(item.arguments))
           elif item.name == "generate_observation_report":
               result = generate_observation_report(**json.loads(item.arguments))
                
           # Append the output text
           input_list.append(
               FunctionCallOutput(
                   type="function_call_output",
                   call_id=item.call_id,
                   output=result,
               )
           )
    ```

    このコードでは、反復処理でエージェントの応答内の項目を順に調べて関数呼び出しを見つけます。 関数呼び出しが見つかった場合は、対応する関数ツールを取得し、指定された引数を付けてその関数を実行し、結果を入力リストの末尾に追加します。このリストがエージェントに送り返されることになります。

1. コメント **Send function call outputs back to the model and retrieve a response** を見つけ、次のコードを追加します。

    ```python
   # Send function call outputs back to the model and retrieve a response
   if input_list:
       response = openai_client.responses.create(
           input=input_list,
           previous_response_id=response.id,
           extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
       )
   # Display the agent's response
   print(f"AGENT: {response.output_text}")
    ```

    このコードでは、関数呼び出し出力が入力リストの中にあるかどうかを調べて、ある場合はその出力を入力としてエージェントに送り返し、更新された応答を取得します。 最後に、エージェントの応答を出力します。

1. **Delete the agent when done (終了したらエージェントを削除する)** というコメントを検索して、次のコードを追加します。

    ```python
   # Delete the agent when done
    project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
    print("Deleted agent.")
    ```

1. ファイルに追加した完全なコードを確認します。 次のセクションが含まれるはずです。
   - 必要なライブラリをインポートする
    - Foundry プロジェクトと OpenAI クライアントに接続する
    - エージェントに使用させる関数ツールを定義する
    - これらの関数ツールを使用するエージェントを作成する
    - エージェントにメッセージを送信して応答を取得する
    - エージェントによって関数呼び出しが行われた場合は処理して出力をエージェントに送り返す
    - エージェントの応答を表示する
    - 完了したらエージェントを削除する

1. 完了したら、コード ファイルを保存します (*CTRL + S*)。

## エージェント アプリケーションを実行する

1. 統合ターミナルで、次のコマンドを入力してアプリケーションを実行します。

    ```
   python agent.py
    ```

1. メッセージが表示されたら、次のようなプロンプトを入力します。

    ```
   Find me the next event I can see from South America and give me the cost for 5 hours of premium telescope time at normal priority.
    ```

    このプロンプトではエージェントに、あなたが定義した関数ツール、つまり `next_visible_event` と `calculate_observation_cost` の両方を使用するように依頼していることに注目してください。 エージェントは、この両方の関数を同じ会話ターンで呼び出すことができ、これらの関数呼び出しからの出力を使用して有益な応答をユーザーに返します。

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

    次のような出力が表示されます。

    ```output
    AGENT: The next astronomical event you can observe from South America is the Jupiter-Venus Conjunction, taking place on May 1st.
    The cost for 5 hours of premium telescope time at normal priority for this observation will be $1,875. 
    ```

1. 観察レポートを生成するためのフォローアップ プロンプトを、たとえば次のように入力します。

    ```
    Generate that information in a report for Bellows College.
    ```

    次のような応答が表示されます。

    ```output
    AGENT: Here is your report for Bellows College:

    - Next visible astronomical event: Jupiter-Venus Conjunction
    - Date: May 1st
    - Visible from: South America
    - Observation details:
        - Telescope tier: Premium
        - Duration: 5 hours
        - Priority: Normal
    - Observation cost: $1,875

    A formal report has been generated for Bellows College.
    ```

    エクスプローラーを見ると、`report-<event-type>.txt` という名前の新しいファイルが作成されており、この中に生成されたレポートがあります。 このファイルを開くと、レポートの内容を見ることができます。

1. 「`quit`」と入力してアプリケーションを終了します。

    `deactivate` を使用して、ターミナルの Python 仮想環境を終了することもできます。

## まとめ

この演習では、カスタム関数ツールを使用する AI エージェントを作成し、ユーザー プロンプトに基づいて情報の取得と計算の実行ができるようにしました。 関数ツールを定義してそのパラメーターを JSON スキーマで定義し、エージェントによって行われた関数呼び出しを処理するロジックを実装しました。 その後でアプリケーションを実行し、エージェントと対話しながら、エージェントがどのように関数ツールを使用して有益な応答を返すかを確認しました。 上出来

## クリーンアップ

Foundry VS Code 拡張機能を調べ終えたら、不要な Azure コストが発生しないようにリソースをクリーンアップする必要があります。

### モデルの削除

1. VS Code で、**[Azure リソース]** ビューを更新します。

1. **[モデル]** サブセクションを展開します。

1. デプロイしたモデルを右クリックし、**[削除]** を選択します。

### リソース グループを削除します

1. [Azure Portal](https://portal.azure.com)を開きます。

1. Microsoft Foundry リソースを含むリソース グループに移動します。

1. **[リソース グループの削除]** を選択し、削除を確認します。
