---
lab:
  title: セマンティック カーネルを使用してマルチエージェント ソリューションを開発する
  description: Semantic Kernel SDK を使用して共同作業するよう複数のエージェントを構成する方法について説明します
---

# マルチエージェント ソリューションの開発

この演習では、Semantic Kernel SDK を使用して 2 つの AI エージェントを調整するプロジェクトを作成します。 *インシデント マネージャー* エージェントは、サービス ログ ファイルを分析して問題がないか調べます。 問題が見つかった場合、インシデント マネージャーは解決アクションを推奨します。推奨を受け取った *DevOps アシスタント* エージェントは修正関数を呼び出して解決を実行します。 その後、インシデント マネージャー エージェントは、更新されたログを確認して、解決が成功したことを確認します。

この演習では、4 つのサンプル ログ ファイルが提供されます。 DevOps アシスタント エージェント コードは、サンプル ログ ファイルのみをいくつかのログ メッセージ例で更新します。

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
    - **リージョン**: ***AI サービスでサポートされている場所を選択します***\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択し、選んだ gpt-4 モデル デプロイを含むプロジェクトが作成されるまで待ちます。
1. プロジェクトが作成されると、チャット プレイグラウンドが自動的に開きます。

    > **注**: このモデルの既定の TPM 設定は、この演習には低すぎる場合があります。 TPM を低めに設定しておくことで、サブスクリプション内のクォータが過剰に消費されるのを防ぐことができます。 

1. 左側のナビゲーション ウィンドウで、**[モデルとエンドポイント]** を選択し、**[gpt-4o]** デプロイを選択します。

1. **[編集]** を選択して、**[1 分あたりのレート制限の トークン数]** を増やします。　

   > **注**: この演習で使用するデータには、40,000 TPM あれば十分でしょう。 使用可能なクォータがこれより低い場合は、演習を完了できますが、レート制限を超えた場合は、少し待ってからプロンプトを再送信する必要がある場合があります。

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

