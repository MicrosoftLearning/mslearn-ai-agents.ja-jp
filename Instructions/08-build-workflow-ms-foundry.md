---
lab:
  title: Microsoft Foundry でワークフローを構築する
  description: Microsoft Foundry ポータルを使用して、AI エージェント用のワークフローを作成します。
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

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開かれるヒントやクイック スタートのペインを閉じ、必要に応じて左上の **Foundry** ロゴを使用して、次の図のようなホーム ページに移動します (**[ヘルプ]** ペインが開いている場合は閉じます)。

1. **[新しい Foundry]** トグルを "オン" にします。**

    <img src="./Media/ai-foundry-toggle.png" alt="Screenshot of the New Foundry toggle" width="300">

1. [新しい Foundry] エクスペリエンスに進む前に、それを作成するように求められます。 **[新しいプロジェクトの作成]** を選択します。

    <img src="./Media/ai-foundry-new-project.png" alt="Screenshot of the Create project pane." width="600">

1. テキスト ボックスに Foundry プロジェクトの名前を入力し、**[作成]** を選択します。

    プロジェクトが作成されるまでしばらく待ちます。 新しい Foundry ポータルのホーム ページが表示されます。

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

    <img src="./Media/node-new-variable.png" alt="Screenshot of creating a new variable in the Set variable node." width="500">

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

    <img src="./Media/agent-parameters.png" alt="Screenshot of the Parameters button in the agent editor." width="400">

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

## ワークフローを実行してテストする

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

## まとめ

この演習では、Microsoft Foundry で、カスタマー サポート チケットを処理するシーケンシャル ワークフローを作成しました。 条件付きロジックと構成済みの AI エージェントを使用して、JSON 形式の出力を生成しました。 ワークフローでは、AI エージェントを使用して各チケットを分類し、条件付きロジックを使用して信頼度の低い分類を処理し、課金以外の問題に対して推奨される応答を生成しました。 よくできました。

## クリーンアップ

Microsoft Foundry でのワークフローを調べ終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) に移動し、Foundry プロジェクトをデプロイしたリソース グループの内容を表示します。

1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。