---
lab:
  title: セマンティック カーネル SDK を使用して Azure AI エージェントを開発する
  description: セマンティック カーネル SDK を使用して、Azure AI エージェント サービス エージェントを作成して使用する方法について説明します。
---

# セマンティック カーネル SDK を使用して Azure AI エージェントを開発する

この演習では、Azure AI エージェント サービスとセマンティック カーネルを使用して、経費請求を処理する AI エージェントを作成します。

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
    - **場所**: 次のいずれかのリージョンを選択します:\*
        - eastus
        - eastus2
        - swedencentral
        - westus
        - westus3
    - **Azure AI サービスまたは Azure OpenAI への接続**: *新しい AI サービス リソースを作成します*
    - **Azure AI 検索への接続**:接続をスキップする

    > \* この記事の執筆時点において、これらのリージョンでは、エージェント内で使用する gpt-4o モデルがサポートされています。 モデルの可用性は、リージョンのクォータによって制限されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のプロジェクトを作成する必要が生じる可能性があります。

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

これで、エージェントとカスタム関数を定義するクライアント アプリを作成する準備ができました。 いくつかの必要なコードが GitHub リポジトリに用意されています。

### 環境の準備

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

1. リポジトリが複製されたら、次のコマンドを入力して作業ディレクトリをコード ファイルを含むフォルダーに変更し、すべてを一覧表示します。

    ```
   cd ai-agents/Labfiles/04-semantic-kernel/python
   ls -a -l
    ```

    指定されたファイルには、アプリケーション コード、構成設定用のファイル、および経費データを含むファイルが含まれます。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    > **注**: *semantic-kernel[azure]* をインストールすると、セマンティック カーネル互換バージョンの *azure-ai-projects* が自動的にインストールされます。

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイル内で、**[your_project_connection_string]** プレースホルダーをプロジェクトの接続文字列 (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**[your_model_deployment]** プレースホルダーを gpt-4o モデル デプロイに割り当てた名前に置き換えます。
1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### エージェント アプリのコードを作成する

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 既存のコメントを参照して、同じレベルのインデントで新しいコードを入力します。

1. 次のコマンドを入力して、提供されたエージェント コード ファイルを編集します。

    ```
   code semantic-kernel.py
    ```

1. ファイル内のコードを確認します。 その構成要素を次に示します。
    - よく使用される名前空間への参照を追加するための一部の **import** ステートメント
    - 経費データを含むファイルを読み込み、ユーザーに指示を求めてから呼び出す *main* 関数。
    - エージェントを作成して使用するコードを追加する必要がある **process_expenses_data** 関数
    - **send_email** という名前のカーネル関数を含む **EmailPlugin** クラス。これは、エージェントが電子メールの送信に使用する機能をシミュレートするために使用します。

1. ファイルの先頭にある既存の **import** ステートメントの後に「**Add references （参照の追加)**」コメントを見つけ、エージェントを実装するために必要なライブラリ内の名前空間を参照する次のコードを追加します。

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity.aio import DefaultAzureCredential
   from semantic_kernel.agents import AzureAIAgent, AzureAIAgentSettings, AzureAIAgentThread
   from semantic_kernel.functions import kernel_function
   from typing import Annotated
    ```

1. ファイルの下部にある「**Create a Plugin for the email functionality (電子メール機能用のプラグインを作成する)**」コメントを見つけて次のコードを追加し、エージェントが電子メールの送信に使用する関数を含むプラグインのクラスを定義します (プラグインは、セマンティック カーネル エージェントにカスタム機能を追加する方法です)。

    ```python
   # Create a Plugin for the email functionality
   class EmailPlugin:
       """A Plugin to simulate email functionality."""
    
       @kernel_function(description="Sends an email.")
       def send_email(self,
                      to: Annotated[str, "Who to send the email to"],
                      subject: Annotated[str, "The subject of the email."],
                      body: Annotated[str, "The text body of the email."]):
           print("\nTo:", to)
           print("Subject:", subject)
           print(body, "\n")
    ```

    > **注**: この関数は、コンソールにメールを出力することで電子メールの送信を*シミュレート*します。 実際のアプリケーションでは、SMTP サービスなどの手法で電子メールを実際に送信します。

1. 新しい **EmailPlugin** クラス コードの上に戻り、**create_expense_claim** 関数内で「**Get configuration settings (校正設定を取得する)**」というコメントを見つけて、次のコードを追加し、構成ファイルをロードして **AzureAIAgentSettings** オブジェクト (構成からの Azure AI エージェントの設定を自動的に含みます) を作成します。

    (インデント レベルは必ず維持してください)

    ```python
   # Get configuration settings
   load_dotenv()
   ai_agent_settings = AzureAIAgentSettings()
    ```

1. 「**Connect to the Azure AI Foundry project (Azure AI Foundry プロジェクトに接続する)**」コメントを見つけ、以下のコードを追加して、現在サインインに使用している Azure 資格情報で Azure AI Foundry プロジェクトに接続します。

    (インデント レベルは必ず維持してください)

    ```python
   # Connect to the Azure AI Foundry project
   async with (
        DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True) as creds,
        AzureAIAgent.create_client(
            credential=creds
        ) as project_client,
   ):
    ```

1. 「**Define an Azure AI agent that sends an expense claim email （経費請求の電子メールを送信する Azure AI エージェントを定義する)**」コメントを見つけ、以下のコードを追加して、使用するエージェント向けの Azure AI エージェント定義を作成します。

    (インデント レベルは必ず維持してください)

    ```python
   # Define an Azure AI agent that sends an expense claim email
   expenses_agent_def = await project_client.agents.create_agent(
        model= ai_agent_settings.model_deployment_name,
        name="expenses_agent",
        instructions="""You are an AI assistant for expense claim submission.
                        When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                        Then confirm to the user that you've done so."""
   )
    ```

1. 「**Create a semantic kernel agent (Semantic Kernel エージェントを作成する)**」コメントを見つけ、以下のコードを追加して、使用する Azure AI エージェント向けの Semantic Kernel エージェント オブジェクトを作成し、**EmailPlugin** プラグインへの参照を含めます。

    (インデント レベルは必ず維持してください)

    ```python
   # Create a semantic kernel agent
   expenses_agent = AzureAIAgent(
        client=project_client,
        definition=expenses_agent_def,
        plugins=[EmailPlugin()]
   )
    ```

1. 「**Use the agent to process the expenses data （エージェントを使用して経費データを処理する)**」というコメントを見つけて、次のコードを追加し、エージェントを実行するスレッドを作成してから、チャット メッセージで呼び出します。

    (インデント レベルは必ず維持してください)。

    ```python
   # Use the agent to process the expenses data
   thread: AzureAIAgentThread = AzureAIAgentThread(client=project_client)
   try:
        # Add the input prompt to a list of messages to be submitted
        prompt_messages = [f"{prompt}: {expenses_data}"]
        # Invoke the agent for the specified thread with the messages
        response = await expenses_agent.get_response(thread_id=thread.id, messages=prompt_messages)
        # Display the response
        print(f"\n# {response.name}:\n{response}")
   except Exception as e:
        # Something went wrong
        print (e)
   finally:
        # Cleanup: Delete the thread and agent
        await thread.delete() if thread else None
        await project_client.agents.delete_agent(expenses_agent.id)
    ```

1. 完成したエージェント向けコードを確認し (コメントを見ることで各コード ブロックの機能を理解できます)、コードの変更を保存します (**CTRL+S**)。
1. コード内の入力ミスを修正する必要がある場合は、コード エディターを開いたままにしておきますが、コマンド ライン コンソールをもっと大きく表示できるようにペイン サイズを変更すること。

### Azure にサインインしてアプリを実行する

1. コード エディターの下の Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    ```
    az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。
    
1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、Azure AI Foundry ハブを含むサブスクリプションを選択します。
1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```
   python semantic-kernel.py
    ```
    
    アプリケーションは、認証済みの Azure セッションの資格情報を使用して実行され、プロジェクトに接続してエージェントを作成して実行します。

1. 経費データの処理方法を質問されたら、次のプロンプトを入力します。

    ```
   Submit an expense claim
    ```

1. アプリケーションが完了したら、出力を確認します。 エージェントは、提供されたデータに基づいて経費請求の電子メールを作成するはずです。

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

## まとめ

この演習では、Azure AI Agent Service SDK とセマンティック カーネルを使用してエージェントを作成しました。

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