1. コード ファイルで、**your_project_endpoint** プレースホルダーをプロジェクトのエンドポイント (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを gpt-4o モデル デプロイに割り当てた名前に置き換えます。

1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### AI エージェントを作成する

これで、マルチエージェント ソリューション向けにエージェントを作成する準備ができました。 それでは始めましょう。

1. 次のコマンドを入力して、**agent_chat.py** ファイルを編集します。

    ```
   code agent_chat.py
    ```

1. ファイル内のコードを確認します。以下が含まれているのを確認すること。
    - 2 つのエージェントの名前と命令を定義する定数。
    - **main**関数。マルチエージェント ソリューションを実装するほとんどのコードが追加されます。
    - **SelectionStrategy** クラス。会話のターンごとにどのエージェントを選択するかを決定するのに必要なロジックの実装に使用します。
    - **ApprovalTerminationStrategy** クラス。このクラスは、会話を終了するタイミングを決定するのに必要なロジックの実装に使用します。
    - devops 操作を実行する関数を含む **DevopsPlugin** クラス。
    - ログ ファイルの読み取りと書き込みを行う関数を含む **LogFilePlugin** クラス。

    まず、*インシデント マネージャー* エージェントを作成します。このエージェントは、サービス ログ ファイルの分析、潜在的な問題の特定、解決アクションの推奨、および必要に応じて問題のエスカレートを行います。

1. **INCIDENT_MANAGER_INSTRUCTIONS** 文字列をメモします。 エージェント向け命令は以下の通りです。

1. **main** 関数で、「**Create the incident manager agent on the Azure AI agent service (Azure AI エージェント サービスでインシデント マネージャー エージェントを作成する)**」コメントを見つけ、以下のコードを追加して Azure AI エージェントを作成します。

    ```python
   # Create the incident manager agent on the Azure AI agent service
   incident_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=INCIDENT_MANAGER,
        instructions=INCIDENT_MANAGER_INSTRUCTIONS
   )
    ```

    このコードにより、お使いのAzure AI Project クライアント上にエージェント定義が作成されます。

1. 「**Create a Semantic Kernel agent for the Azure AI incident manager agent (Azure AI インシデント マネージャー エージェント向け Semantic Kernel エージェンを作成する)**」コメントを見つけ、以下のコードを追加して、Azure AI エージェントの定義に基づく Semantic Kernel エージェントを作成します。

    ```python
   # Create a Semantic Kernel agent for the Azure AI incident manager agent
   agent_incident = AzureAIAgent(
        client=client,
        definition=incident_agent_definition,
        plugins=[LogFilePlugin()]
   )
    ```

    このコードは、**LogFilePlugin** にアクセスできる Semantic Kernel エージェントを作成します。 このプラグインを使用すると、エージェントはログ ファイルの内容を読み取ることができます。

    次に、問題に対応し、DevOps 操作を実行して解決する 2 つ目のエージェントを作成しましょう。

1. コード ファイルの先頭で、**DEVOPS_ASSISTANT_INSTRUCTIONS** 文字列を確認します。 以下は、新しい DevOps アシスタント エージェントに提供する命令です。

1. 「**Create the devops agent on the Azure AI agent service (Azure AI エージェント サービスで devops エージェントを作成する)**」コメントを見つけ、以下のコードを追加して Azure AI エージェント定義を作成します。
    
    ```python
   # Create the devops agent on the Azure AI agent service
   devops_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=DEVOPS_ASSISTANT,
        instructions=DEVOPS_ASSISTANT_INSTRUCTIONS,
   )
    ```

1. 「**Create a Semantic Kernel agent for the devops Azure AI agent (devops Azure AI エージェント向け Semantic Kernel エージェントを作成する)**」コメントを見つけ、以下のコードを追加して、Azure AI エージェントの定義に基づく Semantic Kernel エージェントを作成します。
    
    ```python
   # Create a Semantic Kernel agent for the devops Azure AI agent
   agent_devops = AzureAIAgent(
        client=client,
        definition=devops_agent_definition,
        plugins=[DevopsPlugin()]
   )
    ```

    **DevopsPlugin**を使用すると、エージェントはサービスの再起動やトランザクションのロールバックなどの devops タスクをシミュレートできます。

### グループ チャット戦略を定義する

次に、会話内で次のターンを実行するエージェントの選択と、会話を終了するタイミングの決定に使用するロジックを指定する必要があります。

最初に、次のターンを実行するエージェントを決定する **SelectionStrategy** の定義から始めます。

1. **SelectionStrategy** クラス (**main** 関数の下) で、「**Select the next agent that should take the next turn in the chat (チャット内で次のターンを実行するエージェントを選択する)**」コメントを見つけ、以下のコードを追加して選択関数を定義します。

    ```python
   # Select the next agent that should take the next turn in the chat
   async def select_agent(self, agents, history):
        """"Check which agent should take the next turn in the chat."""

        # The Incident Manager should go after the User or the Devops Assistant
        if (history[-1].name == DEVOPS_ASSISTANT or history[-1].role == AuthorRole.USER):
            agent_name = INCIDENT_MANAGER
            return next((agent for agent in agents if agent.name == agent_name), None)
        
        # Otherwise it is the Devops Assistant's turn
        return next((agent for agent in agents if agent.name == DEVOPS_ASSISTANT), None)
    ```

    このコードをターンごとに実行することで、チャット内でどのエージェントが最後に応答したかを確認し、次に応答するエージェントを決定します。

    次に、**ApprovalTerminationStrategy** クラスを実装して、目標が達成されたことを通知して会話を終了できるようにします。

1. **ApprovalTerminationStrategy** クラス内で「**End the chat if the agent has indicated there is no action needed (アクションが不要であることをエージェントが示した時はチャットを終了する)**」コメントを見つけ、以下のコードを追加して終了関数を定義します。

    ```python
   # End the chat if the agent has indicated there is no action needed
   async def should_agent_terminate(self, agent, history):
        """Check if the agent should terminate."""
        return "no action needed" in history[-1].content.lower()
    ```

    カーネルは、エージェントの応答の後にこの関数を呼び出して、終了条件が満たされているかどうかを判断します。 ここでインシデント マネージャーが「アクションは不要です」と応答すると、目標が達成されたと見なされます。 このフレーズは、インシデント マネージャー エージェントの命令文内で定義されています。

### グループ チャットを実装する

これで 2 つのエージェントを構築するとともに、各エージェントにターンを実行させかつ必要に応じてチャットを終了する戦略を用意できたので、グループ チャットを実装することができます。

1. main 関数に戻り、「**Add the agents to a group chat with a custom termination and selection strategy (終了と選択のカスタム戦略を使用してエージェントをグループ チャットに追加する)**」というコメントを見つけて、次のコードを追加してグループ チャットを作成します。

    ```python
   # Add the agents to a group chat with a custom termination and selection strategy
   chat = AgentGroupChat(
        agents=[agent_incident, agent_devops],
        termination_strategy=ApprovalTerminationStrategy(
            agents=[agent_incident], 
            maximum_iterations=10, 
            automatic_reset=True
        ),
        selection_strategy=SelectionStrategy(agents=[agent_incident,agent_devops]),      
   )
    ```

    このコードにより、インシデント マネージャー エージェントと devops エージェントを使用するエージェント グループ チャット オブジェクトを作成します。 また、チャットの終了と選択の戦略も定義します。 なお、**ApprovalTerminationStrategy** は、インシデント マネージャー エージェントにのみ関連付けられ、devops エージェントには関連付けられないことに留意してください。 これにより、インシデント マネージャー エージェントはチャットの終了を通知する役割を担います。 **SelectionStrategy** には、チャット内でターンを実行するすべてのエージェントが含まれます。

    終了したチャットは、自動リセット フラグにより自動的にクリアされます。 これによりエージェントは、多くの不要なトークンを使用するチャット履歴オブジェクトなしでファイルの分析を続行できます。 

1. 「**Append the current log file to the chat （現在のログ ファイルをチャットに追加する)**」というコメントを見つけて、次のコードを追加し、最後に読み取ったログ ファイルのテキストをチャットに追加します。

    ```python
   # Append the current log file to the chat
   await chat.add_chat_message(logfile_msg)
   print()
    ```

