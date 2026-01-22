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

    ![Foundry ポータルのスクリーンショット。](./Media/ai-foundry-home-new.png)

    > **重要**: このラボでは、**新しい** Foundry エクスペリエンスを使用しています。

1. 上部のバナーで **[構築の開始]** を選択して、新しい Microsoft Foundry エクスペリエンスをお試しください。

1. プロンプトが表示されたら、**[新しいプロジェクトの作成]** を選択し、プロジェクトに有効な名前を入力します。

1. **[詳細オプション]** を展開し、次の設定を指定します。
    - **Foundry リソース**: "Foundry リソースの有効な名前"**
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *お手持ちのリソース グループを選択するか、新しいリソース グループを作成します*
    - **リージョン**: ***AI Foundry が推奨するもの***\** の中から選択します

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。

1. プロジェクトが作成されたら、ナビゲーション バーから **[ビルド]** を選択します。

1. 左側のメニューから **[モデル]** を選択し、**[基本モデルのデプロイ]** を選択します。

1. 検索ボックスに「**gpt-4.1**」と入力し、検索結果から **[gpt-4.1]** モデルを選択します。

1. 既定の設定で **[デプロイ]** を選択して、モデルのデプロイを作成します。

    モデルがデプロイされると、モデルのプレイグラウンドが表示されます。

1. 左側のナビゲーション バーで **[Microsoft Foundry]** を選択して Foundry のホーム ページに戻ります。

