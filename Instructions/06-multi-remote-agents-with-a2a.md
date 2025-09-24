---
lab:
  title: A2A プロトコルを使用してリモート エージェントに接続する
  description: A2A プロトコルを使用してリモート エージェントと共同作業を行います。
---

# A2A プロトコルを使用してリモート エージェントに接続する

この演習では、Azure AI エージェント サービスと A2A プロトコルを使用して、相互に対話するシンプルなリモート エージェントを作成します。 これらのエージェントには、開発者ブログ記事を準備するテクニカル ライターを支援する機能があります。 タイトル エージェントを使用て見出しを生成し、アウトライン エージェントでタイトルを使用して記事の簡潔なアウトラインを作成します。 作業を開始する

> **ヒント**: この演習で使用するコードは、Python 用の Azure AI Foundry SDK に基づいています。 Microsoft .NET、JavaScript、Java 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、「[Azure AI Foundry SDK クライアント ライブラリ](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)」を参照してください。

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
    - **リージョン**: **AI Foundry が推奨する**もの*の中から選択します\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。
1. メッセージが表示されたら、(クォータの可用性に応じて) *[Global Standard]* または *[Standard]* デプロイ オプションを使用して、**gpt-4o** モデルをデプロイします。

    >**注**:クォータが使用可能な場合は、エージェントとプロジェクトの作成時に GPT-4o 基本モデルが自動的にデプロイされることがあります。

1. プロジェクトが作成されると、エージェント プレイグラウンドが開きます。

1. 左側のナビゲーション ウィンドウで **[概要]** を選択すると、プロジェクトのメイン ページが表示されます。次のようになります。

    ![Azure AI Foundry プロジェクトの概要ページのスクリーンショット。](./Media/ai-foundry-project.png)

1. **Azure AI Foundry プロジェクト エンドポイント**の値をメモ帳にコピーします。後でこれを使用して、クライアント アプリケーション内でプロジェクトに接続します。

## A2A アプリケーションを作成する

