---
title: タスク 1 - Foundry IQ ナレッジ エージェントを作成し、コードから接続する
lab:
  title: タスク 1 - Foundry IQ ナレッジ エージェントを作成し、コードから接続する
  description: Microsoft Foundry ポータルでエンタープライズナレッジ エージェントを作成し、Foundry IQ を使って Caldova のナレッジ ベースに典拠させ、ナレッジ検索の前に承認を要求し、コードから接続して承認フローを処理します。
  type: task
  parent: B
  order: 1
  section: core
  difficulty: 3
  duration: 35
  access: open
  level: 300
  concepts: 'Foundry IQ, enterprise knowledge grounding, tool approvals, conversations API'
  status: draft
---

# タスク 1 — Foundry IQ ナレッジ エージェントを作成し、コードから接続する

これは、**エージェントをエンタープライズ ナレッジおよび Microsoft 365 と統合する**ラボの一部です。初めてご覧になる方は、「[はじめに](B0-getting-started.md)」から開始してください。**

> **設定 (ここから始めます):** このタスクには Foundry プロジェクト (展開済みモデル付き) とスタート コードが必要です。 まだ用意していない場合は、「[はじめに](B0-getting-started.md)」を完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定します。 次に、VS Code で開いた `Python` フォルダーから準備ができていることを確認します。

```
python ../setup/check_env.py --task 1
```

> **前のタスクから続けている場合** プロジェクト、仮想環境、`.env` が既に設定されている場合は、セットアップを飛ばして下記の「**エージェントを作成**」にスキップします。

---

**Caldova Knowledge Assistant** を構築します。**Foundry IQ** を使用して、会社の内部ドキュメント (現場業務、工場のキャパシティ、CMO ディレクトリ、技術移転、サプライヤー) に基づくエージェントを構築し、これを承認手順で各ナレッジ検索を制御する Python アプリから接続します。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>Foundry IQ とは</summary>
<div class="concept-body" markdown="1">

**Foundry IQ** は、Azure AI 検索 でサポートされる独自のドキュメントから構築された検索可能なインデックスである**ナレッジ ベース**にエージェントを接続します。 エージェントで事実が必要とされるとき、そのナレッジ ベースに対して "エージェント取得" を行い、発見したものを引用します。** 各検索の前に **承認** を要求して、アプリケーションですべてのナレッジ ベースへのアクセスを確認および管理できます。

