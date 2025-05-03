---
lab:
  title: AI エージェントでカスタム関数を使用する
  description: 関数を使用してエージェントにカスタム機能を追加する方法について説明します。
---

# AI エージェントでカスタム関数を使用する

この演習では、タスクを完了するためのツールとして、カスタム関数を使用できるエージェントの作成について確認します。

技術的な問題の詳細を収集し、サポート チケットを生成できる簡単なテクニカル サポート エージェントを構築します。

この演習の所要時間は約 **30** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Azure AI Foundry プロジェクトを作成する

まず、Azure AI Foundry プロジェクトを作成します。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./Media/ai-foundry-home.png)

1. ホーム ページで、**[+ 作成]** を選択します。
1. **[プロジェクトの作成]** ウィザードで、プロジェクトの有効な名前を入力し、既存のハブが推奨される場合は、新しいハブを作成するオプションを選択します。 次に、ハブとプロジェクトをサポートするために自動的に作成される Azure リソースを確認します。
1. **[カスタマイズ]** を選択し、ハブに次の設定を指定します。
    - **ハブ名**: *ハブの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **場所**: 次のいずれかのリージョンを選択します\*:
        - eastus
        - eastus2
        - swedencentral
        - westus
        - westus3
    - **Azure AI サービスまたは Azure OpenAI への接続**: *新しい AI サービス リソースを作成します*
    - **Azure AI 検索への接続**:接続をスキップする

    > \*この記事の執筆時点で、これらのリージョンでは、エージェントに使用できる gpt-4o モデルがサポートされています。 モデルの可用性は、リージョンのクォータによって制限されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のプロジェクトを作成する必要が生じる可能性があります。

1. **[次へ]** を選択し、構成を確認します。 **[作成]** を選択し、プロセスが完了するまで待ちます。
1. プロジェクトが作成されたら、表示されているヒントをすべて閉じて、Azure AI Foundry ポータルのプロジェクト ページを確認します。これは次の画像のようになっているはずです。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./Media/ai-foundry-project.png)

## 生成 AI モデルを展開する

これで、エージェントをサポートする生成 AI 言語モデルをデプロイする準備ができました。

1. プロジェクトの左側のウィンドウの **[マイ アセット]** セクションで、**[モデル + エンドポイント]** ページを選択します。
1. **[モデル + エンドポイント]** ページの **[モデル デプロイ]** タブの **[+ モデルのデプロイ]** メニューで、**[基本モデルのデプロイ]** を選択します。
1. 一覧で **GPT-4o** モデルを検索してから、それを選択して確認します。
1. デプロイの詳細で **[カスタマイズ]** を選択して、以下の設定でモデルをデプロイします。
    - **デプロイ名**: モデル デプロイの有効な名前**
    - **デプロイの種類**: グローバル標準
    - **バージョンの自動更新**: 有効
    - **モデルのバージョン**: *利用可能な最新バージョンを選択します*
    - **接続されている AI リソース**: *使用している Azure OpenAI リソース接続を選択します*
    - **1 分あたりのトークンのレート制限 (1,000)**: 50,000 *(または 50,000 未満の場合はサブスクリプションで使用可能な最大値)*
    - **コンテンツ フィルター**: DefaultV2

    > **注**:TPM を減らすと、ご利用のサブスクリプション内で使用可能なクォータが過剰に消費されることを回避するのに役立ちます。 この演習で使用するデータには、50,000 TPM で十分です。 使用可能なクォータがこれより低い場合は、演習を完了できますが、レート制限を超えた場合は、少し待ってからプロンプトを再送信する必要がある場合があります。

1. デプロイが完了するまで待ちます。

## 関数ツールを使用するエージェントを開発する

