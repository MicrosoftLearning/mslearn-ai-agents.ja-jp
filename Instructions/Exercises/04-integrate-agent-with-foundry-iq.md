---
lab:
  title: AI エージェントと Foundry IQ を統合する
  description: Azure AI Agent サービスを使い、Foundry IQ を使ってナレッジ ベースを検索するエージェントを開発します。
---

# AI エージェントと Foundry IQ を統合する

この演習では、Azure AI Foundry ポータルを使い、Foundry IQ と統合してナレッジ ベースの情報の検索と取得を行うエージェントを作成します。 検索リソースを作成し、サンプル データを使ってナレッジ ベースを構成し、ポータルでエージェントを構築した後、Visual Studio Code からそれに接続してプログラムで対話します。

> **ヒント**: この演習で使われるコードは、Microsoft Foundry SDK for Python が基になっています。 Microsoft .NET、JavaScript、Java 用の SDK を使用して、同様のソリューションを開発できます。 詳細については、[Microsoft Foundry SDK クライアント ライブラリ](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)に関するページを参照してください。

この演習の所要時間は約 **45** 分です。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

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

1. ホーム ページの **[ビルドの開始]** で、**[エージェントの作成]** を選びます。
1. エージェントの名前を指定します (例: `product-expert-agent`)。
1. **［作成］** を選択します

エージェントの作成時には、既定のモデル (例: `gpt-4.1`) がデプロイされます。 エージェントが作成されると、その既定のモデルが自動的に選ばれたエージェント プレイグラウンドが表示されます。

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
1. Foundry IQ のセットアップ ウィンドウで **[AI Search リソースに接続する]** を選んでから **[新しいリソースの作成]** を選ぶと、新しいタブで Azure portal が開きます。
1. 次の設定で検索リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]**: プロジェクトと同じリソース グループを使います**
    - **サービス名**:グローバルに一意の名前**
    - **[場所]**: プロジェクトと同じ場所**
    - **価格レベル**: 利用できる場合は *[Free]*、そうでない場合は *[Basic]*

次に、Foundry IQ を使って接続するサンプルの製品情報ドキュメントをアップロードします。

1. 新しいブラウザー タブを開き、`https://github.com/MicrosoftLearning/mslearn-ai-agents/raw/main/Labfiles/09-integrate-agent-with-foundry-iq/data/contoso-products.zip` に移動して、サンプルの製品情報ファイルをダウンロードします
1. zip からファイルを抽出します。Contoso の製品の詳細を示す 3 つの PDF があるはずです。
1. Azure portal タブの上部の検索バーで "**ストレージ アカウント**" を検索し、サービス セクションから **[ストレージ アカウント]** を選びます。
1. 次の設定でストレージ アカウントを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]**: プロジェクトと同じリソース グループを使います**
    - **ストレージ アカウント名**:一意のストレージ アカウント名**
    - **[リージョン]**: プロジェクトと同じ場所**
    - **[優先ストレージの種類]**: *Azure Blob Storage または Azure Data Lake Storage Gen 2*
    - **パフォーマンス**: "標準"**
    - **冗長性**: *"ローカル冗長ストレージ (LRS)"*
1. 作成されたら、作成したストレージ アカウントに移動し、上部のバーから **[アップロード]** を選びます。
1. **[BLOB のアップロード]** ブレードで、`contosoproducts` という名前の新しいコンテナーを作成します。
1. zip ファイルから抽出されたファイルを参照し、3 つの PDF ファイルをすべて選んで、**[アップロード]** を選びます。
1. ファイルがアップロードされたら、Azure portal のタブを閉じて Microsoft Foundry の Foundry IQ ページに戻り、ページを更新します。
1. 検索サービスを選んで、**[接続]** をクリックします。
1. Foundry IQ のページで、**[ナレッジ ベースの作成]** を選び、ナレッジ ソースとして **[Azure Blob Storage]** を選んで、**[接続]** を選びます。
1. 次の設定でナレッジ ソースを構成します。
    - **名前**: `ks-contosoproducts`
    - **説明**: `Contoso product catalog items`
    - **ストレージ アカウント名**: *ストレージ アカウントを選択します。*
    - **[コンテナー名]**: `contosoproducts`
    - **[コンテンツ抽出モード]**: [最小]**
    - **[認証の種類]**:*API キー*
    - **[埋め込みモデルを含める]**: *オン*
    - **[埋め込みモデル]**: 使用できるデプロイ済みモデルを選びます (例: text-embedding-3-small)**
    - **[チャット入力候補モデル]**: 使用できるデプロイ済みモデルを選びます (例: gpt-4.1)**
1. **［作成］** を選択します
1. ナレッジ ベース作成ページで、**[チャット入力候補モデル]** ドロップダウンから `gpt-4.1` モデルを選び、残りのオプション フィールドはそのままにします。
1. **[ナレッジ ベースの保存]** を選んでから、ブラウザーを更新して、ナレッジ ソースの状態が "アクティブ" であることを確認します。** まだそのようになっていない場合は、そうなるまで少し待ってからページを更新します。
1. 右上の **[エージェントで使用する]** ドロップダウンを展開して、`product-expert-agent` を選びます。

## プレイグラウンドでエージェントをテストする

コードから接続する前に、ポータルのプレイグラウンドでエージェントをテストします。

