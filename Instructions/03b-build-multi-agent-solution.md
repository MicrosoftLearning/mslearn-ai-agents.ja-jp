---
lab:
  title: Azure AI Foundry を使用してマルチエージェント ソリューションを開発する
  description: Azure AI Foundry Agent Service を使用して複数のエージェントを連携するように構成する方法を学習する
---

# マルチエージェント ソリューションの開発

この演習では、Azure AI Foundry Agent Service を使用して複数の AI エージェントを調整するプロジェクトを作成します。 チケット トリアージを支援する AI ソリューションを設計します。 接続されたエージェントは、チケットの優先度の評価、チーム割り当ての提案、チケットを完成させるために必要な作業レベルの決定を行います。 それでは始めましょう。

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

## AI エージェント クライアント アプリを作成する

これで、エージェントと指示を定義するクライアント アプリを作成する準備ができました。 GitHub リポジトリには、いくつかのコードが用意されています。

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
   cd ai-agents/Labfiles/03b-build-multi-agent-solution/Python
   ls -a -l
    ```

    指定されたファイルには、アプリケーション コードと構成設定用のファイルがあります。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_endpoint** プレースホルダーをプロジェクトのエンドポイント (Azure AI Foundry ポータルのプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを gpt-4o モデル デプロイに割り当てた名前 (既定では `gpt-4o`) に置き換えます。

1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### AI エージェントを作成する

これで、マルチエージェント ソリューション向けにエージェントを作成する準備ができました。 それでは始めましょう。

1. 次のコマンドを入力して、**agent_triage.py** ファイルを編集します。

    ```
   code agent_triage.py
    ```

1. 各エージェント名と指示の文字列が含まれていることに注意して、ファイル内のコードを確認します。

1. 「**Add references (参照を追加する)**」というコメントを見つけて以下のコードを追加し、必要なクラスをインポートします。

    ```python
   # Add references
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder, ToolSet, FunctionTool
   from azure.identity import DefaultAzureCredential
    ```

1. 環境変数からプロジェクトのエンドポイントとモデル名を読み込むコードが提供されていることに注目してください。

1. コメント **Connect to the agents client (エージェント クライアントに接続する)** を見つけて次のコードを追加し、プロジェクトに接続された AgentsClient を作成します。

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True, 
            exclude_managed_identity_credential=True
        ),
   )
    ```

    次に、AgentsClient を使用して複数のエージェントを作成するコードを追加します。各エージェントには、サポート チケットの処理で果たす特定の役割があります。

    > **ヒント**: 後続のコードを追加するときは、`using agents_client:` ステートメントの下で適切なレベルのインデントを維持してください。

1. コメント **Create an agent to prioritize support tickets (サポート チケットに優先順位を付けるエージェントを作成する)** を見つけて、次のコードを入力します (適切なレベルのインデントを保持するように注意してください)。

    ```python
   # Create an agent to prioritize support tickets
   priority_agent_name = "priority_agent"
   priority_agent_instructions = """
   Assess how urgent a ticket is based on its description.

   Respond with one of the following levels:
   - High: User-facing or blocking issues
   - Medium: Time-sensitive but not breaking anything
   - Low: Cosmetic or non-urgent tasks

   Only output the urgency level and a very brief explanation.
   """

   priority_agent = agents_client.create_agent(
        model=model_deployment,
        name=priority_agent_name,
        instructions=priority_agent_instructions
   )
    ```

1. コメント **Create an agent to assign tickets to the appropriate team (適切なチームにチケットを割り当てるエージェントを作成する)** を見つけて、次のコードを入力します。

    ```python
   # Create an agent to assign tickets to the appropriate team
   team_agent_name = "team_agent"
   team_agent_instructions = """
   Decide which team should own each ticket.

   Choose from the following teams:
   - Frontend
   - Backend
   - Infrastructure
   - Marketing

   Base your answer on the content of the ticket. Respond with the team name and a very brief explanation.
   """

   team_agent = agents_client.create_agent(
        model=model_deployment,
        name=team_agent_name,
        instructions=team_agent_instructions
   )
    ```

1. コメント **Create an agent to estimate effort for a support ticket (サポート チケットの作業量を見積もるエージェントを作成する)** を見つけて、次のコードを入力します。

    ```python
   # Create an agent to estimate effort for a support ticket
   effort_agent_name = "effort_agent"
   effort_agent_instructions = """
   Estimate how much work each ticket will require.

   Use the following scale:
   - Small: Can be completed in a day
   - Medium: 2-3 days of work
   - Large: Multi-day or cross-team effort

   Base your estimate on the complexity implied by the ticket. Respond with the effort level and a brief justification.
   """

   effort_agent = agents_client.create_agent(
        model=model_deployment,
        name=effort_agent_name,
        instructions=effort_agent_instructions
   )
    ```

    ここまでで、3 つのエージェントを作成しました。それぞれに、サポート チケットのトリアージにおいて特定の役割があります。 では、これらのエージェントごとに ConnectedAgentTool オブジェクトを作成し、他のエージェントが使用できるようにしましょう。

