---
lab:
  title: AI エージェントを開発する
  description: Azure AI エージェント サービスを利用して、組み込みツールを使用するエージェントを開発します。
---

# AI エージェントを開発する

この演習では、Azure AI エージェント サービスを利用して、データを分析してグラフを作成するシンプルなエージェントを作成します。 エージェントは、組み込みの "コード インタープリター" ツールを使用して、データの分析に必要なコードを動的に生成できます。**

> **ヒント**: この演習で使用するコードは、Microsoft Foundry SDK for Python に基づいています。 Microsoft .NET、JavaScript、Java 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、[Microsoft Foundry SDK クライアント ライブラリ](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)に関するページを参照してください。

この演習の所要時間は約 **30** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Foundry プロジェクトを作成する

まず、Foundry プロジェクトを作成しましょう。

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインする場合に開かれるヒントまたはクイック スタートのペインを閉じ、必要に応じて、左上にある **[Foundry]** ロゴを使用してホーム ページに移動します。次の図のようなページが表示されます (**[ヘルプ]** ペインが表示される場合は閉じます)。

    ![Foundry ポータルのスクリーンショット。](./Media/ai-foundry-home-new.png)

    > **重要**: このラボでは、**新しい** Foundry エクスペリエンスを使用しています。
1. 上部のバナーで **[構築の開始]** を選択して、新しい Microsoft Foundry エクスペリエンスをお試しください。
1. プロンプトが表示されたら、**新しい**プロジェクトを作成し、プロジェクトに有効な名前を入力します。
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

## エージェント クライアント アプリを作成する

これで、エージェントを使用するクライアント アプリを作成する準備ができました。 一部のコードは GitHub リポジトリに用意されています。

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
   cd ai-agents/Labfiles/02-build-ai-agent/Python
   ls -a -l
    ```

    指定されたファイルには、アプリケーション コード、構成設定、データが含まれます。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**[your_project_endpoint]** プレースホルダーをプロジェクトのエンドポイント (Foundry ポータルのプロジェクトの概要ページからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名 (*[gpt-4.1]*) に設定されていることを確認します。

1. プレースホルダーを置き換えたら、**Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### エージェント アプリのコードを作成する

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 コメントのインデント レベルをガイドとして利用します。

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    ```
   code agent.py
    ```

1. アプリケーション構成設定を取得し分析対象の *data.txt* からデータを読み込むための既存のコードを確認します。 ファイルの残りの部分には、データ分析エージェントの実装に必要なコードを追加するコメントが含まれています。
1. 「**Add references （参照の追加)**」コメントを見つけ、以下のコードを追加して、組み込みのコード インタープリター ツールを使用する Azure AI エージェントの構築に必要なクラスをインポートします。

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import PromptAgentDefinition, CodeInterpreterTool, CodeInterpreterToolAuto

    ```

1. **AI プロジェクトと OpenAI クライアントに接続する**のコメントを見つけ、次のコードを追加して、Azure AI プロジェクトに接続します。

    > **ヒント**: インデント レベルを正しく維持するように注意してください。

    ```python
   # Connect to the AI Project and OpenAI clients
   with (
       DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True) as credential,
        AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
        project_client.get_openai_client() as openai_client
   ):
    ```

    このコードでは、現在の Azure 資格情報を使用して Foundry プロジェクトに接続します。 最後の *with project_client* ステートメントを使用すると、クライアントのスコープを定義するコード ブロックが開始され、ブロック内のコードが完了したときにクリーンアップされます。

1. *with project_client* ブロック内で「**Upload the data file and create a CodeInterpreterTool （データファイルをアップロードして CodeInterpreterTool を作成する)**」コメントを見つけ、以下のコードを追加してデータ ファイルをプロジェクトにアップロードするとともに、その中のデータにアクセスできる CodeInterpreterTool を作成します。

    ```python
   # Upload the data file and create a CodeInterpreterTool
   file = openai_client.files.create(
       file=open(file_path, "rb"), purpose="assistants"
   )
   print(f"Uploaded {file.filename}")

   code_interpreter = CodeInterpreterTool(
       container=CodeInterpreterToolAuto(file_ids=[file.id])
   )

    ```
    
1. 「**Define an agent that uses the CodeInterpreterTool （CodeInterpreterTool を使用するエージェントを定義する)**」コメントを見つけ、以下のコードを追加して、データを分析し定義されたコード インタープリター ツールを使用する AI エージェントを定義します。

    ```python
   # Define an agent that uses the CodeInterpreterTool
   agent = project_client.agents.create_version(
       agent_name="data-agent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions="You are an AI agent that analyzes the data in the file that has been uploaded. Use Python to calculate statistical metrics as necessary.",
           tools=[code_interpreter],
       ),
   )
   print(f"Using agent: {agent.name}")
    ```

