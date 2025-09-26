---
lab:
  title: セマンティック カーネルを使用してマルチエージェント ソリューションを開発する
  description: Semantic Kernel SDK を使用して共同作業するよう複数のエージェントを構成する方法について説明します
---

# マルチエージェント ソリューションの開発

この演習では、セマンティック カーネル SDK で順次オーケストレーション パターンを使用する方法を練習します。 連携して顧客からのフィードバックを処理し、次の手順を提案する 3 つのエージェントのシンプルなパイプラインを作成します。 次のエージェントを作成します。

- サマライザー エージェントは、生のフィードバックを短い中立的な文に要約します。
- 分類子エージェントは、フィードバックを肯定的、否定的、または機能要求に分類します。
- 最後に、推奨アクション エージェントは、適切なフォローアップ手順を推奨します。

セマンティック カーネル SDK を使用して問題を切り分け、適切なエージェントにルーティングし、実用的な結果を生成する方法について学習します。 それでは始めましょう。

> **ヒント**: この演習で使用するコードは、Python 用の Semantic Kernel SDK に基づいています。 Microsoft .NET および Java 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、「[サポートされているセマンティック カーネル言語](https://learn.microsoft.com/semantic-kernel/get-started/supported-languages)」を参照してください。

この演習の所要時間は約 **30** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Azure AI Foundry プロジェクトにモデルをデプロイする

まず、Azure AI Foundry プロジェクトにモデルをデプロイします。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./Media/ai-foundry-home.png)

1. ホーム ページの **[モデルと機能を調査する]** セクションで、プロジェクトで使用する `gpt-4o` モデルを検索します。
1. 検索結果で **gpt-4o** モデルを選んで詳細を確認してから、モデルのページの上部にある **[このモデルを使用する]** を選択します。
1. プロジェクトの作成を求められたら、プロジェクトの有効な名前を入力し、**[詳細]** オプションを展開します。
1. プロジェクトについて次の設定を確認します。
    - **Azure AI Foundry リソース**: *Azure AI Foundry リソースの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **リージョン**: **AI Foundry が推奨する**もの*の中から選択します\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択し、選んだ gpt-4 モデル デプロイを含むプロジェクトが作成されるまで待ちます。

1. プロジェクトが作成されると、チャット プレイグラウンドが自動的に開きます。

1. 左側のナビゲーション ウィンドウで、**[モデルとエンドポイント]** を選択し、**[gpt-4o]** デプロイを選択します。

1. **[セットアップ]** ウィンドウで、モデル デプロイの名前をメモします (**gpt-4o** のはずです)。 これを確認するには、**[モデルとエンドポイント]** ページでデプロイを表示します (左側のナビゲーション ウィンドウでそのページを開くだけです)。
1. 左側のナビゲーション ウィンドウで **[概要]** を選択すると、プロジェクトのメイン ページが表示されます。次のようになります。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./Media/ai-foundry-project.png)

## AI エージェント クライアント アプリを作成する