1. エージェント ページでは、プレイグラウンド タブが選ばれ、ナレッジ セクションにナレッジ ベースの一覧が表示されているはずです。
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
    - **プロジェクト エンドポイント**: プロジェクトの設定または概要のページにあります

## クライアント アプリケーションからエージェントに接続する

ここでは、プログラムを使ってエージェントと対話する Python アプリケーションを作成します。 すぐに始められるように、スターター ファイルが GitHub リポジトリに用意されています。

### アプリケーション コードを含むリポジトリを複製する

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

1. 次のコマンドを入力して、作業ディレクトリをコード ファイルを含むフォルダーに変更し、すべてを一覧表示します。

    ```
   cd ai-agents/Labfiles/09-integrate-agent-with-foundry-iq/Python
   ls -a -l
    ```

    提供されたファイルには、アプリケーション コード、構成設定、エージェント クライアント スタート コードが含まれます。

### アプリケーション設定を構成する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

    >**注:** ライブラリのインストール中に表示される警告やエラー メッセージは無視しても構いません。

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_endpoint** プレースホルダーを (Foundry ポータルのプロジェクトの **[概要]** ページからコピーした) プロジェクトのエンドポイントに置き換え、AGENT_NAME 変数がエージェント名 (*product-expert-agent* になっているはずです) に設定されていることを確認します。
1. プレースホルダーを置き換えたら、**Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### エージェント クライアント コードを完成させる

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。 コメントのインデント レベルをガイドとして利用します。

1. 次のコマンドを入力して、エージェント コード ファイルを編集します。

    ```
   code agent_client.py
    ```

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
        extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
        input=""
    )

    # Check if the response output contains an MCP approval request
    approval_request = None
    if hasattr(response, 'output') and response.output:
        for item in response.output:
            if hasattr(item, 'type') and item.type == 'mcp_approval_request':
                approval_request = item
                break
    
    # Handle approval request if present
    if approval_request:
        print(f"[Approval required for: {approval_request.name}]\n")
        print(f"Server: {approval_request.server_label}")
        
        # Parse and display the arguments (optional, for transparency)
        import json
        try:
            args = json.loads(approval_request.arguments)
            print(f"Arguments: {json.dumps(args, indent=2)}\n")
        except:
            print(f"Arguments: {approval_request.arguments}\n")
        
        # Prompt user for approval
        approval_input = input("Approve this action? (yes/no): ").strip().lower()
        
        if approval_input in ['yes', 'y']:
            print("Approving action...\n")
            
            # Create approval response item
            approval_response = {
                "type": "mcp_approval_response",
                "approval_request_id": approval_request.id,
                "approve": True
            }
        else:
            print("Action denied.\n")
            
            # Create denial response item
            approval_response = {
                "type": "mcp_approval_response",
                "approval_request_id": approval_request.id,
                "approve": False
            }
        
        # Add the approval response to the conversation
        openai_client.conversations.items.create(
            conversation_id=conversation.id,
            items=[approval_response]
        )
        
        # Get the actual response after approval/denial
        response = openai_client.responses.create(
            conversation=conversation.id,
            extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
            input=""
        )
    
    ```

1. コードを追加したら、**Ctrl + S** キーのコマンドを使って変更を保存します。 

1. コードが会話 API を使ってエージェントとの対話を管理するようになっていることを確認します。次のようなものです。
    - 会話が作成され、その ID によって追跡されます
    - `conversations.items.create()` を使ってユーザー メッセージが会話に追加されます
    - `responses.create()` とエージェントの参照を使って応答が生成されます
    - **MCP 承認処理**: エージェントは、Foundry IQ にアクセスする必要がある場合、応答出力で `mcp_approval_request` を返して承認を要求します
    - このコードは、続ける前にアクションを承認または拒否するようユーザーに求めます
    - 承認または拒否の後、`mcp_approval_response` が会話に追加されて、新しい応答が生成されます
    - エージェントは、ユーザーの承認の決定に基づいて Foundry IQ から情報を取得します

1. **Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

## 統合をテストします

次に、アプリケーションを実行し、ナレッジ ベースから情報を取得するエージェントの機能をテストします。

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Azure にサインインします。

    ```
    az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。

1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、メッセージが表示されたら Foundry リソースを含むサブスクリプションを選択します。

1. Cloud Shell のコマンドライン ペインで、アプリケーションを実行します。

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
- Python SDK を使って Azure Cloud Shell からエージェントに接続しました
- クライアント アプリケーションに MCP 承認処理、会話履歴、エラー処理を実装しました
- 外部ツールのアクセスについてユーザーが制御する承認を使って、ナレッジ ベースから情報を取得して合成するエージェントの機能をテストしました

これにより、AI エージェントと Foundry IQ を統合し、会話のコンテキストを維持しながらエンタープライズ ナレッジ ベースで情報を検索して取得できるインテリジェントなアプリケーションを作成する方法がわかります。

## クリーンアップ

Azure AI Agent サービスと Foundry IQ を調べる演習が済んだら、不要な Azure コストが発生しないよう、この演習で作成したリソースを削除する必要があります。

1. Cloud Shell のブラウザー タブを閉じます。
1. ブラウザーに戻り、`https://portal.azure.com` で [Azure portal](https://portal.azure.com) を開きます。
1. Foundry リソースと AI Search リソースを含むリソース グループに移動します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。