これで、エージェントを使用するクライアント アプリを作成する準備ができました。 一部のコードは GitHub リポジトリに用意されています。

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
   cd ai-agents/Labfiles/06-build-remote-agents-with-a2a/python
   ls -a -l
    ```

    提供されるファイルは次のとおりです。
    ```output
    python
    ├── outline_agent/
    │   ├── agent.py
    │   ├── agent_executor.py
    │   └── server.py
    ├── routing_agent/
    │   ├── agent.py
    │   └── server.py
    ├── title_agent/
    │   ├── agent.py
    |   ├── agent_executor.py
    │   └── server.py
    ├── client.py
    └── run_all.py
    ```

    各エージェント フォルダーには、Azure AI エージェント コードと、エージェントをホストするサーバーが格納されています。 **ルーティング エージェント**は、**タイトル** エージェントと**アウトライン** エージェントの検出と通信を担当します。 ユーザーは**クライアント**を使用して、ルーティング エージェントにプロンプトを送信できます。 `run_all.py` を使用すると、すべてのサーバーを起動し、クライアントを実行することができます。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects a2a-sdk
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_endpoint** プレースホルダーをプロジェクトのエンドポイント (Azure AI Foundry ポータルのプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名 (*gpt-4o*) に設定されていることを確認します。
1. プレースホルダーを置き換えたら、**Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### 検出可能なエージェントを作成する

このタスクでは、ライターが記事の傾向に沿ったヘッドラインを作成できるように支援するタイトル エージェントを作成します。 また、エージェントを検出可能にするために、A2A プロトコルで必要なエージェントのスキルとカードも定義します。

1. `title_agent` ディレクトリに移動します。

    ```
   cd title_agent
    ```

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 コメントのインデント レベルをガイドとして利用します。

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    ```
   code agent.py
    ```

1. コメント "**Create the agents client**" を見つけて、Azure AI プロジェクトに接続する次のコードを追加します。

    > **ヒント**: コードのインデント レベルを正しく維持するように注意してください。

    ```python
   # Create the agents client
   self.client = AgentsClient(
       endpoint=os.environ['PROJECT_ENDPOINT'],
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       )
   )
    ```

1. コメント "**Create the title agent**" を見つけて、エージェントを作成する次のコードを追加します。

    ```python
   # Create the title agent
   self.agent = self.client.create_agent(
       model=os.environ['MODEL_DEPLOYMENT_NAME'],
       name='title-agent',
       instructions="""
       You are a helpful writing assistant.
       Given a topic the user wants to write about, suggest a single clear and catchy blog post title.
       """,
   )
    ```

1. コメント "**Create a thread for the chat session**" を見つけて、チャット スレッドを作成する次のコードを追加します。

    ```python
   # Create a thread for the chat session
   thread = self.client.threads.create()
    ```

1. コメント "**Send user message**" を見つけて、ユーザーのプロンプトを送信する次のコードを追加します。

    ```python
   # Send user message
   self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. コメント "**Create and run the agent**" の下に、エージェントの応答生成を開始する次のコードを追加します。

    ```python
   # Create and run the agent
   run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    ファイルの残りの部分にあるコードにより、エージェントの応答が処理され、返されます。 

1. コード ファイルを保存します (*Ctrl+S* キー)。 これで、エージェントのスキルとカードを A2A プロトコルと共有する準備が整いました。 

1. 次のコマンドを入力して、エージェントの `server.py` ファイルを編集します  

    ```
   code server.py
    ```

1. コメント "**Define agent skills**" を見つけて、エージェントの機能を指定する次のコードを追加します。

    ```python
   # Define agent skills
   skills = [
       AgentSkill(
           id='generate_blog_title',
           name='Generate Blog Title',
           description='Generates a blog title based on a topic',
           tags=['title'],
           examples=[
               'Can you give me a title for this article?',
           ],
       ),
   ]
    ```

1. コメント "**Create agent card**" を見つけて、メタデータを定義する次のコードを追加します。これにより、エージェントは検出可能になります。

    ```python
   # Create agent card
   agent_card = AgentCard(
       name='AI Foundry Title Agent',
       description='An intelligent title generator agent powered by Azure AI Foundry. '
       'I can help you generate catchy titles for your articles.',
       url=f'http://{host}:{port}/',
       version='1.0.0',
       default_input_modes=['text'],
       default_output_modes=['text'],
       capabilities=AgentCapabilities(),
       skills=skills,
   )
    ```

1. コメント "**Create agent executor**" を見つけて、エージェント カードを使用してエージェント Executor を初期化する次のコードを追加します。

    ```python
   # Create agent executor
   agent_executor = create_foundry_agent_executor(agent_card)
    ```

    エージェント Executor は、作成したタイトル エージェントのラッパーとして機能します。

1. コメント "**Create request handler**" を見つけて、Executor を使用して受信要求を処理する次のコードを追加します。

    ```python
   # Create request handler
   request_handler = DefaultRequestHandler(
       agent_executor=agent_executor, task_store=InMemoryTaskStore()
   )
    ```

1. コメント "**Create A2A application**" の下に、A2A 互換のアプリケーション インスタンスを作成する次のコードを追加します。

    ```python
   # Create A2A application
   a2a_app = A2AStarletteApplication(
       agent_card=agent_card, http_handler=request_handler
   )
    ```
    
    このコードを使用すると、タイトル エージェントの情報を共有し、タイトル エージェント Executor を使用してこのエージェントへの受信要求を処理する A2A サーバーを作成することができます。

1. 完了したら、コード ファイルを保存します (*CTRL + S*)。

### エージェント間のメッセージを有効にする

このタスクでは、A2A プロトコルを使用して、ルーティング エージェントが他のエージェントにメッセージを送信できるようにします。 また、エージェント Executor クラスを実装することで、タイトル エージェントがメッセージを受信できるようにします。

1. `routing_agent` ディレクトリに移動します。

    ```
   cd ../routing_agent
    ```

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    ```
   code agent.py
    ```

    ルーティング エージェントは、ユーザー メッセージを処理し、どのリモート エージェントが要求を処理するかを決定するオーケストレーターとして機能します。

    ユーザー メッセージを受信すると、ルーティング エージェントは次の処理を実行します。
    - 会話スレッドを開始します。
    - `create_and_process` メソッドを使用して、ユーザーのメッセージに最適なエージェントを評価します。
    - このメッセージは、`send_message` 関数を使用して HTTP 経由で適切なエージェントにルーティングされます。
    - リモート エージェントによってメッセージが処理され、応答が返されます。

    最終的にルーティング エージェントによって応答が取り込まれ、スレッドを介してユーザーに返されます。

    `send_message` メソッドは非同期であり、エージェントの実行が正常に完了するまで待機する必要があることに注目してください。

1. コメント "**Retrieve the remote agent's A2A client using the agent name**" の下に次のコードを追加します。

    ```python
   # Retrieve the remote agent's A2A client using the agent name 
   client = self.remote_agent_connections[agent_name]
    ```

1. コメント "**Construct the payload to send to the remote agent**" を見つけて、次のコードを追加します。

    ```python
   # Construct the payload to send to the remote agent
   payload: dict[str, Any] = {
       'message': {
           'role': 'user',
           'parts': [{'kind': 'text', 'text': task}],
           'messageId': message_id,
       },
   }
    ```

1. コメント "**Wrap the payload in a SendMessageRequest object**" を見つけて、次のコードを追加します。

    ```python
   # Wrap the payload in a SendMessageRequest object
   message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. コメント "**Send the message to the remote agent client and await the response**" の下に次のコードを追加します。

    ```python
   # Send the message to the remote agent client and await the response
   send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```


