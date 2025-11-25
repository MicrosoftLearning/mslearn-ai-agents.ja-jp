---
lab:
  title: AI エージェントでカスタム関数を使用する
  description: 関数を使用してエージェントにカスタム機能を追加する方法について説明します。
---

# AI エージェントでカスタム関数を使用する

この演習では、タスクを完了するためのツールとしてカスタム関数を使用できるエージェントの作成について説明します。 技術的な問題の詳細を収集し、サポート チケットを生成するシンプルなテクニカル サポート エージェントを構築します。

> **ヒント**: この演習で使用するコードは、Microsoft Foundry SDK for Python に基づいています。 Microsoft .NET、JavaScript、Java 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、[Microsoft Foundry SDK クライアント ライブラリ](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)に関するページを参照してください。

この演習の所要時間は約 **30** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Foundry プロジェクトを作成する

まず、Foundry プロジェクトを作成しましょう。

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインする場合に開かれるヒントまたはクイック スタートのペインを閉じ、必要に応じて、左上にある **[Foundry]** ロゴを使用してホーム ページに移動します。次の図のようなページが表示されます (**[ヘルプ]** ペインが表示される場合は閉じます)。

    ![Foundry ポータルのスクリーンショット。](./Media/ai-foundry-home.png)

    > **重要**: このラボの場合、**[新しい Foundry]** トグルが "オフ" になっていることを確認します。**

1. ホーム ページで、**[エージェントを作成する]** を選択します。
1. プロジェクトの作成を求められたら、プロジェクトの有効な名前を入力し、**[詳細]** オプションを展開します。
1. プロジェクトについて次の設定を確認します。
    - **Foundry リソース**: "Foundry リソースの有効な名前"**
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **リージョン**: **AI Foundry が推奨する**もの*の中から選択します\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。
1. メッセージが表示されたら、(クォータの可用性に応じて) *[Global Standard]* または *[Standard]* デプロイ オプションを使用して、**gpt-4o** モデルをデプロイします。

    >**注**:クォータが使用可能な場合は、エージェントとプロジェクトの作成時に GPT-4o 基本モデルが自動的にデプロイされることがあります。

1. プロジェクトが作成されると、エージェント プレイグラウンドが開きます。

1. 左側のナビゲーション ウィンドウで **[概要]** を選択すると、プロジェクトのメイン ページが表示されます。次のようになります。

    ![Foundry プロジェクトの [概要] ページのスクリーンショット。](./Media/ai-foundry-project.png)

1. **[Azure AI Foundry プロジェクト エンドポイント]** の値をメモ帳にコピーします。後でこれらを使用して、クライアント アプリケーション内でプロジェクトに接続します。

## 関数ツールを使用するエージェントを開発する

AI Foundry でプロジェクトを作成できたので、カスタム関数ツールを使用してエージェントを実装するアプリを開発しましょう。

### アプリケーション コードを含むリポジトリを複製する

1. 新しいブラウザー タブを開きます (既存のタブでは Foundry ポータルを開いたままにしておきます)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。

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
   cd ai-agents/Labfiles/03-ai-agent-functions/Python
   ls -a -l
    ```

    指定されたファイルには、アプリケーション コードと構成設定用のファイルが含まれます。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents
    ```

    >**注:** ライブラリのインストール中に表示される警告やエラー メッセージは無視しても構いません。

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、プレースホルダー **your_project_endpoint** をプロジェクトのエンドポイント (Foundry ポータルのプロジェクトの **[概要]** ページからコピーした値) に置き換え、変数 MODEL_DEPLOYMENT_NAME がモデルのデプロイ名 (*gpt-4o*) に設定されていることを確認します。
1. プレースホルダーを置き換えたら、**Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### カスタム関数を定義する

1. 次のコマンドを入力して、使用する関数コード向けに提供されているコード ファイルを編集します。

    ```
   code user_functions.py
    ```

1. 「**Create a function to submit a support ticket (サポート チケットを送信する関数を作成する)**」コメントを見つけて以下のコードを追加し、チケット番号を生成してサポート チケットをテキスト ファイルとして保存します。

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

1. 「**Define a set of callable functions**」コメントを見つけて以下のコードを追加し、このコード ファイル内の呼び出し可能な関数セットを静的に定義します (この場合は関数は 1 つだけですが、実際のソリューションではエージェントが呼び出すことができる複数の関数が存在する可能性があります)。

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. ファイルを保存します (*CTRL+S*)。

### 関数を使用できるエージェントを実装するコードを記述する

1. 次のコマンドを入力して、エージェント コードの編集を開始します。

    ```
    code agent.py
    ```

    > **ヒント**: コード ファイルにコードを追加する際は、必ず正しいインデントを維持してください。