1. **チャット セッションの会話を作成する**のコメントを見つけ、次のコードを追加して、エージェントとのチャット セッションを実行するスレッドを開始します。

    ```python
   # Create a conversation for the chat session
   conversation = openai_client.conversations.create()
    ```
    
1. コードの次のセクションにはユーザーがプロンプトを入力するためのループが設定されており、ユーザーが「quit （終了）」と入力するとチャットが終了します。

1. 「**Send a prompt to the agen （エージェントにプロンプトを送信する)**」コメントを見つけ、以下のコードを追加して、プロンプトにユーザー メッセージ (および前の手順で読み込んだファイルのデータ) を追加して、エージェントでスレッドを実行します。

    ```python
   # Send a prompt to the agent
   openai_client.conversations.items.create(
       conversation_id=conversation.id,
       items=[{"type": "message", "role": "user", "content": user_prompt}],
   )

   response = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
       input="",
   )
    ```

1. **エラーの応答状態を確認する**のコメントを見つけ、次のコードを追加して、エラーがないかどうかを確認します。

    ```python
   # Check the response status for failures
   if response.status == "failed":
       print(f"Response failed: {response.error}")
    ```

1. 「**Show the latest response from the agent エージェントからの最新の応答を表示する)**」コメントを見つけて以下のコードを追加し、完了済みのスレッドからメッセージを取得してエージェントによって最後に送信されたメッセージを表示します。

    ```python
   # Show the latest response from the agent
   print(f"Agent: {response.output_text}")
    ```

1. ループの終了後にある「**Get the conversation history (会話履歴を取得する)**」を見つけて以下のコードを追加すると、会話スレッドからメッセージが出力されます。メッセージは時系列順を逆にして古いものから順に表示されます。

    ```python
    # Get the conversation history
    print("\nConversation Log:\n")
    items = openai_client.conversations.items.list(conversation_id=conversation.id)
    for item in items:
        if item.type == "message":
            print(f"item.content[0].type = {item.content[0].type}")
            role = item.role.upper()
            content = item.content[0].text
            print(f"{role}: {content}\n")
    ```

1. 「**Cleain up （クリーンアップする)**」コメントを見つけて以下のコードを追加し、不要になったらエージェントとスレッドを削除します。

    ```python
   # Clean up
   openai_client.conversations.delete(conversation_id=conversation.id)
   print("Conversation deleted")

   project_client.agents.delete(agent_name=agent.name, agent_version=agent.version)
   print("Agent deleted")
    ```

1. コードを確認します。コメントを見ることで以下のコード機能を把握できます。
    - どのように AI Foundry プロジェクトに接続しているか
    - どのようにデータ ファイルをアップロードし、かつそれにアクセスできるコード インタープリター ツールを作成しているか
    - どのようにしてコード インタープリター ツールを使用する新しいエージェントを作成し、必要に応じて統計分析に Python を使用するための明示的な指示を用意しているか。
    - 分析対象のデータとユーザーからのプロンプト メッセージを含むスレッドをどのように実行しているか。
    - エラーが発生した場合に実行の状態を確認します
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
    
    アプリケーションは認証済みの Azure セッションの資格情報を使用して実行され、必要なプロジェクトに接続してエージェントを作成し実行します。

1. メッセージが表示されたら、アプリが *data.txt* テキスト ファイルから読み込んだデータを確認します。 次に、以下のようなプロンプトを入力します。

    ```
   What's the category with the highest cost?
    ```

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

1. 応答を表示します。 次に、別のプロンプトを入力し、今度は視覚化を要求します。

    ```
   Create a text-based bar chart showing cost by category
    ```

1. 応答を表示します。 次に、別のプロンプトを入力し、今度は統計メトリックを要求します。

    ```
   What's the standard deviation of cost?
    ```

    応答を表示します。

1. 必要に応じて会話を続けることができます。 スレッドは*ステートフル*であるため、会話履歴を保持します。つまり、エージェントには各応答の完全なコンテキストが保持されます。 完了したら、「`quit`」と入力します。
1. スレッドから取得された会話メッセージを確認します。これには、コード インタープリター ツールを使用する際の手順を説明するためにエージェントによって生成されたメッセージが含まれる場合があります。

## まとめ

この演習では、Azure AI エージェント サービス SDK を使用して、AI エージェントを使用するクライアント アプリケーションを作成しました。 エージェントは、組み込みのコード インタープリター ツールを使用して、統計分析を実行する動的な Python コードを実行できます。

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
