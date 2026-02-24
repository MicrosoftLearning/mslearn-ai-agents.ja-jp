---
lab:
  title: Microsoft Foundry でワークフローを構築する
  description: Microsoft Foundry ポータルを使用して、AI エージェント用のワークフローを作成します。
  level: 300
  duration: 45
---

# Microsoft Foundry でワークフローを構築する

この演習では、Microsoft Foundry ポータルを使用してワークフローを作成します。 ワークフローは、AI エージェントに関連する一連のアクションを定義するのに使用できる UI ベースのツールです。 この演習では、カスタマー サポート要求の解決に役立つワークフローを作成します。

**ワークフローの概要**

- 受信したサポート チケットを収集する
    
    このワークフローは、事前に定義されたカスタマー サポートの問題の配列から始まります。 配列内の各項目は、ContosoPay に送信された個々のサポート チケットを表します。

- チケットを一度に 1 つずつ処理する
    
    for-each ループは配列を反復処理し、各サポート チケットが同じワークフロー ロジックを使用しながら個別に処理されるようにします。

- AI エージェントを使用して各チケットを分類する
    
    ワークフローでは、チケットごとにトリアージ エージェントを呼び出して、問題を課金、技術、または一般として分類し、信頼度を設定します。

- 条件付きロジックを使用して不確実性を処理する
    
    信頼度スコアが定義されたしきい値を下回る場合、ワークフローはそのチケットに関する追加情報を推奨します。

- 問題のカテゴリに基づいてルーティングする
    
    課金の問題にはエスカレーションのフラグが設定され、自動解決パスから削除されます。
    技術的および一般的な問題については、自動処理が続行されます。

- 推奨される応答を生成する
    
    課金以外のチケットの場合、ワークフローは解決エージェントを呼び出して、カテゴリに適したサポート応答の下書きを作成します。

この演習の所要時間は約 **30** 分です。

> **注**:Microsoft Foundry のワークフロー ビルダーは現在プレビュー段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Foundry プロジェクトを作成する

まず、Foundry プロジェクトを作成しましょう。

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。

1. **[新しい Foundry]** トグルが *[オン]* に設定されていることを確認します。

    <img src="../Media/ai-foundry-toggle.png" alt="Screenshot of the New Foundry toggle" width="300">

1. 新しい Foundry エクスペリエンスに進む前に、新しいプロジェクトを作成するように求められる場合があります。 **[新しいプロジェクトの作成]** を選択します。

    <img src="../Media/ai-foundry-new-project.png" alt="Screenshot of the Create project pane." width="600">

    メッセージが表示されない場合は、左上の [プロジェクト] ドロップダウン メニューを選択してから、**[新しいプロジェクトの作成]** を選択します。

1. テキスト ボックスに Foundry プロジェクトの名前を入力し、**[作成]** を選択します。

    プロジェクトが作成されるまでしばらく待ちます。 プロジェクトが選択された状態で、新しい Foundry ポータルのホーム ページが表示されます。

## カスタマー サポートのトリアージ ワークフローを作成する

このセクションでは、ContosoPay という架空の会社のカスタマー サポート要求をトリアージして応答するのに役立つワークフローを作成します。 このワークフローでは、サポート チケットを分類して応答する 2 つの AI エージェントを使用します。

1. Foundry ポータルのホーム ページで、ツール バー メニューから **[ビルド]** を選択します。

1. 左側のメニューで、**[ワークフロー]** を選択します。

1. 右上隅にある **[作成]** > **[空のワークフロー]** を選択して、新しい空のワークフローを作成します。

    この演習で作成するワークフローの種類は、シーケンシャル ワークフローです。 ただし、空のワークフローから開始すると、必要なノードを追加するプロセスが簡略化されます。

1. ビジュアライザーで **[保存]** を選択して、新しいワークフローを保存します。 ダイアログ ボックスで、ワークフローの名前 (*ContosoPay-Customer-Support-Triage* など) を入力し、**[保存]** を選択します。

### チケットの配列の変数を作成する

1. ワークフロー ビジュアライザーで、**[+]** (プラス) アイコンを選択して、新しいノードを追加します。