1. コメント **Create a connected agent tool for the priority agent (サポート エージェントの接続済みエージェント ツールを作成する)** を見つけて、次のコードを追加します。

    ```python
   # Create connected agent tools for the support agents
   priority_agent_tool = ConnectedAgentTool(
        id=priority_agent.id, 
        name=priority_agent_name, 
        description="Assess the priority of a ticket"
   )
    
   team_agent_tool = ConnectedAgentTool(
        id=team_agent.id, 
        name=team_agent_name, 
        description="Determines which team should take the ticket"
   )
    
   effort_agent_tool = ConnectedAgentTool(
        id=effort_agent.id, 
        name=effort_agent_name, 
        description="Determines the effort required to complete the ticket"
   )
    ```

    これで、必要に応じて接続済みエージェントを使用して、チケットのトリアージ プロセスを調整するプライマリ エージェントを作成する準備ができました。

1. コメント **Create an agent to triage support ticket processing by using connected agents (接続済みエージェントを使用してサポート チケット処理をトリアージするためのエージェントを作成する)** を見つけて、次のコードを入力します。

    ```python
   # Create an agent to triage support ticket processing by using connected agents
   triage_agent_name = "triage-agent"
   triage_agent_instructions = """
   Triage the given ticket. Use the connected tools to determine the ticket's priority, 
   which team it should be assigned to, and how much effort it may take.
   """

   triage_agent = agents_client.create_agent(
        model=model_deployment,
        name=triage_agent_name,
        instructions=triage_agent_instructions,
        tools=[
            priority_agent_tool.definitions[0],
            team_agent_tool.definitions[0],
            effort_agent_tool.definitions[0]
        ]
   )
    ```

    プライマリ エージェントを定義したので、これにプロンプトを送信し、他のエージェントを使用してサポートの問題をトリアージすることができます。

1. コメント **Use the agents to triage a support issue (エージェントを使用してサポートの問題をトリアージする)** を見つけて、次のコードを入力します。

    ```python
   # Use the agents to triage a support issue
   print("Creating agent thread.")
   thread = agents_client.threads.create()  

   # Create the ticket prompt
   prompt = input("\nWhat's the support problem you need to resolve?: ")
    
   # Send a prompt to the agent
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=prompt,
   )   
    
   # Run the thread usng the primary agent
   print("\nProcessing agent thread. Please wait.")
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=triage_agent.id)
        
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")

   # Fetch and display messages
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
   
    ```

1. コメント **Cleain up (クリーンアップする)** を見つけて、不要になったエージェントを削除するための次のコードを入力します。

    ```python
   # Clean up
   print("Cleaning up agents:")
   agents_client.delete_agent(triage_agent.id)
   print("Deleted triage agent.")
   agents_client.delete_agent(priority_agent.id)
   print("Deleted priority agent.")
   agents_client.delete_agent(team_agent.id)
   print("Deleted team agent.")
   agents_client.delete_agent(effort_agent.id)
   print("Deleted effort agent.")
    ```
    

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
   python agent_triage.py
    ```

1. `Users can't reset their password from the mobile app.` のようなプロンプトを入力します。

    エージェントがプロンプトを処理すると、次のような出力が表示されるはずです。

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    Users can't reset their password from the mobile app.

    MessageRole.AGENT:
    ### Ticket Assessment

    - **Priority:** High — This issue blocks users from resetting their passwords, limiting access to their accounts.
    - **Assigned Team:** Frontend Team — The problem lies in the mobile app's user interface or functionality.
    - **Effort Required:** Medium — Resolving this problem involves identifying the root cause, potentially updating the mobile app functionality, reviewing API/backend integration, and testing to ensure compatibility across Android/iOS platforms.

    Cleaning up agents:
    Deleted triage agent.
    Deleted priority agent.
    Deleted team agent.
    Deleted effort agent.
    ```

    別のシナリオを使用してプロンプトを変更し、エージェントがどのように共同作業を行うかを確認できます。 たとえば、「検索エンドポイントからときどき発生する 502 エラーを調査する」などです。

## クリーンアップ

Azure AI サービスを用いた演習が完了したら、不要な Azure コストが発生しないように、演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。

1. ツール バーの **[リソース グループの削除]** を選びます。

1. リソース グループ名を入力し、削除することを確認します。