1. **[プロジェクト エンドポイント]** の値をメモ帳にコピーし、これを使用して、クライアント アプリケーションのプロジェクトに接続します。

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
   pip install -r requirements.txt azure-ai-agents
    ```

    >**注:** ライブラリのインストール中に表示される警告やエラー メッセージは無視しても構いません。

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**[your_project_endpoint]** プレースホルダーをプロジェクトのエンドポイント (Foundry ポータルのプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名 (*[gpt-4.1]*) に設定されていることを確認します。

1. プレースホルダーを置き換えたら、**Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### エージェントが使用する関数を作成する

1. 次のコマンドを入力して、エージェント コードの編集を開始します。

    ```
    code agent.py
    ```

    > **ヒント**: コード ファイルにコードを追加する際は、必ず正しいインデントを維持してください。

1. アプリケーション構成設定を取得してユーザーがエージェント向けプロンプトを入力するためのループを設定する既存のコードを確認します。 ファイルの残りの部分には、テクニカル サポート エージェントの実装に必要なコードを追加するコメントが含まれています。

1. 「**Add references (参照を追加する)**」コメントを見つけて以下のコードを追加し、関数ツールを使用する Azure AI エージェントの構築に必要なクラスをインポートします。

    ```python
   # Add references
   import json
   import uuid
   from pathlib import Path
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import PromptAgentDefinition, FunctionTool
   from openai.types.responses.response_input_param import FunctionCallOutput, ResponseInputParam
    ```

1. 「**Create a function to submit a support ticket (サポート チケットを送信する関数を作成する)**」コメントを見つけて、以下のコードを追加します。

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

    このコードは、チケット番号を生成し、サポート チケットをテキスト ファイルとして保存し、チケットが送信されたことを示すメッセージを返す関数を定義します。

### Foundry プロジェクトに接続する

1. 「**Connect to the AI Project client (AI Project クライアントに接続する)**」コメントを見つけて以下のコードを追加し、Azure AI プロジェクトに接続します。

    > **ヒント**: インデント レベルを正しく維持するように注意してください。

    ```python
   # Connect to the AI Project client
   with (
       DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True) as credential,
       AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
       project_client.get_openai_client() as openai_client,
   ):
    ```

### 関数ツールを定義する

1. 「**Create a FunctionTool definition (FunctionTool 定義を作成する)**」コメントを見つけ、次のコードを追加して、カスタム関数を使用する関数ツールを定義します。

    ```python
   # Create a FunctionTool definition
   tool = FunctionTool(
       name="submit_support_ticket",
       parameters={
           "type": "object",
           "properties": {
               "email_address": {"type": "string", "description": "The user's email address"},
               "description": {"type": "string", "description": "A description of the technical issue"},
            },
            "required": ["email_address", "description"],
            "additionalProperties": False,
       },
       description="Submit a support ticket for a technical issue",
       strict=True,
   )
    ```

    **FunctionTool** オブジェクトは、JSON スキーマを使用して、関数が受け入れるパラメーターと、関数により何が実行されるかの説明を定義します。

### 関数ツールを使用するエージェントを作成する

1. 「**Define an agent that can use the custom functions (カスタム関数を使用できるエージェントを定義する)**」コメントを見つけ、以下のコードを追加して、定義した関数ツールを使用できるエージェントを作成します。

    ```python
   # Initialize the agent with the FunctionTool
   agent = project_client.agents.create_version(
       agent_name="support-agent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
           tools=[tool],
       ),
   )
   print(f"Using agent: {agent.name} (version: {agent.version})")
    ```

### メッセージをエージェントに送信し、応答を処理する

1. 「**Create a thread for the chat session (チャット セッションのスレッドを作成する)**」というコメントを見つけて、次のコードを追加します。

    ```python
   # Create a thread for the chat session
       conversation = openai_client.conversations.create()
       print(f"Created conversation (id: {conversation.id})")
    ```

1. 「**Send a prompt to the agent (エージェントにプロンプトを送信する)**」コメントを見つけ、以下のコードを追加してユーザーのプロンプトをメッセージとして追加します。

    ```python
   # Send a prompt to the agent
   openai_client.conversations.items.create(
       conversation_id=conversation.id,
       items=[{"type": "message", "role": "user", "content": user_prompt}],
   )
    ```

1. コメント「**Get the agent's response (エージェントの応答を取得する)**」を見つけて、次のコードを追加してエージェントの応答を取得します。

    ```python
   # Get the agent's response
   response = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
       input="",
   )
    ```

1. 「**Check the run status for failures （実行状態にエラーがないかチェックする)**」コメントを見つけて以下のコードを追加し、発生したエラーを表示します。

    ```python
   # Check the run status for failures
   if response.status == "failed":
       print(f"Response failed: {response.error}")
    ```

### 関数呼び出しを処理し、エージェントの応答を表示する

1. 「**Process function calls (関数呼び出しを処理する)**」コメントを見つけて、エージェントによって行われた関数呼び出しを処理する次のコードを追加します。

    ```python
   # Process function calls
   input_list: ResponseInputParam = []
   for item in response.output:
       if item.type == "function_call":
           if item.name == "submit_support_ticket":
               # Execute the function logic for submit_support_ticket
               result = submit_support_ticket(**json.loads(item.arguments))

               # Provide function call results to the model
               input_list.append(
                   FunctionCallOutput(
                       type="function_call_output",
                       call_id=item.call_id,
                       output=result,
                   )
               )
    ```

    このコードは、エージェントの応答で関数呼び出しをチェックし、対応する関数を実行して、エージェントに返される結果を準備します。

1. 「**If there are function call outputs, send them back to the model (関数呼び出しの出力がある場合は、それらをモデルに送り返します)**」というコメントを見つけて、以下を追加します。

    ```python
   # If there are function call outputs, send them back to the model
   if input_list:
       response = openai_client.responses.create(
           input=input_list,
           previous_response_id=response.id,
           extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
       )

   print(f"Agent response: {response.output_text}")
    ```

    このコードは、関数呼び出しの結果をエージェントに返し、エージェントの応答を出力します。


1. 「**Cleain up （クリーンアップする)**」コメントを見つけて以下のコードを追加し、不要になったらエージェントとスレッドを削除します。

    ```python
   # Clean up
   openai_client.conversations.delete(conversation_id=conversation.id)
   print("Conversation deleted")

   project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
   print("Agent deleted")
    ```

1. ファイルに追加した完全なコードを確認します。 次のセクションが含まれるはずです。
   - 必要なライブラリをインポートする
   - サポート チケットを送信する関数を定義する
   - Foundry プロジェクトと AI Project クライアントに接続する
   - カスタム関数を使用して関数ツールを定義する
   - 関数ツールを使用できるエージェントを作成する
   - チャット セッションの会話スレッドを作成する
   - エージェントにユーザー プロンプトを送信し、応答を取得する
   - エージェントによって行われた関数呼び出しを処理する
   - 関数呼び出しの結果をエージェントに送り返し、応答を表示する
   - 会話とエージェントを削除してリソースをクリーンアップする

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

1. 会話から取得されたメッセージと、生成されたチケットを確認します。

1. ツールにより、サポート チケットがアプリ フォルダーに保存されているはずです。 `ls` コマンドを使用してチケットが保存されているのをチェックしたら、次のように `cat` コマンドを使用してファイルの内容を表示できます。

    ```
   cat ticket-<ticket_num>.txt
    ```

## クリーンアップ

これで演習が完了したので、不要なリソース使用を避けるために、作成したクラウド リソースを削除してください。

1. `https://portal.azure.com` で [Azure ポータル](https://portal.azure.com) を開き、この演習で使用したハブ リソースをデプロイしたリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