1. ワークフローのアクション メニューの **[データ変換]** で、**[変数の設定]** を選択して、サポート チケットの配列を初期化するノードを追加します。

1. **[変数の設定]** ノード エディターで、**[新しい変数の作成]** を選択して新しい変数を作成します。 *SupportTickets* などの名前を入力します。

    <img src="../Media/node-new-variable.png" alt="Screenshot of creating a new variable in the Set variable node." width="500">

    新しい変数が `Local.SupportTickets` として表示されます。

1. **[対象値]** フィールドに、サンプルのサポート チケットを含む次の配列を入力します。

    ```output
   [ 
    "The API returns a 403 error when creating invoices, but our API key hasn't changed.", 
    "Is there a way to export all invoices as a CSV?", 
    "I was charged twice for the same invoice last Friday and my customer is also seeing two receipts. Can someone fix this?"]
    ```

1. **[完了]** を選択して、ノードを保存します。

### チケットを処理するための for-each ループを追加する

1. **[変数の設定]** の下にある **[+]** (プラス) アイコンを選択し、配列内の各サポート チケットを処理する **[For each]** ノードを作成します。

1. **[For each]** ノード エディターで、**[For each ループする項目を選択する]** フィールドを、先ほど作成した変数 (`Local.SupportTickets`) に設定します。

1. **[ループ値変数]** フィールドで、`CurrentTicket` という名前の新しい変数を作成します。

1. **[完了]** を選択して、ノードを保存します。

### エージェントを呼び出してチケットを分類する

1. **For each**ノード内の **[+]** (プラス) アイコンを選択して、現在のサポート チケットを分類する新しいノードを追加します。

1. ワークフローのアクション メニューの **[呼び出し]** で、**[エージェントの呼び出し]** を選択して、エージェント ノードを追加します。

1. **[エージェントの呼び出し]** ノード エディターの **[エージェントの選択]** で、**[新しいエージェントの作成]** を選択します。

1. *Triage-Agent* などのエージェント名を入力し、**[作成]** を選択します。

#### エージェント設定を構成する

1. エディターの **[詳細]** で、モデル名の近くにある **[パラメーター]** ボタンを選択します。

    <img src="../Media/agent-parameters.png" alt="Screenshot of the Parameters button in the agent editor." width="400">

1. **[パラメーター]** ペインで、**[テキスト形式]** の横にある **[JSON スキーマ]** を選択します。

1. **[応答形式の追加]** ペインで、次の定義を入力し、**[保存]** を選択します。

    ```json
    {
    "name": "category_response",
    "schema": {
        "type": "object",
        "properties": {
            "customer_issue": {
                "type": "string"
            },
            "category": {
                "type": "string"
            },
            "confidence": {
                "type": "number"
            }
        },
        "additionalProperties": false,
        "required": [
            "customer_issue",
            "category",
            "confidence"
        ]
    },
    "strict": true
    }
    ```

1. [エージェントの呼び出し] の [詳細] ペインで、**[指示]** フィールドを次のプロンプトに設定します。

    ```output
    Classify the user's problem description into exactly ONE category from the list below. Provide a confidence score from 0 to 1.

    Billing
    - Charges, refunds, duplicate payments
    - Missing or incorrect payouts
    - Subscription pricing or invoices being charged

    Technical
    - API errors, integrations, webhooks
    - Platform bugs or unexpected behavior

    General
    - How-to questions
    - Feature availability
    - Data exports, reports, or UI navigation

    Important rules
    - Questions about exporting, viewing, or downloading invoices are General, not Billing
    - Billing ONLY applies when money was charged, refunded, or paid incorrectly
    ```

1. **[アクションの設定]** を選択して、エージェントの入力と出力を構成します。

1. **[入力メッセージ]** フィールドを `Local.CurrentTicket` 変数に設定します。

1. **[名前を付けてエージェントの出力メッセージを保存]** で、`TriageOutputText` という名前の新しい変数を作成します。

1. **[名前を付けて出力 json_object を保存]** で、`TriageOutputJson` という名前の新しい変数を作成します。

1. **[完了]** を選択して、ノードを保存します。

### 信頼度の低い分類を処理する