AI Foundry でプロジェクトを作成したので、カスタム関数ツールを使用してエージェントを実装するアプリを開発します。

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

    > **ヒント**: Cloudshell にコマンドを入力すると、出力が大量のスクリーン バッファーを占有し、現在の行のカーソルが見づらくなる可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. 次のコマンドを入力して、作業ディレクトリをコード ファイルを含むフォルダーに変更し、すべてを一覧表示します。

    ```
   cd ai-agents/Labfiles/03-ai-agent-functions/Python
   ls -a -l
    ```

    指定されたファイルには、アプリケーション コードと構成設定用のファイルがあります。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects
    ```

    >**注:** ライブラリのインストール中に表示される警告またはエラー メッセージは無視できます。

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_connection_string** プレースホルダーをプロジェクトの接続文字列 (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを GPT-4o モデル デプロイに割り当てた名前に置き換えます。
1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### カスタム関数を定義する

1. 次のコマンドを入力して、関数コードに提供されたコード ファイルを編集します。

    ```
   code user_functions.py
    ```

1. 「**サポート チケットを送信する関数を作成する**」というコメントを見つけて、次のコードを追加します。このコードでは、チケット番号が生成され、サポート チケットがテキスト ファイルとして保存されます。

    ```python
   # Create a function to submit a support ticket
   def submit_support_ticket(email_address: str, description: str) -> str:
        script_dir = Path(__file__).parent  # Get the directory of the script
        ticket_number = str(uuid.uuid4()).replace('-', '')[:6]
        file_name = f"ticket-{ticket_number}.txt"
        file_path = script_dir / file_name
        text = f"Support ticket: {ticket_number}\nSubmitted by: {email_address}\nDescription:\n{description}"
        file_path.write_text(text)
    
        message_json = json.dumps({"message": f"Support ticket {ticket_number} submitted. The ticket file is saved as {file_name}"})
        return message_json
    ```

1. 「**呼び出し可能な関数のセットを定義する**」というコメントを見つけて、次のコードを追加します。このコード ファイルでは、呼び出し可能な関数のセットが静的に定義されます (この場合は 1 つだけですが、実際のソリューションでは、エージェントが呼び出すことができる複数の関数がある可能性があります)。

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. ファイルを保存します (*Ctrl キーを押しながら S キーを押します*)。

### 関数を使用できるエージェントを実装するコードを記述する

1. 次のコマンドを入力して、エージェント コードの編集を開始します。

    ```
    code agent.py
    ```

    > **ヒント**: コード ファイルにコードを追加する際は、必ず正しいインデントを維持してください。

1. アプリケーション構成設定を取得し、ユーザーがエージェントのプロンプトを入力できるように、ループを設定する既存のコードを確認します。 ファイルのその他の部分には、テクニカル サポート エージェントの実装に必要なコードを追加するコメントが記載されています。
1. 「**参照を追加する**」というコメントを見つけて、次のコードを追加して、関数コードをツールとして使用する Azure AI エージェントの構築に必要なクラスをインポートします。

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FunctionTool, ToolSet
   from user_functions import user_functions
    ```

1. 「**Azure AI Foundry プロジェクトに接続する**」というコメントを見つけて次のコードを追加し、現在の Azure 資格情報を使用して Azure AI プロジェクトに接続します。

    > **ヒント**: インデント レベルを正しく維持するように注意してください。

    ```python
   # Connect to the Azure AI Foundry project
   project_client = AIProjectClient.from_connection_string(
        credential=DefaultAzureCredential
            (exclude_environment_credential=True,
             exclude_managed_identity_credential=True),
        conn_str=PROJECT_CONNECTION_STRING
   )
    ```
    