1. 「**Invoke a response from the agents (エージェントからの応答を呼び出す)**」コメントを見つけ、以下のコードを追加してグループ チャットを呼び出します。

    ```python
   # Invoke a response from the agents
   async for response in chat.invoke():
        if response is None or not response.name:
            continue
        print(f"{response.content}")
    ```

    これは、チャットをトリガーするコードです。 ログ ファイル テキストがメッセージとして追加されているため、どのエージェントがメッセージを読み取って応答するかが選択戦略により決定され、終了戦略の条件が満たされるか、イテレーションの最大数に達するまでエージェント間の会話が続行されます。

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
   python agent_chat.py
    ```

    次のような出力が表示されるはずです。

    ```output
    
    INCIDENT_MANAGER > /home/.../logs/log1.log | Restart service ServiceX
    DEVOPS_ASSISTANT > Service ServiceX restarted successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log2.log | Rollback transaction for transaction ID 987654.
    DEVOPS_ASSISTANT > Transaction rolled back successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log3.log | Increase quota.
    DEVOPS_ASSISTANT > Successfully increased quota.
    (continued)
    ```

    > **注**: アプリには、TPM レート制限超過リスクの軽減のために各ログ ファイルの処理の間に待機するコードと、問題が発生した場合の例外処理が含まれています。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

1. **logs** フォルダー内のログ ファイルが、DevopsAssistant からの解決操作メッセージで更新されていることを確認します。

    たとえば、log1.logには次のログ メッセージが追加されている必要があります。

    ```log
    [2025-02-27 12:43:38] ALERT  DevopsAssistant: Multiple failures detected in ServiceX. Restarting service.
    [2025-02-27 12:43:38] INFO  ServiceX: Restart initiated.
    [2025-02-27 12:43:38] INFO  ServiceX: Service restarted successfully.
    ```

## まとめ

この演習では、Azure AI エージェント サービスと Semantic Kernel SDK を使用して、問題を自動的に検出して解決策を適用する AI インシデント エージェントおよび devops エージェントを作成しました。 上出来

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。

1. ツール バーの **[リソース グループの削除]** を選びます。

1. リソース グループ名を入力し、削除することを確認します。