1. **[エージェントの呼び出し]** ノードの下にある **[+]** (プラス) アイコンを選択して、信頼度の低い分類を処理する新しいノードを追加します。

1. ワークフローのアクション メニューの **[フロー]** で、**[If/Else]** を選択して、条件付きロジック ノードを追加します。

1. **[If/Else]** ノード エディターで鉛筆アイコンを選択して、**[If]** 分岐条件を編集します。

1. **[条件]** フィールドを次の式に設定して、信頼度スコアが 0.6 を上回るかどうかを確認します。

    ```output
   Local.TriageOutputJson.confidence > 0.6
    ```

1. **[完了]** を選択して、ノードを保存します。

### 信頼度の低いチケットの追加情報を推奨する

1. ビジュアライザーで、**[If/Else 条件]** ノードの **[Else]** 分岐の下にある **[+]** (プラス) アイコンを選択して、信頼度の低いチケットの追加情報を推奨する新しいノードを追加します。

1. ワークフローのアクション メニューの **[基本情報]** で、**[メッセージの送信]** を選択して、メッセージの送信アクティビティを追加します。

1. **[メッセージの送信]** ノード エディターで、**[送信するメッセージ]** フィールドを次の応答に設定します。

    ```output
   The support ticket classification has low confidence. Requesting more details about the issue: "{Local.CurrentTicket}"
    ```

### カテゴリに基づいてチケットをルーティングする

このセクションでは、信頼度スコアが十分に高い場合に、分類されたカテゴリに基づいてチケットをルーティングする条件付きロジックを追加します。

1. ビジュアライザーで、**[If/Else 条件]** ノードの **[If]** 分岐の下にある **[+]** (プラス) アイコンを選択して、カテゴリに基づいてチケットをルーティングする新しいノードを追加します。

1. ワークフローのアクション メニューの **[フロー]** で、**[If/Else]** を選択して、別の条件付きロジック ノードを追加します。

1. **[If/Else]** ノード エディターで、**[If 条件]** を次の式に設定して、チケット カテゴリが "課金" かどうかを確認します。

    ```output
    Local.TriageOutputJson.category = "Billing"
    ```

1. **[If/Else]** ノードの **[If]** 分岐の下にある **[+]** (プラス) アイコンを選択して、課金以外のチケットに対する応答の下書きを行う新しいノードを追加します。

1. ワークフローのアクション メニューの **[基本情報]** で、**[メッセージの送信]** を選択して、メッセージの送信アクティビティを追加します。

1. **[メッセージの送信]** ノード エディターで、**[送信するメッセージ]** を次の応答に設定します。

    ```output
   Escalate billing issue to human support team.
    ```

1. **[完了]** を選択して、ノードを保存します。

### 推奨される応答を生成する

1. ビジュアライザーで、2 番目の **[If/Else]** ノードの **[Else]** 分岐の下にある **[+]** (プラス) アイコンを選択して、課金以外のチケットに対する応答を下書きする新しいノードを追加します。

1. ワークフローのアクション メニューの **[エージェント]** で、**[エージェントの呼び出し]** を選択して、エージェント ノードを追加します。

1. **[エージェントの呼び出し]** ノード エディターで、**[新しいエージェントの作成]** を選択します。

1. *Resolution-Agent * などのエージェント名を入力し、**[作成]** を選択します。

1. エージェント エディターで、**[指示]** フィールドを次のプロンプトに設定します。

    ```output
    You are a customer support resolution assistant for ContosoPay, a B2B payments and invoicing platform.

    Your task is to draft a clear, professional, and friendly support response based on the issue category and customer message.

    Guidelines:
    If the issue category is Technical:
    Suggest 1–2 common troubleshooting steps at a high level.

    Avoid asking for logs, credentials, or sensitive data.

    Do not imply fault by the customer.
    If the issue category is General:
    Provide a concise, helpful explanation or guidance.
    Keep the response under 5 sentences.

    Tone:
    Professional, calm, and supportive
    Clear and concise
    No emojis

    Output:
    Return only the drafted response text.
    Do not include internal reasoning or analysis.
    ```

1. **[アクションの設定]** を選択して、エージェントの入力と出力を構成します。

