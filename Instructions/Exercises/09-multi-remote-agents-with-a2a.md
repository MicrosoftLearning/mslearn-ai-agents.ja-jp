---
lab:
  title: A2A プロトコルを使用してリモート エージェントに接続する
  description: A2A プロトコルを使用してリモート エージェントと共同作業を行います。
  level: 300
  duration: 30
  islab: true
---

# A2A プロトコルを使用してリモート エージェントに接続する

この演習では、Azure AI エージェント サービスと A2A プロトコルを使用して、相互に対話するシンプルなリモート エージェントを作成します。 これらのエージェントには、開発者ブログ記事を準備するテクニカル ライターを支援する機能があります。 タイトル エージェントを使用て見出しを生成し、アウトライン エージェントでタイトルを使用して記事の簡潔なアウトラインを作成します。 では、始めましょう。

> **ヒント**: この演習で使われるコードは、Microsoft Foundry SDK for Python が基になっています。 Microsoft .NET、JavaScript、Java 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、[Microsoft Foundry SDK クライアント ライブラリ](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)に関するページを参照してください。

この演習の所要時間は約 **30** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

この演習を開始するには、以下のものが必要です。

- ローカル コンピューターに [Visual Studio Code](https://code.visualstudio.com/) がインストールされている
- 有効な [Azure サブスクリプション](https://azure.microsoft.com/free/)
- [Python 3.13](https://www.python.org/downloads/) 以降がインストールされている
- ローカル コンピューターに [Git](https://git-scm.com/downloads) がインストールされている

> \* Python 3.13 を使用できますが、一部の依存関係がそのリリース用にまだコンパイルされていません。 このラボでは Python 3.13.12 でテストが正常に終了しました。

## Foundry Toolkit for VS Code 拡張機能を使用して Foundry プロジェクトを作成する

開発者は、Foundry ポータルである程度の時間作業しますが、Visual Studio Code でも多くの時間を費やす可能性があります。 Foundry Toolkit for VS Code 拡張機能は、開発環境を離れることなく Foundry プロジェクト リソースを操作するための便利な方法を提供します。

1. Visual Studio Code を開きます。

2. 左側のペインから **[拡張機能]** を選択します (または **Ctrl + Shift + X** キーを押します)。

3. 拡張機能マーケットプレースで Microsoft の `Foundry Toolkit` 拡張機能を検索し、**[インストール]** を選択します。

    > **注**: 拡張機能は現在、**Foundry Toolkit** として登録されていますが、一部の VS Code のラベル、コマンド、または以前のスクリーンショットでは、引き続き **AI Toolkit** と表示されている場合があります。 このラボでは、これらの名前が同じ拡張機能エクスペリエンスを表しているものとして扱います。

4. 拡張機能をインストールしたら、サイド バーでそのアイコンを選択して Foundry Toolkit ビューを開きます。 

    Azure アカウントへのサインインがまだの場合は、プロンプトが表示されるはずです。

5. **[Microsoft Foundry リソース]** で **[プロジェクトの作成]** を選択します。

    既定のプロジェクトが既にアクティブな場合、プロジェクト名は **[マイ リソース]** の下に表示されます。 新しいプロジェクトを作成するには、アクティブなプロジェクトを右クリックし、**[Azure 拡張機能での既定プロジェクトの切り替え]** を選択します。

6. Azure サブスクリプションとリソース グループを選択し、Foundry プロジェクトの名前を入力して、この演習用の新しいプロジェクトを作成します。

    デプロイが完了すると、プロジェクトが既定のプロジェクトとして [Foundry Toolkit] ペインに表示されます。

## モデルをデプロイする

どの生成 AI プロジェクトの中核にも、少なくとも 1 つの生成 AI モデルがあります。 このタスクでは、エージェントで使用するモデル カタログからモデルをデプロイします。

1. "プロジェクトが正常にデプロイされました" というポップアップが表示されたら、**[新しいモデルのデプロイ]** ボタンを選択します。 これにより、モデル カタログが開きます。

   > **ヒント**: モデル カタログにアクセスするには、[リソース] セクションで **[モデル]** の横にある **[+]** アイコンを選択するか、**F1** キーを押し、コマンド「**AI Toolkit: Show model Catalog**」を実行する方法もあります。

1. モデル カタログで、**[gpt-4.1]** モデルを見つけます (検索バーを使用するとすばやく見つけることができます)。

2. gpt-4.1 モデルの横にある **[デプロイ]** を選択します。

3. デプロイ設定を構成します。
   - **デプロイ名**:"gpt-4.1" のような名前を入力します
   - **デプロイの種類**:**[グローバル標準]** (グローバル標準を使用できない場合は **[標準]**) を選択します
   - **モデルのバージョン**: デフォルトのままにする
   - **1 分あたりのトークン数**:デフォルトのままにする

4. 左下隅にある **[Microsoft Foundry にデプロイ]** を選択します。

5. デプロイが完了するまで待ちます。 デプロイしたモデルが、[リソース] ビューの **[モデル]** セクションに表示されます。

6. プロジェクトのデプロイ名を右クリックし、**[プロジェクト エンドポイントのコピー]** を選択します。 次のステップでエージェントを Foundry プロジェクトに接続するために、この URL が必要になります。

    ![Foundry Toolkit VS Code 拡張機能でのプロジェクト エンドポイントのコピーを示すスクリーンショット。](../Media/vs-code-endpoint.png)
    
## スタート コード リポジトリをクローンする

この演習では、Foundry プロジェクトに接続し、経費データを処理できるエージェントを作成するのに役立つスタート コードを使用します。 このコードは GitHub リポジトリから複製することになります。

1. VS Code で、コマンド パレット (**Ctrl + Shift + P** または **[表示] > [コマンド パレット]**) を開きます。

1. 「**Git:Clone**」と入力してこれを一覧から選択します。

1. リポジトリの URL を入力します。

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. リポジトリをクローンするローカル コンピューター上の場所を選択します。

1. プロンプトが表示されたら **[開く]** を選択して、VS Code でクローンされたリポジトリを開きます。

1. リポジトリが開いたら、**[ファイル] > [フォルダーを開く]** を選択し、`mslearn-ai-agents/Labfiles/06-build-remote-agents-with-a2a` に移動して、**[フォルダーの選択]** を選択します。

1. [エクスプローラー] ペインで、**[Python]** フォルダーを展開して、この演習のコード ファイルを表示します。 

1. [エクスプローラー] ビューで **Labfiles/06-build-remote-agents-with-a2a/Python** フォルダーに移動します。ここに、この演習用のスタート コードがあります。

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

1. **requirements.txt** ファイルを右クリックし、**[統合ターミナルで開く]** を選択します。

1. ターミナルで、次のコマンドを入力して、仮想環境に必要な Python パッケージをインストールします。

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

1. **.env** ファイルを開き、**your_project_endpoint** プレースホルダーをプロジェクトのエンドポイント (Foundry Toolkit 拡張機能のプロジェクト デプロイ リソースからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名に設定されていることを確認します。 これらの変更を行った後、**Ctrl + S** を使用してファイルを保存します。

## 検出可能なエージェントを作成する

このタスクでは、ライターが記事の傾向に沿ったヘッドラインを作成できるように支援するタイトル エージェントを作成します。 また、エージェントを検出可能にするために、A2A プロトコルで必要なエージェントのスキルとカードも定義します。

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 既存のコメントを参照して、同じレベルのインデントで新しいコードを入力します。

1. コード エディターで **title_agent/agent.py** ファイルを開きます。

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

1. コード エディターで **title_agent/server.py** ファイルを開きます。

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
       name='Microsoft Foundry Title Agent',
       description='An intelligent title generator agent powered by Foundry. '
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

## エージェント間のメッセージを有効にする

このタスクでは、A2A プロトコルを使用して、ルーティング エージェントが他のエージェントにメッセージを送信できるようにします。 また、エージェント Executor クラスを実装することで、タイトル エージェントがメッセージを受信できるようにします。

1. コード エディターで **routing_agent/agent.py** ファイルを開きます。

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

1. コード エディターで **title_agent/agent_executor.py** ファイルを開きます。

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

## アプリケーションをテストする

1. 統合ターミナルで、次のコマンドを入力してアプリケーションを実行します。
    ```
    az login
    ```

    ```
    python run_all.py
    ```

    アプリケーションは、認証済みの Azure セッションの資格情報を使用して実行され、プロジェクトに接続してエージェントを作成して実行します。 各サーバーが起動すると、その出力が表示されます。

1. 入力のプロンプトが表示されるまで待ってから、次のようなプロンプトを入力します。

    ```
   Create a title and outline for an article about React programming.
    ```

    しばらくすると、エージェントから、結果を含む応答が表示されます。

1. `quit` と入力して、プログラムを終了し、サーバーを停止します。

    `deactivate` を使用して、ターミナルの Python 仮想環境を終了することもできます。

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
