---
lab:
  title: Microsoft Agent Framework SDK を使用して Azure AI エージェントを開発する
  description: Microsoft Agent Framework SDK を使用して、Azure AI チャット エージェントを作成して使用する方法について学習します。
  level: 300
  duration: 30
  islab: true
---

# Microsoft Agent Framework SDK を使用して Azure AI チャット エージェントを開発する

この演習では、Azure AI Agent サービスと Microsoft Agent Framework を使用して、経費請求を処理する AI エージェントを作成します。

この演習の所要時間は約 **30** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

この演習を開始するには、以下のものが必要です。

- ローカル コンピューターにインストールされている [Visual Studio Code](https://code.visualstudio.com/)
- 有効な [Azure サブスクリプション](https://azure.microsoft.com/free/)
- [Python 3.13](https://www.python.org/downloads/) 以降がインストールされている
- ローカル コンピューターにインストールされている [Git](https://git-scm.com/downloads)

> \* Python 3.13 を使用できますが、一部の依存関係がそのリリース用にまだコンパイルされていません。 このラボでは Python 3.13.12 でテストが正常に終了しました。

## AI Toolkit VS Code 拡張機能を使用して Foundry プロジェクトを作成する

開発者は、Foundry ポータルである程度の時間作業しますが、Visual Studio Code でも多くの時間を費やす可能性があります。 AI Toolkit 拡張機能は、開発環境を離れることなく Foundry プロジェクト リソースを操作するための便利な方法を提供します。

1. Visual Studio Code を開きます。

2. 左側のペインから **[拡張機能]** を選択します (または **Ctrl + Shift + X** キーを押します)。

3. 拡張機能マーケットプレースで Microsoft の `AI Toolkit` 拡張機能を検索し、**[インストール]** を選択します。

4. 拡張機能をインストールしたら、サイド バーでそのアイコンを選択して AI Toolkit ビューを開きます。 

    Azure アカウントへのサインインがまだの場合は、プロンプトが表示されるはずです。
   
5. **[Microsoft Foundry リソース]** で **[プロジェクトの作成]** を選択します。

    既定のプロジェクトが既にアクティブな場合、プロジェクト名は **[マイ リソース]** の下に表示されます。 新しいプロジェクトを作成するには、アクティブなプロジェクトを右クリックし、**[既定プロジェクトの切り替え]** を選択します。

6. Azure サブスクリプションとリソース グループを選択し、Foundry プロジェクトの名前を入力して、この演習用の新しいプロジェクトを作成します。

    デプロイが完了すると、プロジェクトが既定のプロジェクトとして [AI Toolkit] ペインに表示されます。

## モデルをデプロイする

どの生成 AI プロジェクトの中核にも、少なくとも 1 つの生成 AI モデルがあります。 このタスクでは、エージェントで使用するモデル カタログからモデルをデプロイします。

1. "プロジェクトが正常にデプロイされました" というポップアップが表示されたら、**[モデルのデプロイ]** ボタンを選択します。 これにより、モデル カタログが開きます。

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

    ![AI Toolkit VS Code 拡張機能でプロジェクト エンドポイントをコピーしているスクリーンショット。](../Media/vs-code-endpoint.png)

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

1. リポジトリが開いたら、**[ファイル] > [フォルダーを開く]** を選択し、`mslearn-ai-agents/Labfiles/07-agent-framework` に移動して、**[フォルダーの選択]** を選択します。

1. [エクスプローラー] ペインで、**[Python]** フォルダーを展開して、この演習のコード ファイルを表示します。 

1. **requirements.txt** ファイルを右クリックし、**[統合ターミナルで開く]** を選択します。

1. ターミナルで、次のコマンドを入力して、必要な Python パッケージ仮想環境にインストールします。

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

1. **.env** ファイルを開き、**[your_project_endpoint]** プレースホルダーをプロジェクトのエンドポイント (AI Toolkit 拡張機能のプロジェクト デプロイ リソースからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名に設定されていることを確認します。 これらの変更を行った後、**Ctrl + S** を使用してファイルを保存します。

これで、カスタム ツールを使用して経費データを処理する AI エージェントを作成する準備ができました。

## カスタム ツールを使用してエージェントを作成する

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 既存のコメントを参照して、同じレベルのインデントで新しいコードを入力します。

1. コード エディターで **agent-framework.py** ファイルを開きます。

1. ファイル内のコードを確認します。 その構成要素を次に示します。
    - よく使用される名前空間への参照を追加するための一部の **import** ステートメント
    - 経費データを含むファイルを読み込み、ユーザーに指示を求めてから呼び出す *main* 関数。
    - エージェントを作成して使用するコードを追加する必要がある **process_expenses_data** 関数

1. ファイルの先頭にある既存の **import** ステートメントの後に「**Add references （参照の追加)**」コメントを見つけ、エージェントを実装するために必要なライブラリ内の名前空間を参照する次のコードを追加します。

    ```python
   # Add references
   from agent_framework import tool, Agent
   from agent_framework.azure import AzureOpenAIResponsesClient
   from azure.identity import AzureCliCredential
   from pydantic import Field
   from typing import Annotated
    ```

1. ファイルの下部にある **Create a tool function for the email functionality (電子メール機能用のツール関数を作成する)** というコメントを見つけて次のコードを追加し、エージェントが電子メールを送信するために使用する関数を定義します (ツールは、エージェントにカスタム機能を追加する手段です)

    ```python
   # Create a tool function for the email functionality
   @tool(approval_mode="never_require")
   def submit_claim(
       to: Annotated[str, Field(description="Who to send the email to")],
       subject: Annotated[str, Field(description="The subject of the email.")],
       body: Annotated[str, Field(description="The text body of the email.")]):
           print("\nTo:", to)
           print("Subject:", subject)
           print(body, "\n")
    ```

    > **注**: この関数は、コンソールにメールを出力することで電子メールの送信を*シミュレート*します。 実際のアプリケーションでは、SMTP サービスなどの手法で電子メールを実際に送信します。

1. **send_email** コードの上に戻り、**process_expenses_data** 関数で、コメント **Create a client and initialize an agent with the tool and instructions** を見つけ、次のコードを追加します。

    (インデント レベルは必ず維持してください)

    ```python
   # Create a client and initialize an agent with the tool and instructions
   credential = AzureCliCredential()
   async with (
        Agent(
            client=AzureOpenAIResponsesClient(
                credential=credential,
                deployment_name=os.getenv("MODEL_DEPLOYMENT_NAME"),
                project_endpoint=os.getenv("PROJECT_ENDPOINT"),
            ),
            instructions="""You are an AI assistant for expense claim submission.
                        At the user's request, create an expense claim and use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                        Then confirm to the user that you've done so. Don't ask for any more information from the user, just use the data provided to create the email.""",
            tools=[submit_claim],
        ) as agent,
    ):
    ```

    **AzureCliCredential** オブジェクトを使用すると、お使いの Azure アカウントに対してコードが認証できるようになることに注意してください。 **AzureOpenAIResponsesClient** オブジェクトには、.env 構成からの Foundry プロジェクト設定が含まれています。 **Agent** オブジェクトは、クライアント、エージェントへの指示、およびメール送信のために定義したツール関数を使用して初期化されます。

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

## アプリケーションをテストする

1. 統合ターミナルで、次のコマンドを入力してアプリケーションを実行します。

    ```
   python agent-framework.py
    ```

1. 経費データの処理方法を質問されたら、次のプロンプトを入力します。

    ```
   Submit an expense claim
    ```

1. アプリケーションが完了したら、出力を確認します。 エージェントは、提供されたデータに基づいて経費請求の電子メールを作成するはずです。

    > **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

1. 完了したら、ターミナルに「`deactivate`」と入力して、Python 仮想環境を終了します。

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

### モデルの削除

1. VS Code で、**[Azure リソース]** ビューを更新します。

1. **[モデル]** サブセクションを展開します。

1. デプロイしたモデルを右クリックし、**[削除]** を選択します。

### リソース グループを削除します

1. [Azure Portal](https://portal.azure.com)を開きます。

1. Microsoft Foundry リソースを含むリソース グループに移動します。

1. **[リソース グループの削除]** を選択し、削除を確認します。