1. **[入力メッセージ]** フィールドを `Local.TriageOutputText` 変数に設定します。

1. **[名前を付けてエージェントの出力メッセージを保存]** で、`ResolutionOutputText` という名前の新しい変数を作成します。

1. **[完了]** を選択して、ノードを保存します。

## ワークフローをプレビューする

1. **[保存]** ボタンを選択して、ワークフローに対するすべての変更を保存します。

1. **[プレビュー]** ボタンを選択してワークフローを開始します。

1. 表示されるチャット ウィンドウで、何らかのテキスト (例: `Start processing support tickets.`) を入力して、ワークフローをトリガーします

1. ワークフローで各サポート チケットが順番に処理されるのを確認します。 チャット ウィンドウで、ワークフローによって生成されたメッセージを確認します。

    課金の問題については、エスカレートされていることを示す出力がいくつか表示され、技術的な問題と一般的な問題については、下書きされた応答が返されます。 次に例を示します。

    ```output
    Current Ticket:
    The API returns a 403 error when creating invoices, but our API key hasn't changed.


    Copilot said:
    Thank you for reaching out about the 403 error when creating invoices. This error typically indicates a permissions or access issue. 
    Please ensure that your API key has the necessary permissions for invoice creation and that your request is being sent to the correct endpoint. 
    If the issue persists, try regenerating your API key and updating it in your integration to see if that resolves the problem.
    ```

## コードでワークフローを使用する

Foundry ポータルでワークフローをビルドしてテストしたら、それを Azure AI Projects SDK を使用して自分のコードから呼び出すこともできます。 これにより、ワークフローをアプリケーションに統合したり、その実行を自動化したりできます。

### 前提条件

この演習を開始するには、以下のものが必要です。
- Visual Studio Code がインストールされていること
- 有効な Azure サブスクリプション
- Python バージョン 3.10 以降がインストールされている

### Microsoft Foundry VS Code 拡張機能をインストールする

VS Code 拡張機能をインストールして設定することから始めましょう。

1. Visual Studio Code を開きます。

1. 左側のペインから **[拡張機能]** を選択します (または **Ctrl + Shift + X** キーを押します)。

1. 検索バーに「**Microsoft Foundry**」と入力し、Enter キーを押します。

1. Microsoft から **[Microsoft Foundry]** 拡張機能を選択し、**[インストール]** をクリックします。

1. インストールが完了したら、Visual Studio Code の左側のプライマリ ナビゲーション バーに拡張機能が表示されていることを確認します。

### Azure にサインインしてプロジェクトを作成する

次に、Azure リソースに接続し、新しい AI Foundry プロジェクトを作成します。

1. VS Code のサイド バーで、**[Microsoft Foundry]** 拡張機能のアイコンを選択します。

1. [リソース] ビューで、**[Azure にサインイン...]** を選択し、認証プロンプトに従います。

   > **注**:既にサインインしている場合、このオプションは表示されません。

1. Foundry 拡張機能ビューで **[リソース]** の横にある **[+]** (プラス) アイコンを選択して、新しい Foundry プロジェクトを作成します。

1. ドロップダウンから Azure サブスクリプションを選択します。

1. 新しいリソース グループを作成するか、既存のリソース グループを使用するかを選択します。
   
   **新しいリソース グループを作成する場合:**
   - **[新しいリソース グループの作成]** を選択し、Enter キーを押します
   - リソース グループの名前 (例: "rg-ai-agents-lab") を入力し、Enter キーを押します
   - 使用可能なオプションから場所を選択し、Enter キーを押します
   
   **既存のリソース グループを使用する場合:**
   - 使用するリソース グループを一覧から選択し、Enter キーを押します

1. テキストボックスに Foundry プロジェクトの名前 (例: "ai-agents-project") を入力し、Enter キーを押します。

1. プロジェクトのデプロイが完了するまで待ちます。 ポップアップが表示され、"プロジェクトが正常にデプロイされました" というメッセージが表示されます。

### モデルをデプロイする

このタスクでは、エージェントで使用するモデル カタログからモデルをデプロイします。

