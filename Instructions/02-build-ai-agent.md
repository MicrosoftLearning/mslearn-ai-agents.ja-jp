---
lab:
  title: AI エージェントを開発する
  description: Azure AI エージェント サービスを利用して、組み込みツールを使用するエージェントを開発します。
---

# AI エージェントを開発する

この演習では、Azure AI エージェント サービスを利用して、データを分析してグラフを作成するシンプルなエージェントを作成します。 このエージェントは、組み込みの*コード インタープリター* ツールを使用して、グラフの作成に必要なコードを画像として動的に生成し、取得したグラフ イメージを保存します。

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

## エージェント クライアント アプリを作成する

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

    > **ヒント**: Cloudshell にコマンドを入力すると、出力が大量のスクリーン バッファーを占有し、現在の行のカーソルが見づらくなる可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

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
   pip install python-dotenv azure-identity azure-ai-projects
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_connection_string** プレースホルダーをプロジェクトの接続文字列 (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを GPT-4o モデル デプロイに割り当てた名前に置き換えます。
1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

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
   from azure.ai.projects.models import FilePurpose, CodeInterpreterTool
    ```

1. 「**Connect to the Azure AI Foundry project （Azure AI Foundry プロジェクトに接続する)**」コメントを見つけ、以下のコードを追加して Azure AI プロジェクトに接続します。

    > **ヒント**: インデント レベルを正しく維持するように注意してください。

    ```python
   # Connect to the Azure AI Foundry project
   project_client = AIProjectClient.from_connection_string(
        credential=DefaultAzureCredential
            (exclude_environment_credential=True,
             exclude_managed_identity_credential=True),
        conn_str=PROJECT_CONNECTION_STRING
   )
   with project_client:
    ```

    このコードは、現在使用中の Azure 資格情報で Azure AI Foundry プロジェクトに接続します。 最後の *with project_client* ステートメントを使用すると、クライアントのスコープを定義するコード ブロックが開始され、ブロック内のコードが完了したときにクリーンアップされます。

1. *with project_client* ブロック内に「**Upload the data file and create a CodeInterpreterTool （データファイルをアップロードして CodeInterpreterTool を作成する)**」コメントを見つけ、以下のコードを追加してデータ ファイルをプロジェクトにアップロードするとともに、その中のデータにアクセスできる CodeInterpreterTool を作成します。

    ```python
   # Upload the data file and create a CodeInterpreterTool
   file = project_client.agents.upload_file_and_poll(
        file_path=file_path, purpose=FilePurpose.AGENTS
   )
   print(f"Uploaded {file.filename}")

   code_interpreter = CodeInterpreterTool(file_ids=[file.id])
    ```
    
1. 「**Define an agent that uses the CodeInterpreterTool （CodeInterpreterTool を使用するエージェントを定義する)**」コメントを見つけ、以下のコードを追加して、データを分析し定義されたコード インタープリター ツールを使用する AI エージェントを定義します。

    ```python
   # Define an agent that uses the CodeInterpreterTool
   agent = project_client.agents.create_agent(
        model=MODEL_DEPLOYMENT,
        name="data-agent",
        instructions="You are an AI agent that analyzes the data in the file that has been uploaded. If the user requests a chart, create it and save it as a .png file.",
        tools=code_interpreter.definitions,
        tool_resources=code_interpreter.resources,
   )
   print(f"Using agent: {agent.name}")
    ```

1. 「**Create a thread for the conversation （会話用のスレッドを作成する)**」コメントを見つけ、以下のコードを追加して、エージェントとのチャット セッションを実行するスレッドを開始します。

    ```python
   # Create a thread for the conversation
   thread = project_client.agents.create_thread()
    ```
    
1. コードの次のセクションにはユーザーがプロンプトを入力するためのループが設定されており、ユーザーが「quit （終了）」と入力するとチャットが終了します。

1. 「**Send a prompt to the agen （エージェントにプロンプトを送信する)**」コメントを見つけ、以下のコードを追加して、プロンプトにユーザー メッセージ (および前の手順で読み込んだファイルのデータ) を追加して、エージェントでスレッドを実行します。

    ```python
   # Send a prompt to the agent
   message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_prompt,
    )

    run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)
     ```

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

1. 「**Get the conversation history (会話履歴を取得する)**」というコメントを見つけて、次のコードを追加して、会話スレッドからメッセージを出力します （順序を逆にして、時系列順に表示します)。

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = project_client.agents.list_messages(thread_id=thread.id)
   for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
    ```

1. 「**Get any generated files (生成されたファイルがあれば取得する)**」というコメントを見つけて、次のコードを追加して、メッセージからファイル パスの注釈 (エージェントが内部ストレージにファイルを保存していることを示します) を取得し、ファイルをアプリ フォルダーにコピーします。

    ```python
   # Get any generated files
   for file_path_annotation in messages.file_path_annotations:
        project_client.agents.save_file(file_id=file_path_annotation.file_path.file_id, file_name=Path(file_path_annotation.text).name)
        print(f"File saved as {Path(file_path_annotation.text).name}")
    ```

1. 「**Clean up （クリーンアップする)**」コメントを見つけ、以下のコードを追加して、不要になったエージェントとスレッドを削除します。

    ```python
   # Clean up
   project_client.agents.delete_agent(agent.id)
   project_client.agents.delete_thread(thread.id)
    ```

1. コードを確認します。コメントを見ることで以下のコード機能を把握できます。
    - どのように AI Foundry プロジェクトに接続しているか
    - どのようにデータ ファイルをアップロードし、かつそれにアクセスできるコード インタープリター ツールを作成しているか
    - どのようにコード インタープリター ツールを使用する新しいエージェントを作成すると共に、データの分析と.png ファイル形式でのグラフの作成に必要な明示的な指示を提供しているか。
    - 分析対象のデータとユーザーからのプロンプト メッセージを含むスレッドをどのように実行しているか。
    - エラーが発生した場合に実行の状態を確認します
    - 完了したスレッドからメッセージを取得し、エージェントによって最後に送信されたメッセージを表示します。
    - 会話履歴を表示する
    - 生成された各ファイルを保存します。
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

1. メッセージが表示されたら、アプリが *data.txt* テキスト ファイルから読み込んだデータを確認します。 次に、以下のようなプロンプトを入力します。

    ```
   What's the category with the highest cost?
    ```

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

1. 応答を表示します。 次に、グラフの作成を要求する別のプロンプトを入力します。

    ```
   Create a pie chart showing cost by category
    ```

    エージェントは必要に応じてコード インタープリター ツールを選択的に使用できる必要があり、ここでは受け取った要求に基づいてグラフを作成します。

1. 必要に応じて会話を続けることができます。 スレッドは*ステートフル*であるため、会話履歴を保持します。つまり、エージェントには各応答の完全なコンテキストが保持されます。 完了したら、「`quit`」と入力します。
1. スレッドから取得された会話メッセージと、生成されたファイルを確認します。

1. アプリケーションが完了したら、Cloud Shell の **download** コマンドを使用して、アプリ フォルダーに保存されている各.png ファイルをダウンロードします。 次に例を示します。

    ```
   download ./<file_name>.png
    ```

    ダウンロード コマンドを実行すると、ブラウザーの右下にポップアップ リンクが作成され、ここからファイルをダウンロードして開くことができます。

## まとめ

この演習では、Azure AI エージェント サービス SDK を使用して、AI エージェントを使用するクライアント アプリケーションを作成しました。 このエージェントは、組み込みのコード インタープリター ツールを使用して、イメージを作成する動的コードを実行します。

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