これで、エージェントとカスタム関数を定義するクライアント アプリを作成する準備ができました。 GitHub リポジトリには、いくつかのコードが用意されています。

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

    > **ヒント**: Cloud Shell にコマンドを入力すると、出力が大量のスクリーン バッファーを占有し、現在のライン上のカーソルが隠れてしまう可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. リポジトリが複製されたら、次のコマンドを入力して作業ディレクトリをコード ファイルを含むフォルダーに変更し、すべてを一覧表示します。

    ```
   cd ai-agents/Labfiles/05-agent-orchestration/Python
   ls -a -l
    ```

    指定されたファイルには、アプリケーション コードと構成設定用のファイルがあります。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel --upgrade
    ```

    > **注**:*semantic-kernel* をインストールすると、セマンティック カーネル互換バージョンの *azure-ai-projects* が自動的にインストールされます。

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**[your_openai_endpoint]** プレースホルダーをプロジェクトの Azure Open AI エンドポイント (Azure AI Foundry ポータルの **Azure OpenAI** の下にあるプロジェクトの **[概要]** ページからコピーしたもの) に置き換えます。 **[your_openai_api_key]** をプロジェクトの API キーに置き換え、MODEL_DEPLOYMENT_NAME 変数がモデルのデプロイ名 (*gpt-4o*) に設定されていることを確認します。

1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### AI エージェントを作成する

これで、マルチエージェント ソリューション向けにエージェントを作成する準備ができました。 それでは始めましょう。

1. 次のコマンドを入力して、**agent.py** ファイルを編集します。

    ```
   code agents.py
    ```

1. ファイルの先頭にあるコメント **Add references (参照を追加する)** の下に、エージェントを実装するために必要なライブラリ内の名前空間を参照する次のコードを追加します。

    ```python
   # Add references
   import asyncio
   from semantic_kernel.agents import Agent, ChatCompletionAgent, SequentialOrchestration
   from semantic_kernel.agents.runtime import InProcessRuntime
   from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
   from semantic_kernel.contents import ChatMessageContent
    ```


1. **get_agents** 関数で、コメント **Create a summarizer agent (サマライザー エージェントを作成する)** の下に次のコードを追加します。

    ```python
   # Create a summarizer agent
   summarizer_agent = ChatCompletionAgent(
       name="SummarizerAgent",
       instructions="""
       Summarize the customer's feedback in one short sentence. Keep it neutral and concise.
       Example output:
       App crashes during photo upload.
       User praises dark mode feature.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. コメント **Create a classifier agent (分類子エージェントを作成する)** の下に次のコードを追加します。

    ```python
   # Create a classifier agent
   classifier_agent = ChatCompletionAgent(
       name="ClassifierAgent",
       instructions="""
       Classify the feedback as one of the following: Positive, Negative, or Feature request.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. コメント **Create a recommended action agent (推奨アクション エージェントを作成する)** の下に次のコードを追加します。

    ```python
   # Create a recommended action agent
   action_agent = ChatCompletionAgent(
       name="ActionAgent",
       instructions="""
       Based on the summary and classification, suggest the next action in one short sentence.
       Example output:
       Escalate as a high-priority bug for the mobile team.
       Log as positive feedback to share with design and marketing.
       Log as enhancement request for product backlog.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. コメント **Return a list of agents (エージェントの一覧を返す)** の下に次のコードを追加します。

    ```python
   # Return a list of agents
   return [summarizer_agent, classifier_agent, action_agent]
    ```

    この一覧のエージェントの順序が、オーケストレーション中に選択される順序になります。

## 順次オーケストレーションを作成する

1. **main** 関数で、コメント **Initialize the input task (入力タスクを初期化する)** を見つけて、次のコードを追加します。
    
    ```python
   # Initialize the input task
   task="""
   I tried updating my profile picture several times today, but the app kept freezing halfway through the process. 
   I had to restart it three times, and in the end, the picture still wouldn't upload. 
   It's really frustrating and makes the app feel unreliable.
   """
    ```

1. コメント **Create a sequential orchestration (順次オーケストレーションを作成する)** の下に次のコードを追加し、応答コールバックを使用して順次オーケストレーションを定義します。

    ```python
   # Create a sequential orchestration
   sequential_orchestration = SequentialOrchestration(
       members=get_agents(),
       agent_response_callback=agent_response_callback,
   )
    ```

    `agent_response_callback` を使用すると、オーケストレーション中に各エージェントからの応答を表示できます。

1. コメント **Create a runtime and start it (ランタイムを作成して開始する)** の下に次のコードを追加します。

    ```python
   # Create a runtime and start it
   runtime = InProcessRuntime()
   runtime.start()
    ```

1. コメント **Invoke the orchestration with a task and the runtime (タスクとランタイムでオーケストレーションを起動する)** の下に次のコードを追加します。

    ```python
   # Invoke the orchestration with a task and the runtime
   orchestration_result = await sequential_orchestration.invoke(
       task=task,
       runtime=runtime,
   )
    ```

1. コメント **Wait for the results (結果を待機する)** の下に次のコードを追加します。

    ```python
   # Wait for the results
   value = await orchestration_result.get(timeout=20)
   print(f"\n****** Task Input ******{task}")
   print(f"***** Final Result *****\n{value}")
    ```

    このコードで、オーケストレーションの結果を取得して表示します。 指定したタイムアウト内にオーケストレーションが完了しない場合は、タイムアウト例外がスローされます。

1. コメント **Stop the runtime when idle (アイドル時はランタイムを停止する)** を見つけて、次のコードを追加します。

    ```python
   # Stop the runtime when idle
   await runtime.stop_when_idle()
    ```

    処理が完了したら、ランタイムを停止してリソースをクリーンアップします。

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。 エラーを修正するためにコードを編集する必要がある場合は、開いたままにしてもかまいません。また、**CTRL+Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままでコード エディターを閉じることもできます。

### Azure にサインインしてアプリを実行する

これで、コードを実行し、AI エージェント間の共同作業を確認する準備ができました。

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    ```
   az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。

1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、Azure AI Foundry ハブを含むサブスクリプションを選択します。

1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```
   python agents.py
    ```

    次のような出力が表示されるはずです。

    ```output
    # SummarizerAgent
    App freezes during profile picture upload, preventing completion.
    # ClassifierAgent
    Negative
    # ActionAgent
    Escalate as a high-priority bug for the development team.

    ****** Task Input ******
    I tried updating my profile picture several times today, but the app kept freezing halfway through the process.
    I had to restart it three times, and in the end, the picture still wouldn't upload.
    It's really frustrating and makes the app feel unreliable.

    ***** Final Result *****
    Escalate as a high-priority bug for the development team.
    ```

1. 必要に応じて、次のようなさまざまなタスク入力を使用してコードの実行を試すこともできます。

    ```output
    I use the dashboard every day to monitor metrics, and it works well overall. But when I'm working late at night, the bright screen is really harsh on my eyes. If you added a dark mode option, it would make the experience much more comfortable.
    ```
    ```output
    I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
    ```

## まとめ

この演習では、セマンティック カーネル SDK を使用した順次オーケストレーションを練習し、複数のエージェントを 1 つの合理化されたワークフローに結合しました。 上出来

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。

1. ツール バーの **[リソース グループの削除]** を選びます。

1. リソース グループ名を入力し、削除することを確認します。