1. "プロジェクトが正常にデプロイされました" というポップアップが表示されたら、**[モデルのデプロイ]** ボタンを選択します。 これにより、モデル カタログが開きます。

   > **ヒント**: モデル カタログにアクセスするには、[リソース] セクションで **[モデル]** の横にある **[+]** アイコンを選択します。または、**F1** キーを押し、コマンド **Microsoft Foundry: Open Model Catalog**を実行して、モデル カタログにアクセスすることもできます。

1. モデル カタログで、**[gpt-4.1]** モデルを見つけます (検索バーを使用するとすばやく見つけることができます)。

    ![Foundry VS Code 拡張機能の [モデル カタログ] のスクリーンショット。](../Media/vs-code-model.png)

1. gpt-4.1 モデルの横にある **[デプロイ]** を選択します。

1. デプロイ設定を構成します。
   - **デプロイ名**:"gpt-4.1" のような名前を入力します
   - **デプロイの種類**:**[グローバル標準]** (グローバル標準を使用できない場合は **[標準]**) を選択します
   - **モデルのバージョン**: デフォルトのままにする
   - **1 分あたりのトークン数**:デフォルトのままにする

1. 左下隅にある **[Microsoft Foundry にデプロイ]** を選択します。

1. 確認ダイアログで **[デプロイ]** を選択して、モデルをデプロイします。

1. デプロイが完了するまで待ちます。 デプロイしたモデルが、[リソース] ビューの **[モデル]** セクションに表示されます。

1. プロジェクトのデプロイ名を右クリックし、**[プロジェクト エンドポイントのコピー]** を選択します。 次のステップでエージェントを Foundry プロジェクトに接続するために、この URL が必要になります。

   <img src="../Media/vs-code-endpoint.png" alt="Screenshot of copying the project endpoint in the Foundry VS Code extension." width="550">

### スタート コード リポジトリをクローンする

この演習では、Foundry プロジェクトに接続してワークフローを呼び出すのに役立つスタート コードを使用します。

1. VS Code の **[ようこそ]** タブに移動します (メニュー バーから **[ヘルプ] > [ようこそ]** を選択すると開きます)。

1. **[Git リポジトリの複製]** を選択し、スタート コード リポジトリの URL `https://github.com/MicrosoftLearning/mslearn-ai-agents.git` を入力します。

1. 新しいフォルダーを作成し、**[リポジトリの宛先として選択]** を選択し、プロンプトが表示されたらクローンされたリポジトリを開きます。

1. エクスプローラー ビューで、**[Labfiles/08-build-workflow-ms-foundry/Python]** フォルダーに移動して、この演習のスタート コードを見つけます。

1. **requirements.txt** ファイルを右クリックし、**[統合ターミナルで開く]** を選択します。

1. ターミナルで、次のコマンドを入力して、必要な Python パッケージをインストールします。

    ```
    pip install -r requirements.txt
    ```

1. **.env** ファイルを開き、**[your_project_endpoint]** プレースホルダーをプロジェクトのエンドポイント (Microsoft Foundry 拡張機能のプロジェクト デプロイ リソースからコピーしたもの) に置き換え、MODEL_DEPLOYMENT_NAME 変数がモデル デプロイ名に設定されていることを確認します。 これらの変更を行った後、**Ctrl + S** を使用してファイルを保存します。

### ワークフローに接続して実行する

これで、ワークフローを呼び出すプロジェクトを作成する準備ができました。 それでは始めましょう。

1. コード エディターで **workflow.py** ファイルを開きます。 

1. 各エージェント名と指示の文字列が含まれていることに注意して、ファイル内のコードを確認します。