1. アプリケーション構成設定を取得してユーザーがエージェント向けプロンプトを入力するためのループを設定する既存のコードを確認します。 ファイルの残りの部分には、テクニカル サポート エージェントの実装に必要なコードを追加するコメントが含まれています。
1. 「**Add references (参照を追加する)**」コメントを見つけて以下のコードを追加し、関数コードをツールとして使用する Azure AI エージェントの構築に必要なクラスをインポートします。

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, ToolSet, ListSortOrder, MessageRole
   from user_functions import user_functions
    ```

1. 「**Connect to the Agent client (エージェント クライアントに接続する)**」コメントを見つけて以下のコードを追加し、現在サインインに使用している Azure 資格情報で Azure AI プロジェクトに接続します。

    > **ヒント**: コードのインデント レベルを正しく維持するように注意してください。

    ```python
   # Connect to the Agent client
   agent_client = AgentsClient(
       endpoint=project_endpoint,
       credential=DefaultAzureCredential
           (exclude_environment_credential=True,
            exclude_managed_identity_credential=True)
   )
    ```
    
1. 「**Define an agent that can use the custom functions (カスタム関数を使用できるエージェントを定義する)**」コメントを見つけて、以下のコードの追加により必要な関数コードをツールセットに追加した後、そのツールセットとチャットセッションを実行するスレッドを使用できるエージェントを作成します。

    ```python
   # Define an agent that can use the custom functions
   with agent_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
        agent_client.enable_auto_function_calls(toolset)
            
        agent = agent_client.create_agent(
            model=model_deployment,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = agent_client.threads.create()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. 「**Send a prompt to the agent (エージェントにプロンプトを送信する)**」コメントを見つけ、以下のコードの追加によりユーザー プロンプトをメッセージとして追加してスレッドを実行します。

    ```python
   # Send a prompt to the agent
   message = agent_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    ```

    > **注**: 「**create_and_process**」メソッドを使用してスレッドを実行すると、エージェントは必要な関数を自動的に検索し、名前とパラメーターに基づいて関数を選択的に使用します。 あるいは別の方法として、**create_run** メソッドを使用することもできます。このメソッドの場合、実行状態をポーリングするコードを記述して、関数呼び出しが必要かどうかを判断し、関数を呼び出して結果をエージェントに返します。

1. 「**Check the run status for failures （実行状態にエラーがないかチェックする)**」コメントを見つけて以下のコードを追加し、発生したエラーを表示します。

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. 「**Show the latest response from the agent エージェントからの最新の応答を表示する)**」コメントを見つけて以下のコードを追加し、完了済みのスレッドからメッセージを取得してエージェントによって最後に送信されたメッセージを表示します。

    ```python
   # Show the latest response from the agent
   last_msg = agent_client.messages.get_last_message_text_by_role(
       thread_id=thread.id,
       role=MessageRole.AGENT,
   )
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. 「**Get the conversation history (会話履歴を取得する)**」コメントを見つけて以下のコードを追加すると、会話スレッドからメッセージが出力されます。メッセージは時系列順に並べ替えて出力されます。

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = agent_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
           last_msg = message.text_messages[-1]
           print(f"{message.role}: {last_msg.text.value}\n")
    ```

1. 「**Cleain up （クリーンアップする)**」コメントを見つけて以下のコードを追加し、不要になったらエージェントとスレッドを削除します。

    ```python
   # Clean up
   agent_client.delete_agent(agent.id)
   print("Deleted agent")
    ```

1. コメントを使用してコードを確認し、以下の機能がどのように実行されるかを理解します。
    - カスタム関数のセットをツールセットに追加する
    - ツールセットを使用するエージェントを作成する
    - ユーザーからのプロンプト メッセージを含むスレッドを実行する
    - エラーが発生した場合に実行状態を確認する
    - 完了済みのスレッドからメッセージを取得し、エージェントによって最後に送信されたメッセージを表示する
    - 会話履歴を表示する
    - 不要になったエージェントとスレッドを削除する

1. 完了したら、コード ファイルを保存します (*CTRL + S*)。 コード エディターを閉じても構いません (*CTRL + Q*) が、追加したコードを編集する必要がある場合に備えて開いたままにしておくこともできます。 どちらの場合も、Cloud Shell のコマンド ライン ペインは開いたままにします。

### Azure にサインインしてアプリを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Azure にサインインします。

    ```
    az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。
    
1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、Foundry ハブを含むサブスクリプションを選択します。
1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```
   python agent.py
    ```
    
    アプリケーションは、認証済みの Azure セッションの資格情報を使用して実行され、プロジェクトに接続してエージェントを作成して実行します。

1. メッセージが表示されたら、次のようなプロンプトを入力します。

    ```
   I have a technical problem
    ```

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

1. 応答を表示します。 エージェントが、メール アドレスと問題の記述を求める場合があります。 メール アドレス (たとえば、`alex@contoso.com`) と問題の説明 (`my computer won't start` など) は任意のものを自由に使用できます。

    十分な情報がある場合、エージェントは必要に応じて関数の使用を自動で選択します。

1. 必要に応じて会話を続けることができます。 スレッドは*ステートフル*であるため、会話履歴を保持します。つまり、エージェントには各応答の完全なコンテキストが保持されます。 完了したら、「`quit`」と入力します。
1. スレッドから取得された会話メッセージと、生成されたチケットを確認します。
1. ツールにより、サポート チケットがアプリ フォルダーに保存されているはずです。 `ls` コマンドを使用してチケットが保存されているのをチェックしたら、次のように `cat` コマンドを使用してファイルの内容を表示できます。

    ```
   cat ticket-<ticket_num>.txt
    ```

## クリーンアップ

これで演習が完了したので、不要なリソース使用を避けるために、作成したクラウド リソースを削除してください。

1. `https://portal.azure.com` で [Azure ポータル](https://portal.azure.com) を開き、この演習で使用したハブ リソースをデプロイしたリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