1. 「**Define an agent that can use the custom functions (カスタム関数を使用できるエージェントを定義する)**」コメントを見つけ、以下のコードを追加して、関数コードをツールセットに追加すると共に、このツールセットとチャット セッションを実行するスレッドを使用できるエージェントを作成します。

    ```python
   # Define an agent that can use the custom functions
   with project_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
            
        agent = project_client.agents.create_agent(
            model=MODEL_DEPLOYMENT,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = project_client.agents.create_thread()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. 「**Send a prompt to the agent (エージェントにプロンプトを送信する)**」コメントを見つけ、以下のコードを追加して、ユーザーのプロンプトをメッセージとして追加し、スレッドを実行します。

    ```python
   # Send a prompt to the agent
   message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)
    ```

    > **注**: **create_and_process_run** メソッドを使用してスレッドを実行すると、エージェントは関数を自動で検索し、名前とパラメーターに基づいて必要な関数を選択して使用します。 別の方法として、**create_run** メソッドを使用することもできます。このメソッドの場合、実行状態をポーリングするコードを記述して、関数呼び出しがいつ必要かを判断し、関数を呼び出して結果をエージェントに返します。

1. 「**Check the run status for failures (実行状態を確認し、エラーが発生していないかチェックする)**」コメントを見つけ、以下のコードを追加して、発生したエラーがあれば表示します。

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. 「**Show the latest response from the agent (エージェントからの最新の応答を表示する)**」コメントを見つけ、以下のコードを追加して、完了したスレッドからメッセージを取得するとともに、エージェントによって最後に送信されたメッセージを表示します。

    ```python
   # Show the latest response from the agent
   messages = project_client.agents.list_messages(thread_id=thread.id)
   last_msg = messages.get_last_text_message_by_role("assistant")
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. 「**Get the conversation history (会話履歴を取得する)**」コメントを見つけ、以下のコードを追加して、会話スレッドからのメッセージを出力します （順番を並び替えて、古い物から逆に表示します)。

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = project_client.agents.list_messages(thread_id=thread.id)
   for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
    ```

1. 「**Clean up （クリーンアップする)**」コメントを見つけ、以下のコードを追加して、不要になったエージェントとスレッドを削除します。

    ```python
   # Clean up
   project_client.agents.delete_agent(agent.id)
   project_client.agents.delete_thread(thread.id)
    ```

1. コードを確認します。コメントを見ることで以下のコード機能を把握できます。
    - カスタム関数のセットをツールセットに追加する
    - ツールセットを使用するエージェントを作成します。
    - ユーザーからのプロンプト メッセージを含むスレッドを実行します。
    - エラーが発生した場合に実行の状態を確認します
    - 完了したスレッドからメッセージを取得し、エージェントによって最後に送信されたメッセージを表示します。
    - 会話履歴を表示する
    - 不要になったエージェントとスレッドは削除してください。

1. 完了したら、コード ファイル (*CTRL + S*) を保存します。 コード エディター (*CTRL + Q*) を閉じてもかまいませんが、追加したコードを編集する必要がある場合に備えて開いたままにしておくこともできます。 どちらの場合も、Cloud Shell のコマンド ライン ペインは開いたままにします。

### Azure にサインインしてアプリを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    ```
    az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*[az ログイン]* を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*--tenant* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。
    
1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、必要な Azure AI Foundry ハブを含むサブスクリプションを選択します。
1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```
   python agent.py
    ```
    
    アプリケーションは認証済みの Azure セッションの資格情報を使用して実行され、必要なプロジェクトに接続してエージェントを作成し実行します。

1. メッセージが表示されたら、次のようなプロンプトを入力します。

    ```
   I have a technical problem
    ```

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

1. 応答を表示します。 エージェントは、メール アドレスの提供と問題の説明を求める場合があります。 メール アドレス (たとえば、`alex@contoso.com`) と問題の説明 (`my computer won't start` など) は任意のものを自由に使用できます。

    十分な情報がある場合、エージェントは必要に応じて自動で関数の使用を選択します。

1. 必要に応じて会話を続けることができます。 スレッドは*ステートフル*であるため、会話履歴を保持します。つまり、エージェントには各応答の完全なコンテキストが保持されます。 完了したら、「`quit`」と入力します。
1. スレッドから取得した会話メッセージと、生成されたチケットを確認します。
1. サポート チケットはツールによりアプリ フォルダーに自動保存されているはずです。 `ls` コマンドを使用して確認し、次のように `cat` コマンドを使用してファイルの内容を表示できます。

    ```
   cat ticket-<ticket_num>.txt
    ```

## クリーンアップ

これで演習が完了したので、不要なリソース使用を避けるために、作成したクラウド リソースを削除してください。

1. [Azure portal](https://portal.azure.com) を `https://portal.azure.com` で開き、この演習で使用したハブ リソースをデプロイしたリソース グループの内容を確認します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。