1. 完了したら、コード ファイルを保存します (*CTRL + S*)。 これで、ルーティング エージェントはタイトル エージェントを検出し、メッセージを送信できるようになりました。 ルーティング エージェントからの受信メッセージを処理するエージェント Executor コードを作成しましょう。

1. `title_agent` ディレクトリに移動します。

    ```
   cd ../title_agent
    ```

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    ```
   code agent_executor.py
    ```

    `AgentExecutor` クラスの実装にはメソッド `execute` と `cancel` を含める必要があります。 cancel メソッドが用意されています。 `execute` メソッドには、イベントを管理し、タスクが完了したときに呼び出し元にシグナルを送る `TaskUpdater` オブジェクトが含まれています。 タスク実行のロジックを追加しましょう。

1. `execute` メソッドで、コメント "**Process the request**" の下に次のコードを追加します。

    ```python
   # Process the request
   await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. `_process_request` メソッドで、コメント "**Get the title agent**" の下に次のコードを追加します。

    ```python
   # Get the title agent
   agent = await self._get_or_create_agent()
    ```

1. コメント "**Update the task status**" の下に次のコードを追加します。

    ```python
   # Update the task status
   await task_updater.update_status(
       TaskState.working,
       message=new_agent_text_message('Title Agent is processing your request...', context_id=context_id),
   )
    ```

1. コメント "**Run the agent conversation**" を見つけて、次のコードを追加します。

    ```python
   # Run the agent conversation
   responses = await agent.run_conversation(user_message)
    ```

1. コメント "**Update the task with the responses**" を見つけて、次のコードを追加します。

    ```python
   # Update the task with the responses
   for response in responses:
       await task_updater.update_status(
           TaskState.working,
           message=new_agent_text_message(response, context_id=context_id),
       )
    ```

1. コメント "**Mark the task as complete**" を見つけて、次のコードを追加します。

    ```python
   # Mark the task as complete
   final_message = responses[-1] if responses else 'Task completed.'
   await task_updater.complete(
       message=new_agent_text_message(final_message, context_id=context_id)
   )
    ```

    これで、タイトル エージェントは、A2A プロトコルがメッセージを処理するために使用するエージェント Executor でラップされました。 上出来

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
    cd ..
    python run_all.py
    ```
    
    アプリケーションは、認証済みの Azure セッションの資格情報を使用して実行され、プロジェクトに接続してエージェントを作成して実行します。 各サーバーが起動すると、その出力が表示されます。

1. 入力のプロンプトが表示されるまで待ってから、次のようなプロンプトを入力します。

    ```
   Create a title and outline for an article about React programming.
    ```

    しばらくすると、エージェントから、結果を含む応答が表示されます。

1. `quit` と入力して、プログラムを終了し、サーバーを停止します。
    
## まとめ

この演習では、Azure AI エージェント サービス SDK と A2A Python SDK を使用して、リモート マルチ エージェント ソリューションを作成しました。 検出可能な A2A 互換エージェントを作成し、エージェントのスキルにアクセスするためのルーティング エージェントを設定しました。 また、受信 A2A メッセージを処理し、タスクを管理するエージェント Executor も実装しました。 上出来

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
