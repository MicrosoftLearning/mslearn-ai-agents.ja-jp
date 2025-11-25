---
lab:
  title: Microsoft Agent Framework SDK を使用して Azure AI エージェントを開発する
  description: Microsoft Agent Framework SDK を使用して、Azure AI チャット エージェントを作成して使用する方法について学習します。
---

# Microsoft Agent Framework SDK を使用して Azure AI チャット エージェントを開発する

この演習では、Azure AI Agent サービスと Microsoft Agent Framework を使用して、経費請求を処理する AI エージェントを作成します。

この演習の所要時間は約 **30** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Microsoft Foundry プロジェクトにモデルをデプロイする

まず、Foundry プロジェクトにモデルをデプロイしましょう。

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインする場合に開かれるヒントまたはクイック スタートのペインを閉じ、必要に応じて、左上にある **[Foundry]** ロゴを使用してホーム ページに移動します。次の図のようなページが表示されます (**[ヘルプ]** ペインが表示される場合は閉じます)。

    ![Foundry ポータルのスクリーンショット。](./Media/ai-foundry-home.png)

    > **重要**: このラボの場合、**[新しい Foundry]** トグルが "オフ" になっていることを確認します。**

1. ホーム ページの **[モデルと機能を調査する]** セクションで、プロジェクトで使用する `gpt-4o` モデルを検索します。
1. 検索結果で **gpt-4o** モデルを選んで詳細を確認してから、モデルのページの上部にある **[このモデルを使用する]** を選択します。
1. プロジェクトの作成を求められたら、プロジェクトの有効な名前を入力し、**[詳細]** オプションを展開します。
1. プロジェクトについて次の設定を確認します。
    - **Foundry リソース**: "Foundry リソースの有効な名前"**
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **リージョン**: **AI Foundry が推奨する**もの*の中から選択します\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択し、選んだ gpt-4 モデル デプロイを含むプロジェクトが作成されるまで待ちます。
1. プロジェクトが作成されると、チャット プレイグラウンドが自動的に開きます。
1. **[セットアップ]** ウィンドウで、モデル デプロイの名前をメモします (**gpt-4o** のはずです)。 これを確認するには、**[モデルとエンドポイント]** ページでデプロイを表示します (左側のナビゲーション ウィンドウでそのページを開くだけです)。
1. 左側のナビゲーション ウィンドウで **[概要]** を選択すると、プロジェクトのメイン ページが表示されます。次のようになります。

    ![Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./Media/ai-foundry-project.png)

## エージェント クライアント アプリを作成する

これで、エージェントとカスタム関数を定義するクライアント アプリを作成する準備ができました。 いくつかの必要なコードが GitHub リポジトリに用意されています。

### 環境の準備

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

1. リポジトリが複製されたら、次のコマンドを入力して作業ディレクトリをコード ファイルを含むフォルダーに変更し、すべてを一覧表示します。

    ```
   cd ai-agents/Labfiles/04-agent-framework/python
   ls -a -l
    ```

    指定されたファイルには、アプリケーション コード、構成設定用のファイル、および経費データを含むファイルが含まれます。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install azure-identity agent-framework
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、プレースホルダー **your_project_endpoint** をプロジェクトのエンドポイント (Foundry ポータルのプロジェクトの **[概要]** ページからコピーした値) に置き換え、プレースホルダー **your_model_deployment** を gpt-4o モデル デプロイに割り当てた名前に置き換えます。
1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### エージェント アプリのコードを作成する

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 既存のコメントを参照して、同じレベルのインデントで新しいコードを入力します。

1. 次のコマンドを入力して、提供されたエージェント コード ファイルを編集します。

    ```
   code agent-framework.py
    ```

1. ファイル内のコードを確認します。 その構成要素を次に示します。
    - よく使用される名前空間への参照を追加するための一部の **import** ステートメント
    - 経費データを含むファイルを読み込み、ユーザーに指示を求めてから呼び出す *main* 関数。
    - エージェントを作成して使用するコードを追加する必要がある **process_expenses_data** 関数

1. ファイルの先頭にある既存の **import** ステートメントの後に「**Add references （参照の追加)**」コメントを見つけ、エージェントを実装するために必要なライブラリ内の名前空間を参照する次のコードを追加します。

    ```python
   # Add references
   from agent_framework import AgentThread, ChatAgent
   from agent_framework.azure import AzureAIAgentClient
   from azure.identity.aio import AzureCliCredential
   from pydantic import Field
   from typing import Annotated
    ```

1. ファイルの下部にある **Create a tool function for the email functionality (電子メール機能用のツール関数を作成する)** というコメントを見つけて次のコードを追加し、エージェントが電子メールを送信するために使用する関数を定義します (ツールは、エージェントにカスタム機能を追加する手段です)

    ```python
   # Create a tool function for the email functionality
   def send_email(
    to: Annotated[str, Field(description="Who to send the email to")],
    subject: Annotated[str, Field(description="The subject of the email.")],
    body: Annotated[str, Field(description="The text body of the email.")]):
        print("\nTo:", to)
        print("Subject:", subject)
        print(body, "\n")
    ```

    > **注**: この関数は、コンソールにメールを出力することで電子メールの送信を*シミュレート*します。 実際のアプリケーションでは、SMTP サービスなどの手法で電子メールを実際に送信します。

1. **send_email** コードの上に戻り、**process_expenses_data** 関数で、**Create a chat agent (チャット エージェントを作成する)** というコメントを見つけ、次のコードを追加して、ツールと指示を含む **ChatAgent** オブジェクトを作成します。

    (インデント レベルは必ず維持してください)

    ```python
   # Create a chat agent
   async with (
       AzureCliCredential() as credential,
       ChatAgent(
           chat_client=AzureAIAgentClient(async_credential=credential),
           name="expenses_agent",
           instructions="""You are an AI assistant for expense claim submission.
                           When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                           Then confirm to the user that you've done so.""",
           tools=send_email,
       ) as agent,
   ):
    ```

    **AzureCliCredential** オブジェクトを使用すると、お使いの Azure アカウントに対してコードが認証できるようになることに注意してください。 **AzureAIAgentClient** オブジェクトには、.env 構成の Foundry プロジェクト設定が自動的に含まれます。

1. 「**Use the agent to process the expenses data （エージェントを使用して経費データを処理する)**」というコメントを見つけて、次のコードを追加し、エージェントを実行するスレッドを作成してから、チャット メッセージで呼び出します。

    (インデント レベルは必ず維持してください)。

    ```python
   # Use the agent to process the expenses data
   try:
       # Add the input prompt to a list of messages to be submitted
       prompt_messages = [f"{prompt}: {expenses_data}"]
       # Invoke the agent for the specified thread with the messages
       response = await agent.run(prompt_messages)
       # Display the response
       print(f"\n# Agent:\n{response}")
   except Exception as e:
       # Something went wrong
       print (e)
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
    
1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、Foundry ハブを含むサブスクリプションを選択します。
1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```
   python agent-framework.py
    ```
    
    アプリケーションは、認証済みの Azure セッションの資格情報を使用して実行され、プロジェクトに接続してエージェントを作成して実行します。

1. 経費データの処理方法を質問されたら、次のプロンプトを入力します。

    ```
   Submit an expense claim
    ```

1. アプリケーションが完了したら、出力を確認します。 エージェントは、提供されたデータに基づいて経費請求の電子メールを作成するはずです。

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

## まとめ

この演習では、Microsoft Agent Framework SDK を使用して、カスタム ツールを使用したエージェントを作成しました。

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