[詳細情報 →](https://learn.microsoft.com/azure/ai-foundry/)

</div>
</details>

## エージェントを作成する

もし「はじめに」で `caldova-knowledge-agent` を作成済みの場合は、それを開いて (**[ビルド]** → **[エージェント]** → **caldova-knowledge-agent**)、「**データと Foundry IQ を構成する**」にスキップしてください。 それ以外の場合:

1. ホーム ページで **[ビルド]** タブを選択し、**[エージェント]** タブで **[エージェントの作成]** を選択します。
1. エージェントを `caldova-knowledge-agent` という名前で作成します。

エージェントの作成時に、既定のモデル (例: `gpt-5`) が展開されます。 エージェントが作成されると、その既定のモデルが自動的に選ばれたエージェント プレイグラウンドが表示されます。

## データと Foundry IQ を構成する

次に、Foundry IQ を使って Caldova ナレッジ ベースを検索するエージェントを構成します。

1. まず、エージェントに次のように指示します。

    ```
    You are the Caldova staff knowledge assistant, specializing in plant capacity,
    contract manufacturers, tech transfer, site operations, and suppliers. You must
    ALWAYS search the knowledge base to answer questions about our capacity, policies,
    or procedures. Provide detailed, accurate information and always cite your sources.
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

Caldova のナレッジ ドキュメントをアップロードして Foundry IQ に接続します。

1. サンプルのナレッジ ドキュメントをダウンロードします。 これらは、`Labfiles/B-integrate-agents-with-enterprise-knowledge-and-m365/Python/data/` の下のスタート コードにある Markdown ファイルです。
    - `caldova-site-operations.md`
    - `caldova-plant-capacity.md`
    - `caldova-cmo-directory.md`
    - `caldova-tech-transfer-playbook.md`
    - `caldova-capacity-booking-policy.md`
    - `caldova-supplier-guide.md`

    > **ヒント**: これらは既に「はじめに」でローカルに保持しています。 直接ダウンロードする場合は、[リポジトリ](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/main/Labfiles/B-integrate-agents-with-enterprise-knowledge-and-m365/Python/data)の `data` フォルダーを参照し、各ファイルを保存します。

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
1. **[BLOB のアップロード]** ブレードで、`caldovaproducts` という名前の新しいコンテナーを作成します。
1. `data` フォルダーから 6 つの Caldova の Markdown ファイルを探し、すべてを選択して **[アップロード]** を選択します。
1. ファイルがアップロードされたら、自身で作成した検索サービスの画面に移動します。
1. 左側のペインの **[セキュリティとネットワーク]** > **[キー]** で、[API アクセス制御] の **[両方]** を選択し、選択を確定します。 完了したら、Azure portal のタブを開いたまま Foundry ポータルのタブに戻り、ページを最新の情報に更新します。
1. **[ナレッジ]** ページにいることを確認して、**[ナレッジ ベースの作成]** を選択し、ナレッジ ソースとして **[Azure Blob Storage]** を選択して、**[接続]** を選択します。
1. 次の設定でナレッジ ソースを構成します。
    - **名前**: `ks-caldovaproducts`
    - **説明**: `Caldova staff knowledge base`
    - **ストレージ アカウント名**: *ストレージ アカウントを選択します。*
    - **[コンテナー名]**: `caldovaproducts`
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
    - `Which sites can make oral solid dose product?`
    - `Tell me which contract manufacturers are qualified for sterile work.`
    - `How much headroom does Calderwood have?`

1. 応答を確認して次のことを理解します。
    - エージェントは、ナレッジ ベースから特定の情報を提供します
    - ソース ドキュメントの引用または参照が含まれる場合があります
    - エージェントは Caldova の情報に焦点を当てている

1. エージェントの詳細ページで、次の情報を見つけてメモ帳にコピーします (これらは後で必要になります)。
    - **エージェント名**: これは、ご自分で作成した名前です (`caldova-knowledge-agent`)
    - **プロジェクト エンドポイント**: プロジェクトの設定またはホーム ページにあります

### ツール呼び出しの承認を求めるようにエージェントを構成する

ポータルでエージェントを作成すると、Foundry IQ (ナレッジ) ツールは既定で承認を求める**ことなく**実行されます。 確実にアプリで各ナレッジ ベースの検索をレビューして制御できるようにするには、Foundry Toolkit for VS Code 拡張機能でツールを使用する前に、承認を求めるようにエージェントを変更します。

> **注**: Foundry ポータルでは現在、この承認動作を変更するための設定が公開されていないため、代わりに Foundry Toolkit 拡張機能から構成します。

1. Visual Studio Code で、左側のペインから **[拡張機能]** を選択し (または、**Ctrl + Shift + X** キーを押し)、Microsoft の `Foundry Toolkit for VS Code` 拡張機能をマーケットプレースで検索し、**[インストール]** を選びます (まだインストールされていない場合)。

    > **注**: 拡張機能は現在、**Foundry Toolkit** として登録されていますが、一部の VS Code のラベル、コマンド、または以前のスクリーンショットでは、引き続き **AI Toolkit** と表示されている場合があります。 このラボでは、これらの名前が同じ拡張機能エクスペリエンスを表しているものとして扱います。

1. サイドバーの **[Foundry Toolkit]** アイコンを選択し、メッセージが表示されたら Azure アカウントにサインインします。

    > **注**: Foundry Toolkit 拡張機能でサインインできない場合は、Azure 拡張機能を選択する必要がある場合があります。 そこでサインインし、Foundry Toolkit に戻ってリソースにアクセスします。

1. **[Microsoft Foundry リソース]** で、**[既定のプロジェクトの設定]** を選び、前に作成したプロジェクトを選択します。
1. [プロジェクト] セクションを展開します。 **[プロンプト エージェント]** で、`caldova-knowledge-agent` エージェントを選択して **[エージェント ビルダー]** ウィンドウを開きます。
1. **[ツール]** セクションで、`kb-knowledgebase` プレフィックスの後に一意の ID が付いたツール名 (例: `kb-knowledgebase677-7w5fj`) を見つけます。 これは Foundry IQ のナレッジ ベース ツールで、ポータルで Foundry IQ を接続したときに自動的に追加されたものです。

    > **注**: エージェントには複数のツールが一覧表示される場合があります。 既定では、Foundry ポータルが新しいエージェントに **Web 検索**ツールを追加し、スタンドアロンの **Azure AI 検索** ツールも表示される場合があります。 エージェントはナレッジ ベース検索時に実際に `kb-knowledgebase...` ツールを呼び出すので、他のツールで承認を設定しても影響はありません。
1. `kb-knowledgebase...` ツールの 3 つのドットを選択します。 次に、**[ツール使用前に承認が必要]** ドロップダウンで、**[すべてのツールの承認を求める]** を選択し、メッセージが表示されたら変更を保存します。

これで、エージェントは Foundry IQ を使用してナレッジ ベースを検索するたびに承認を求めるようになり、次に完了するクライアント アプリによって処理されます。

## コードからエージェントに接続する

次に、エージェントと通信し承認フローを処理する Python コンソール クライアントを完成させます。 スターター ファイルは `Python` フォルダーに用意されています。

`Python` フォルダーを開き、「[はじめに](B0-getting-started.md)」から仮想環境をアクティブ化し (`.\labenv\Scripts\Activate.ps1`)、下記に進みます。

1. **Python/.env** で、`AGENT_NAME` が `caldova-knowledge-agent` (`.env.example` の既定値) に設定されていることを確認します。 ファイルを保存します。

1. **knowledge_agent.py** を開いて、次の内容を含めてスタート コードを確認します。
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

1. `send_message_to_agent()` 関数内で 2 番目の **TODO** コメントを見つけ、Foundry IQ 承認要求などのメッセージを送信して応答を処理する次のコードを追加します。

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
        except Exception:
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
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
            input=""
        )
    ```

1. コードを追加したら、ファイルを保存します。

1. コードで会話 API を使用してエージェントとの対話を管理する方法を確認します。ここで、次の操作を行います。
    - 会話が作成され、その ID によって追跡されます
    - `conversations.items.create()` を使ってユーザー メッセージが会話に追加されます
    - `responses.create()` とエージェントの参照を使って応答が生成されます
    - **承認処理**: エージェントは、Foundry IQ にアクセスする必要がある場合、応答出力で `mcp_approval_request` を返します
    - このコードは、続ける前にアクションを承認または拒否するようユーザーに求めます
    - 承認または拒否の後、`mcp_approval_response` が会話に追加されて、新しい応答が生成されます

## 統合をテストする

次に、アプリケーションを実行し、ナレッジ ベースから情報を取得するエージェントの機能をテストします。

1. ターミナル (`Python` フォルダー内) で Azure にサインインします。

    ```
    az login
    ```

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。

1. ダイアログが表示されたらサインイン プロセスを完了し、Foundry リソースを含むサブスクリプションを選択します。

1. アプリケーションを実行します。

    ```
    python knowledge_agent.py
    ```

1. アプリケーションが開始したら、次のクエリを使ってエージェントをテストします。

    **クエリ 1 - 製品カテゴリ:**

    ```
    Which sites can make oral solid dose product?
    ```

    承認を求められたら、「**yes**」と入力して、エージェントがナレッジ ベースを検索するのを許可します。 エージェントが複数のドキュメントから情報を取得する方法を観察します。

    **クエリ 2 - キャパシティ ポリシー:**

    ```
    How much headroom does Calderwood have and how are transfer costs calculated?
    ```

    要求を承認し、エージェントがキャパシティ要求ポリシーから特定の詳細を提供する方法に注目します。

    **クエリ 3 - 契約メーカー比較:**

    ```
    What's the difference between Norvent and Halden for a sterile transfer?
    ```

    要求を承認し、エージェントが CMO ディレクトリから情報を合成する方法を確認します。

    **クエリ 4 - サプライヤーと再注文:**

    ```
    When should we reorder sterile vials, and who is our component supplier?
    ```

    要求を承認し、エージェントがサプライヤー ガイドから回答する様子を確認します。

    **クエリ 5 - フォローアップの質問:**

    ```
    What are our site core hours?
    ```

    エージェントが現場業務ドキュメントから会話のコンテキストと回答を維持する方法に注目します。

1. 「`history`」と入力して、完全な会話履歴を表示します。

1. テストを終えたら、「`quit`」と入力します。

> ✅ **チェックポイント**: あなたは Foundry IQ でエンタープライズナレッジ エージェントを作成し、**典拠**しました。各ナレッジ検索の前に**承認**が必要で、コードに接続されています。承認フローは自分で処理します。 これがこのラボの中核です。 下記はすべて任意です。

### オプション: Web チャット アプリと同じエージェントを動かす

同じ典拠済みのエージェントは、共有の Caldova Web チャット ウィンドウを通じて提供できます。 `Python` フォルダーから、下記を実行します。

```
python knowledge_chat_app.py
```

`http://localhost:7860` でブラウザーが開き、**Caldova Staff Knowledge Assistant** と表示されます。 このバリアントは **Foundry IQ のナレッジ ツールを自動承認**し、チャットを円滑に進めます。 上記と同じ質問をしてみましょう。 タブを閉じて **Ctrl + C** を押して停止します。

> **早送り**: ポータルではなくコードでエージェントの典拠を関連付ける場合、`Python` フォルダーから `python ../setup/bootstrap_agent.py` を実行します。 これにより `caldova-knowledge-agent` が作成され、6 つのナレッジ ドキュメントを典拠とするファイル検索が行われ、`.env` に `AGENT_NAME` が書き込まれます。 実行するクライアント コードは同一です。

完了したら、ターミナルに「`deactivate`」と入力して、仮想環境を終了します。

---

**次へ (任意):** [タスク 2 — Microsoft Teams に公開する](B2-publish-to-microsoft-teams.md) · [タスク 3 — Microsoft 365 Copilot に公開する](B3-publish-to-microsoft-365-copilot.md) · [タスク 4 — Work IQ](B4-work-iq-workplace-intelligence.md)