1. 「**Add references (参照を追加する)**」というコメントを見つけて以下のコードを追加し、必要なクラスをインポートします。

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import ItemType
    ```

1. 環境変数からプロジェクトのエンドポイントとモデル名を読み込むコードが提供されていることに注目してください。

1. コメント **Connect to the agents client (エージェント クライアントに接続する)** を見つけて次のコードを追加し、プロジェクトに接続された AgentsClient を作成します。

    ```python
   # Connect to the AI Project client
   with (
       DefaultAzureCredential() as credential,
       AIProjectClient(endpoint=endpoint, credential=credential) as project_client,
       project_client.get_openai_client() as openai_client,
   ):
    ```

    次に、AgentsClient を使用して複数のエージェントを作成するコードを追加します。各エージェントには、サポート チケットの処理で果たす特定の役割があります。

    > **ヒント**: 後続のコードを追加するときは、適切なレベルのインデントを維持してください。

1. コメント **Specify the workflow** と次のコードを見つけます。

    ```python
   # Specify the workflow
    workflow = {
        "name": "ContosoPay-Customer-Support-Triage",
        "version": "1",
    }
    ```

    Foundry ポータルで作成したワークフローの名前とバージョンを必ず使用してください。

1. コメント **Create a conversation and run the workflow** を見つけ、次のコードを追加して会話を作成し、ワークフローを呼び出します。

    ```python
    # Create a conversation and run the workflow
    conversation = openai_client.conversations.create()
    print(f"Created conversation (id: {conversation.id})")

    stream = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent": {"name": workflow["name"], "type": "agent_reference"}},
        input="Start",
        stream=True,
        metadata={"x-ms-debug-mode-enabled": "1"},
    )
    ```

    このコードは、ワークフロー実行の出力をコンソールにストリーミングして、ワークフローが各チケットを処理するときにメッセージのフローを確認できるようにします。

1. コメント **Process events from the workflow run** を見つけ、次のコードを追加してストリーミングされた出力を処理し、メッセージをコンソールに出力します。

    ```python
    # Process events from the workflow run
    for event in stream:
        if (event.type == "response.completed"):
            print("\nResponse completed:")
            for message in event.response.output:
                if message.content:
                    for content_item in message.content:
                        if content_item.type == 'output_text':
                            print(content_item.text)
        if (event.type == "response.output_item.done") and event.item.type == ItemType.WORKFLOW_ACTION:
            print(f"item action ID '{event.item.action_id}' is '{event.item.status}' (previous action ID: '{event.item.previous_action_id}')")
    ```

1. コメント **Clean up resources** を見つけ、次のコードを入力して不要になった会話を削除します。

    ```python
   # Clean up resources
   openai_client.conversations.delete(conversation_id=conversation.id)
   print("\nConversation deleted")
    ```

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。 

### Azure にサインインしてアプリを実行する

これで、コードを実行し、AI エージェント間の共同作業を確認する準備ができました。

1 統合ターミナルで、次のコマンドを実行します: 

    ```
   python workflow.py
    ```

1. ワークフローがチケットを処理するまで少し待ちます。 ワークフローが実行されると、エージェントによって生成されたメッセージやワークフロー内の各アクションの状態の更新など、ワークフローの進行状況を示す出力がコンソールに表示されます。

1. ワークフローが完了すると、次のような出力が表示されるはずです。

    ```output
    Created conversation (id: {id})
    item action ID 'action-{id}' is 'completed' (previous action ID: 'trigger_id')
    item action ID 'action-{id}' is 'completed' (previous action ID: 'action-{id}')
    item action ID 'action-{id}' is 'completed' (previous action ID: 'action-{id}_Start')
    ...

    Response completed:
    ...
    Current Ticket:
    I was charged twice for the same invoice last Friday and my customer is also seeing two receipts. Can someone fix this?
    {"customer_issue":"I was charged twice for the same invoice last Friday and my customer is also seeing two receipts. Can someone fix this?","category":"Billing","confidence":1}
    Escalation required
    ```

    出力では、各チケットの分類や推奨される応答またはエスカレーションを含めて、ワークフローが各ステップをどのように完了するかを確認できます。 上出来

## まとめ

この演習では、Microsoft Foundry で、カスタマー サポート チケットを処理するシーケンシャル ワークフローを作成しました。 条件付きロジックと構成済みの AI エージェントを使用して、JSON 形式の出力を生成しました。 ワークフローでは、AI エージェントを使用して各チケットを分類し、条件付きロジックを使用して信頼度の低い分類を処理し、課金以外の問題に対して推奨される応答を生成しました。 よくできました。

## クリーンアップ

Microsoft Foundry でのワークフローを調べ終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) に移動し、Foundry プロジェクトをデプロイしたリソース グループの内容を表示します。

1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
