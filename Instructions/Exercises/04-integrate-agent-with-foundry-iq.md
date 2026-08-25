---
lab:
  title: AI エージェントと Foundry IQ を統合する
  description: Azure AI Agent サービスを使い、Foundry IQ を使ってナレッジ ベースを検索するエージェントを開発します。
  level: 300
  duration: 45
  islab: true
  status: released
---

# AI エージェントと Foundry IQ を統合する

この演習では、Microsoft Foundry ポータルを使い、Foundry IQ と統合してナレッジ ベースの情報の検索と取得を行うエージェントを作成します。 検索リソースを作成し、サンプル データを使ってナレッジ ベースを構成し、ポータルでエージェントを構築した後、Visual Studio Code からそれに接続してプログラムで対話します。

> **ヒント**: この演習で使われるコードは、Microsoft Foundry SDK for Python が基になっています。 Microsoft .NET、JavaScript、Java 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、[Microsoft Foundry SDK クライアント ライブラリ](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)に関するページを参照してください。

この演習の所要時間は約 **45** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

この演習を開始するには、以下のものが必要です。

- [Azure サブスクリプション](https://azure.microsoft.com/free/): AI リソース作成のためのアクセス許可が付与されていること
- ローカル コンピューターにインストールされている [Visual Studio Code](https://code.visualstudio.com/)
- [Python 3.13](https://www.python.org/downloads/) がインストールされていること
- ローカル コンピューターにインストールされている [Git](https://git-scm.com/downloads)
- Microsoft Foundry ポータルと Python プログラミングに関する基本的な知識

> \* Python 3.14 はまだサポートされていません。一部の依存関係には 3.14 ビルドが含まれていません。 このラボは Python 3.13.12 でテストされました。

## Foundry プロジェクトを作成する

まず、新しい Foundry エクスペリエンスを使って Foundry プロジェクトを作成しましょう。

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインしたときに開かれたヒントまたはクイック スタート ペインを閉じます。

    > **重要**: 更新されたユーザー インターフェイスを使うには、このラボで **[新しい Foundry]** トグルが "オン" になっていることを確認します。**

1. **[新しい Foundry]** に切り替えると、プロジェクトを選択するように求められます。 ドロップダウンで **[新しいプロジェクトの作成]** を選びます。
1. **[プロジェクトの作成]** ダイアログで、プロジェクトの有効な名前 (例: *agent-iq-lab*) を入力します。
1. プロジェクトで次の設定を確認または構成します。
    - **Foundry リソース**: *新しい Foundry リソースを作成するか、既存のリソースを選択します*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **[場所]**: *使用できるリージョンを選択する*\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。 これには数分かかることがあります。
1. プロジェクトが作成されると、プロジェクトのホーム ページが表示されます。

## エージェントを作成する

1. ホーム ページで **[ビルド]** タブを選択し、**[エージェント]** タブで **[エージェントの作成]** を選択します。
1. エージェントを作成するときは、`product-expert-agent` のようなわかりやすい名前を付けましょう。

エージェントの作成時には、既定のモデル (例: `gpt-5`) がデプロイされます。 エージェントが作成されると、その既定のモデルが自動的に選ばれたエージェント プレイグラウンドが表示されます。

## データと Foundry IQ を構成する

次に、Foundry IQ を使ってナレッジ ベースを検索するエージェントを構成します。

1. まず、エージェントに次のように指示します。

    ```
   You are a helpful AI assistant for Contoso, specializing in outdoor camping and hiking products. 
   You must ALWAYS search the knowledge base to answer questions about our products or product 
   catalog. Provide detailed, accurate information and always cite your sources.
   If you don't find relevant information in the knowledge base, say so clearly.
    ```

1. **[保存]** を選んで、現在のエージェント構成を保存します。
1. 次に、**[ナレッジ]** セクションで **[追加]** ドロップダウンを展開して、**[Foundry IQ に接続する]** を選びます。
1. Foundry IQ のセットアップ ウィンドウで **[AI Search リソースに接続する]** を選択してから **[新しいリソースの作成]** を選ぶと、リソースを作成するためのダイアログが開きます。
1. 既定の設定で検索リソースを作成します。
    - **リソース名**: *グローバルに一意の名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]**: プロジェクトと同じリソース グループを使います**
    - **[リージョン]**: プロジェクトと同じ場所**
    - **価格レベル**: [Free] *(利用できる場合)、そうでない場合は [Basic] を選択します*
    - **Foundry IQ ナレッジベースの機能**: 来月まで一時停止

    > **注**: ここでリソースを作成する際に問題が発生した場合は、フォーム下部のリンクを選択して Azure portal から作成してください。

次に、Foundry IQ を使って接続するサンプルの製品情報ドキュメントをアップロードします。

1. 新しいブラウザー タブを開き、`https://github.com/MicrosoftLearning/mslearn-ai-agents/raw/main/Labfiles/04-integrate-agent-with-foundry-iq/data/contoso-products.zip` に移動して、サンプルの製品情報ファイルをダウンロードします
1. zip からファイルを抽出します。Contoso の製品の詳細を示す 3 つの PDF があるはずです。
1. 新しいタブを開いて Azure portal (`https://portal.azure.com`) に移動します。 上部の検索バーで「**ストレージ アカウント**」を検索し、サービス セクションから **[ストレージ アカウント]** を選択します。
1. 次の設定でストレージ アカウントを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]**: プロジェクトと同じリソース グループを使います**
    - **ストレージ アカウント名**:一意のストレージ アカウント名**
    - **[リージョン]**: プロジェクトと同じ場所**
    - **プライマリ サービス**: *Azure Blob Storage または Azure Data Lake Storage*
    - **パフォーマンス**: "標準"**
    - **冗長性**: *"ローカル冗長ストレージ (LRS)"*
1. 作成されたら、作成したストレージ アカウントに移動し、上部のバーから **[アップロード]** を選びます。
1. **[BLOB のアップロード]** ブレードで、`contosoproducts` という名前の新しいコンテナーを作成します。
1. zip ファイルから抽出されたファイルを参照し、3 つの PDF ファイルをすべて選んで、**[アップロード]** を選びます。
1. ファイルがアップロードされたら、自身で作成した検索サービスの画面に移動します。
1. 左側のペインの **[セキュリティとネットワーク]** > **[キー]** で、[API アクセス制御] の **[両方]** を選択し、選択を確定します。 完了したら、Azure portal のタブを開いたまま Foundry ポータルのタブに戻り、ページを最新の情報に更新します。
1. **[ナレッジ]** ページにいることを確認して、**[ナレッジ ベースの作成]** を選択し、ナレッジ ソースとして **[Azure Blob Storage]** を選択して、**[接続]** を選択します。
1. 次の設定でナレッジ ソースを構成します。
    - **名前**: `ks-contosoproducts`
    - **説明**: `Contoso product catalog items`
    - **ストレージ アカウント名**: *ストレージ アカウントを選択します。*
    - **[コンテナー名]**: `contosoproducts`
    - **[認証の種類]**:*API キー*
    - **[コンテンツ抽出モード]**: [最小]**
    - **[埋め込みモデル]**: 使用できるデプロイ済みモデルを選びます (例: text-embedding-3-small)**
    - **[チャット完了モデル]**: *使用可能なデプロイ済みモデル (gpt-5 の可能性があります) を選択します*
1. **［作成］** を選択します
1. ナレッジ ベース作成ページで、**[チャット入力候補モデル]** ドロップダウンから `gpt-5` モデルを選択し、残りのフィールドは既定値のままにします。
1. **[ナレッジ ベースの保存]** を選んでから、ブラウザーを更新して、ナレッジ ソースの状態が "アクティブ" であることを確認します。** まだそのようになっていない場合は、そうなるまで少し待ってからページを更新します。
1. [戻る] ボタンを選択して **[ナレッジ]** ページに戻り、*[接続]* ドロップダウンの横にある **[管理]** リンクを選択します。
1. **[接続されたリソース]** を下スクロールすると、検索サービスが表示されるはずです。 その行を選択し、**[認証]** セクションを見つけます。
1. **[キー認証]** を選択し、**[認証の編集]** を選択します。
1. ダイアログを開いたまま、Azure portal タブに戻ると、検索サービスの **[キー]** ページがまだ開いているはずです。 そのキーの 1 つを Foundry のダイアログにコピーし、**[保存]** を選択します。

Foundry IQ の設定はこれで完了するはずです。

## プレイグラウンドでエージェントをテストする

コードから接続する前に、ポータルのプレイグラウンドでエージェントをテストします。

1. **[ビルド]** > **[エージェント]** ページでエージェントに戻り、作成したエージェントを選択します。
2. エージェントのページでは、プレイグラウンドのタブが選択されているはずです。 ナレッジのセクションを見つけて Foundry IQ を追加し、作成した接続とナレッジ ベースを選択します。
1. 次のテスト クエリを試して、エージェントがナレッジ ベースから情報を取得できることを確認します。
    - `What types of tents does Contoso offer?`
    - `Tell me about which backpacks are available in XL.`
    - `What camping accessories are available?`

1. 応答を確認して次のことを理解します。
    - エージェントは、ナレッジ ベースから特定の情報を提供します
    - ソース ドキュメントの引用または参照が含まれる場合があります
    - エージェントは製品情報に集中し続けます

1. **[プレビュー エージェント]** でエージェントと対話して、より優れた Web アプリ エクスペリエンスを実現することもできます。

1. エージェントの詳細ページで、次の情報を見つけてメモ帳にコピーします (これらは後で必要になります)。
    - **エージェント名**: これは、ご自分で作成した名前です (`product-expert-agent`)
    - **プロジェクト エンドポイント**: プロジェクトの設定またはホーム ページにあります

### ツール呼び出しの承認を求めるようにエージェントを構成する

ポータルでエージェントを作成すると、Foundry IQ (ナレッジ) ツールは既定で承認を求める**ことなく**実行されます。 確実にアプリで各ナレッジ ベースの検索をレビューして制御できるようにするには、Foundry Toolkit for VS Code 拡張機能でツールを使用する前に、承認を求めるようにエージェントを変更します。

> **注**: Foundry ポータルでは現在、この承認動作を変更するための設定が公開されていないため、代わりに Foundry Toolkit 拡張機能から構成します。

1. Visual Studio Code で、左側のペインから **[拡張機能]** を選択し (または、**Ctrl + Shift + X** キーを押し)、Microsoft の `Foundry Toolkit for VS Code` 拡張機能をマーケットプレースで検索し、**[インストール]** を選びます (まだインストールされていない場合)。

    > **注**: 拡張機能は現在、**Foundry Toolkit** として登録されていますが、一部の VS Code のラベル、コマンド、または以前のスクリーンショットでは、引き続き **AI Toolkit** と表示されている場合があります。 このラボでは、これらの名前が同じ拡張機能エクスペリエンスを表しているものとして扱います。

1. サイドバーの **[Foundry Toolkit]** アイコンを選択し、メッセージが表示されたら Azure アカウントにサインインします。
   
    > **注**: Foundry Toolkit 拡張機能でサインインできない場合は、Azure 拡張機能を選択する必要があります。 そこでサインインし、Foundry Toolkit に戻ってリソースにアクセスします。

1. **[Microsoft Foundry リソース]** で、**[既定のプロジェクトの設定]** を選び、前に作成したプロジェクトを選択します。
1. [プロジェクト] セクションを展開します。 **[プロンプト エージェント]** で、`product-expert-agent` エージェントを選択して **[エージェント ビルダー]** ウィンドウを開きます。
1. **[ツール]** セクションには、`kb-knowledgebase` プレフィックスの後に一意の ID が付いたツール名 (例: `kb-knowledgebase677-7w5fj`) が既に表示されているはずです。 これは Foundry IQ のナレッジ ベース ツールで、ポータルで Foundry IQ を接続したときに自動的に追加されたものです。

    > **注**: エージェントには複数のツールが一覧表示される場合があります。 既定では、Foundry ポータルが新しいエージェントに **Web 検索**ツールを追加し、スタンドアロンの **Azure AI 検索** ツールも表示される場合があります。 エージェントはナレッジ ベース検索時に実際に `kb-knowledgebase...` ツールを呼び出すので、他のツールで承認を設定しても影響はありません。

1. `kb-knowledgebase...` ツールの省略記号 (**[...]**) アイコンを選択してから **[すべてのツールで承認を求める]** を選択し、プロンプトが表示されたら変更を保存してください。

これで、エージェントは Foundry IQ を使用してナレッジ ベースを検索するたびに承認を求めるようになり、次に完了するクライアント アプリによって処理されます。

## アプリからエージェントに接続する

ここでは、プログラムを使ってエージェントと対話する Python アプリケーションを作成します。 すぐに始められるように、スターター ファイルが GitHub リポジトリに用意されています。

### Visual Studio Code でアプリの開発準備をする

次に、Visual Studio Code を使用してアプリを開発してみましょう。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

1. Visual Studio Code を開始し、コマンド パレットを開きます (Shift+Ctrl+P)。 次に、**Git: Clone** コマンドを検索して実行し、`https://github.com/MicrosoftLearning/mslearn-ai-agents` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
1. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。

    > **注**: Visual Studio Code に、開いているコードを信頼するかどうかを求めるポップアップ メッセージが表示された場合は、**[はい、作成者を信頼します]** オプションをクリックして続行します。

1. 必要に応じて、リポジトリ内の Python コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます (ダイアログが表示された場合)。

    > **注**: ビルドとデバッグに必要なアセットをインストールするように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

1. **エクスプローラー** ペインで、**Labfiles/04-integrate-agent-with-foundry-iq/Python** フォルダーを展開します。

    提供されたファイルには、アプリケーション コード、構成設定、エージェント クライアント スタート コードが含まれます。

### アプリケーション設定を構成する

1. Visual Studio Code で、**Labfiles/04-integrate-agent-with-foundry-iq/Python** フォルダーの **.env** 構成ファイルを開きます。
1. コード ファイルで、**your_project_endpoint** プレースホルダーを (Foundry ポータルでプロジェクトの **[ホーム]** ページからコピーした) プロジェクトのエンドポイントに置き換え、AGENT_NAME 変数がエージェント名 (*product-expert-agent* になっているはずです) に設定されていることを確認します。
1. プレースホルダーを置き換えた後、ファイルを保存します。

### エージェント クライアント コードを完成させる

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 コメントのインデント レベルをガイドとして利用します。

1. Visual Studio Code で、**Labfiles/04-integrate-agent-with-foundry-iq/Python** フォルダーの **agent_client.py** コード ファイルを開きます。
1. 次のような提供されているスタート コードを確認します。
    - import ステートメントと構成の読み込み
    - `send_message_to_agent()` 関数の構造
    - `display_conversation_history()` 関数
    - メイン プログラム ループ

1. 最初の **TODO** コメントを見つけ、次のコードを追加してプロジェクトに接続し、OpenAI クライアントを取得し、エージェントを取得して、新しい会話を作成します。

    > **ヒント**: インデント レベルを正しく維持するように注意してください。

    ```python
   # Connect to the project and agent
   credential = DefaultAzureCredential(
       exclude_environment_credential=True,
       exclude_managed_identity_credential=True
   )
   project_client = AIProjectClient(
       credential=credential,
       endpoint=project_endpoint
   )

   # Get the OpenAI client
   openai_client = project_client.get_openai_client()

   # Get the agent
   agent = project_client.agents.get(agent_name=agent_name)
   print(f"Connected to agent: {agent.name} (id: {agent.id})\n")

   # Create a new conversation
   conversation = openai_client.conversations.create(items=[])
   print(f"Created conversation (id: {conversation.id})\n")
    ```

1. `send_message_to_agent()` 関数内で 2 番目の **TODO** コメントを見つけ、MCP 承認要求などのメッセージを送信して応答を処理する次のコードを追加します。

    ```python
   # Add user message to the conversation
   openai_client.conversations.items.create(
       conversation_id=conversation.id,
       items=[{"type": "message", "role": "user", "content": user_message}],
   )

   # Store in conversation history (client-side)
   conversation_history.append({
       "role": "user",
       "content": user_message
   })

   # Create a response using the agent
   response = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       input=""
   )

   # Loop until a response has no pending approval requests (zero, one, or many)
   while True:
       approval_requests = [
           item for item in (getattr(response, "output", None) or [])
           if getattr(item, "type", None) == "mcp_approval_request"
       ]

       if not approval_requests:
           break

       approval_items = []
       for approval_request in approval_requests:
           print(f"[Approval required for: {approval_request.name}]\n")
           print(f"Server: {approval_request.server_label}")

           # Show the tool call arguments for transparency
           import json
           try:
               args = json.loads(approval_request.arguments)
               print(f"Arguments: {json.dumps(args, indent=2)}\n")
           except Exception:
               print(f"Arguments: {approval_request.arguments}\n")

           approval_input = input("Approve this action? (yes/no): ").strip().lower()
           approved = approval_input in ['yes', 'y']
           print("Approving action...\n" if approved else "Action denied.\n")

           approval_items.append({
               "type": "mcp_approval_response",
               "approval_request_id": approval_request.id,
               "approve": approved
           })

       # Send the approval decisions and fetch the next response
       openai_client.conversations.items.create(
           conversation_id=conversation.id,
           items=approval_items
       )

       response = openai_client.responses.create(
           conversation=conversation.id,
           extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
           input=""
       )

    ```

    > **注**: エージェントが必ずしも承認を要求するわけではなく、同じターンで複数のツール呼び出しの承認を求めることもあります。 どちらのケースも、`approval_requests` が空になるまでループすることで正しく処理できます。

1. コードを追加したら、ファイルを保存します。

1. コードが会話 API を使ってエージェントとの対話を管理するようになっていることを確認します。次のようなものです。
    - 会話が作成され、その ID によって追跡されます
    - `conversations.items.create()` を使ってユーザー メッセージが会話に追加されます
    - `responses.create()` とエージェントの参照を使って応答が生成されます
    - **MCP 承認処理**: エージェントが Foundry IQ にアクセスする必要がある場合、応答出力で 1 つまたは複数の `mcp_approval_request` 項目を返すことで承認を要求します
    - コードはループし、エージェントが未解決の承認要求 (そもそも承認が必要なかったケースも含む) がなくなったという応答を返すまで、保留中の要求をそれぞれ承認または拒否するように促します
    - 承認または拒否の後、`mcp_approval_response` が会話に追加され、新しい応答が生成されます
    - エージェントは、ユーザーの承認の決定に基づいて Foundry IQ から情報を取得します

## 統合をテストします

次に、アプリケーションを実行し、ナレッジ ベースから情報を取得するエージェントの機能をテストします。

1. Visual Studio Code で、**Labfiles/04-integrate-agent-with-foundry-iq/Python** フォルダーを右クリックし、**[統合ターミナルで開く]** を選択してそのフォルダーの統合ターミナルを開きます。
1. まず、仮想環境を作成し、依存関係をインストールします。

    ```
   python -m venv labenv
   ./labenv/Scripts/activate
   pip install -r requirements.txt
    ```

1. ターミナル ペインで次のコマンドを入力して Azure にサインインします。

    ```
   az login
    ```

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。

1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、メッセージが表示されたら Foundry リソースを含むサブスクリプションを選択します。

1. ターミナル ペインでアプリケーションを実行します。

    ```
   python agent_client.py
    ```

1. アプリケーションが開始したら、次のクエリを使ってエージェントをテストします。

    **クエリ 1 - 製品カテゴリ:**

    ```
   What types of outdoor products does Contoso offer?
    ```

    承認を求められたら、「**yes**」と入力して、エージェントがナレッジ ベースを検索するのを許可します。 エージェントがナレッジ ベース内の複数のドキュメントから情報をどのように取得するかを観察します。

    **クエリ 2 - 特定の製品の詳細:**

    ```
   Tell me about the weatherproof features of your tents.
    ```

    要求を承認し、エージェントがテント カタログから特定の詳細をどのように提供するかを注視します。

    **クエリ 3 - 製品の比較:**

    ```
   What's the difference between your daypacks and expedition backpacks?
    ```

    要求を承認し、エージェントがバックパック ガイドからの情報をどのように合成するかを確認します。

    **クエリ 4 - アクセサリとアドオン:**

    ```
   What camping accessories would you recommend for a weekend hiking trip?
    ```

    要求を承認し、ナレッジ ベースに基づいて推奨事項を提供するエージェントの機能を観察します。

    **クエリ 5 - フォローアップの質問:**

    ```
   How much do those items typically cost?
    ```

    エージェントが前のクエリから会話コンテキストをどのように維持するかに注目します。

1. 「`history`」と入力して、完全な会話履歴を表示します。

1. テストを終えたら、「`quit`」と入力します。

### 結果の確認

エージェントの応答の次の側面について考えてみましょう。

- **MCP 承認フロー**: エージェントは、ナレッジ ベースにアクセスする必要があるたびに承認を要求するので、外部ツールの使用を制御できます
- **精度**: エージェントは、ナレッジ ベースのドキュメントから直接情報を提供します
- **引用**: エージェントは、ソースの参照またはドキュメント ID を含めることができます
- **コンテキストの認識**:エージェントは、会話での以前のメッセージを覚えています
- **典拠**: エージェントは、ナレッジ ベースに関連情報が見つからない場合に示します
- **エラー処理**: アプリケーションはエラーと接続の問題を適切に処理します

## まとめ

この演習では、以下のことを行います。

- 新しい Foundry UI を使って Foundry プロジェクトとエージェントを作成しました
- 製品情報ドキュメントを含むナレッジ ベースを構築しました
- Foundry IQ を有効にしてポータルでエージェントを構成しました
- Python SDK を使って Visual Studio Code からエージェントに接続しました
- クライアント アプリケーションに MCP 承認処理、会話履歴、エラー処理を実装しました
- 外部ツールのアクセスについてユーザーが制御する承認を使って、ナレッジ ベースから情報を取得して合成するエージェントの機能をテストしました

これにより、AI エージェントと Foundry IQ を統合し、会話のコンテキストを維持しながらエンタープライズ ナレッジ ベースで情報を検索して取得できるインテリジェントなアプリケーションを作成する方法がわかります。

## クリーンアップ

Azure AI Agent サービスと Foundry IQ を調べる演習が済んだら、不要な Azure コストが発生しないよう、この演習で作成したリソースを削除する必要があります。

1. Web ブラウザーで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開きます。
1. Foundry リソースと AI Search リソースを含むリソース グループに移動します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
