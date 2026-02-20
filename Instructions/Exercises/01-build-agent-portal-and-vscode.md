---
lab:
  title: ポータルと VS Code を使用して AI エージェントを構築する
  description: Microsoft Foundry ポータルと VS Code 拡張機能の両方を使用して、ファイル検索やコード インタープリターなどの組み込みツールを含む AI エージェントを作成します。
  hidden: true
---

# ポータルと VS Code を使用して AI エージェントを構築する

この演習では、Microsoft Foundry ポータルと Microsoft Foundry VS Code 拡張機能の両方を使用して、完全な AI エージェント ソリューションを構築します。 まず、ポータルで典拠データと組み込みツールを含む基本的なエージェントを作成してから、VS Code を使用してプログラムで操作し、コード インタープリターなどの高度な機能をデータ分析に活用します。

この演習は約 **45** 分かかります。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 学習の目的

この演習を終了すると、次のことができるようになります。

1. Microsoft Foundry ポータルで AI エージェントを作成して構成する
2. 典拠データを追加し、組み込みツール (ファイル検索、コード インタープリター) を有効にする
3. Microsoft Foundry VS Code 拡張機能を使用して、プログラムでエージェントを操作する
4. コード インタープリターを活用してデータを分析し、分析情報を生成する
5. エージェント開発でポータルベースとコードベースのアプローチをそれぞれどのようなときに使用するかを理解する

## 前提条件

この演習を開始するには、以下のものが必要です。

- Azure AI リソースをプロビジョニングするための十分なアクセス許可とクォータを持つ Azure サブスクリプション
- ローカル コンピューターにインストールされている Visual Studio Code
- Python 3.12 以降がインストールされている
- Azure AI サービスと Python プログラミングに関する基本的な知識

## シナリオ

一般的な技術上の問題を抱える従業員を支援する **IT サポート エージェント**を構築します。 エージェントは次の内容を実行します。

- IT ポリシードキュメント (典拠データ) に基づいて質問に応答する
- ファイル検索などの組み込みツールを使用して関連情報を検索する
- コード インタープリターを使用してシステム パフォーマンス データを分析し、傾向と問題を特定する

---

## Microsoft Foundry ポータルで AI エージェントを作成する

まずは、ポータルを使用して Foundry のプロジェクトと基本的なエージェントを作成します。

### Foundry プロジェクトを作成する

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインすると開くヒントやクイック スタートのペインをすべて閉じ、必要な場合は、左上にある **Foundry** のロゴを使ってホーム ページに移動します。

    > **重要**: このラボでは、**新しい** Foundry エクスペリエンスを使用しています。

1. 上部のバナーで **[構築の開始]** を選択して、新しい Microsoft Foundry エクスペリエンスをお試しください。

1. プロンプトが表示されたら、**新規**プロジェクトを作成し、プロジェクトに有効な名前 (`it-support-agent-project`など) を入力します。

1. **[詳細オプション]** を展開し、次の設定を指定します。
    - **Microsoft Foundry リソース**:"Foundry リソースの有効な名前"**
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *お手持ちのリソース グループを選択するか、新しいリソース グループを作成します*
    - **リージョン**: ***AI Foundry が推奨するもの***\** の中から選択します

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。

1. プロジェクトが作成されたら、**[構築の開始]** を選択し、ドロップダウン メニューから **[エージェントの作成]** を選択します。

1. **[エージェント名]** を `it-support-agent` に設定し、エージェントを作成します。

新しく作成したエージェントのプレイグラウンドが開きます。 使用可能なデプロイ済みモデルが既に選択されていることがわかります。

### エージェントに指示と典拠データを構成する

エージェントを作成したので、これに指示を構成し、典拠データを追加しましょう。

1. エージェントのプレイグラウンドで、**[指示]** を次のように設定します。

    ```prompt
    You are an IT Support Agent for Contoso Corporation.
    You help employees with technical issues and IT policy questions.
    
    Guidelines:
    - Always be professional and helpful
    - Use the IT policy documentation to answer questions accurately
    - If you don't know the answer, admit it and suggest contacting IT support directly
    - When creating tickets, collect all necessary information before proceeding
    ```

1. ラボ リポジトリから IT ポリシー ドキュメントをダウンロードします。 新しいブラウザー タブを開き、次に移動します。

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/IT_Policy.txt
    ```

    ファイルをローカル コンピューターに保存します。

    > **注**:このドキュメントには、パスワードのリセット、ソフトウェアのインストール要求、ハードウェアのトラブルシューティングに関するサンプルの IT ポリシーが含まれています。

1. エージェントのプレイグラウンドに戻ります。 **[ツール]** セクションで、**[ファイル検索]** と **[コード インタープリター]** の両方を有効にします。

1. **[ファイル検索]** で、**[ファイルのアップロード]** を選択し、今ダウンロードした `IT_Policy.txt` ファイルをアップロードします。

1. ファイルのインデックスが作成されるまで待ちます。 準備ができたら確認が表示されます。

1. 次に、分析するコード インタープリターのパフォーマンス データをいくつか追加しましょう。 次の場所からシステム パフォーマンス データ ファイルをダウンロードします。

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/system_performance.csv
    ```

    このファイルをローカル コンピューターに保存します。

1. **[コード インタープリター]** で、**[ファイルのアップロード]** を選択し、今ダウンロードした `system_performance.csv` ファイルをアップロードします。

    > **注**:この CSV ファイルには、エージェントが分析できるシミュレート済みの経時的システム メトリック (CPU、メモリ、ディスク使用量) が含まれています。

### エージェントをテストする

典拠データを使用してエージェントをテストし、どのように応答するかを確認しましょう。

1. プレイグラウンドの右側にあるチャット インターフェイスで、次のプロンプトを入力します。

    ```
    What's the policy for password resets?
    ```

1. 応答を確認します。 エージェントは IT ポリシー ドキュメントを参照し、パスワードのリセット手順に関する正確な情報を提供するはずです。

1. 別のプロンプトを試します。

    ```
    How do I request new software?
    ```

1. ここでも、応答を確認し、エージェントがどのように典拠データを使用するかを確認してください。

1. 次は、データ分析要求を使用してコード インタープリターをテストします。

    ```
    Can you analyze the system performance data and tell me if there are any concerning trends?
    ```

1. エージェントは、コード インタープリターを使用して CSV ファイルを分析し、システム パフォーマンスに関する分析情報を提供するはずです。

1. 視覚化を要求してみてください。

    ```
    Create a chart showing CPU usage over time from the performance data
    ```

1. エージェントは、コード インタープリターを使用して視覚化と分析を生成します。

完了。 典拠データ、ファイル検索、コード インタープリター機能を備えたエージェントが作成されました。 次のセクションでは、VS Code を使用してプログラムでこのエージェントと対話します。

---

## VS Code を使用してエージェントと対話する

次に、Microsoft Foundry VS Code 拡張機能を使用して、エージェントをプログラムで操作し、コードから操作する方法を確認します。

### VS Code 拡張機能をインストールして構成する

Foundry の拡張機能を既にインストールしている場合は、このセクションをスキップできます。

1. ローカル コンピューターで Visual Studio Code を開きます。

1. 左側のペインから **[拡張機能]** を選択します (または **Ctrl + Shift + X** キーを押します)。

1. 検索バーに「**Microsoft Foundry**」と入力し、Enter キーを押します。

1. Microsoft から **[Microsoft Foundry]** 拡張機能を選択し、**[インストール]** をクリックします。

1. インストールが完了したら、拡張機能が左側のプライマリ ナビゲーション バーに表示されていることを確認します。

### Foundry プロジェクトに接続する

1. VS Code のサイド バーで、**[Microsoft Foundry]** 拡張機能のアイコンを選択します。

1. [リソース] ビューで、**[Azure にサインイン...]** を選択し、認証プロンプトに従います。

    > **注**:既にサインインしている場合、このオプションは表示されません。

1. サインインしたら、[リソース] ビューでサブスクリプションを展開します。

1. Foundry リソースを見つけて展開し、前に作成したプロジェクト (`it-support-agent-project`) を見つけます。

1. プロジェクトを右クリックし **[アクティブなプロジェクトとして設定]** を選択します。

1. [リソース] ビューでプロジェクトを展開し、**[エージェント]** の下に `it-support-agent` が表示されていることを確認します。

### Python アプリケーションを作成する

次に、エージェントとプログラムで対話する Python アプリケーションを作成しましょう。

1. VS Code で、コマンド パレット (**Ctrl + Shift + P** または **[表示] > [コマンド パレット]**) を開きます。

1. 「**Git:Clone**」と入力してこれを一覧から選択します。

1. リポジトリの URL を入力します。

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. リポジトリをクローンするローカル コンピューター上の場所を選択します。

1. プロンプトが表示されたら **[開く]** を選択して、VS Code でクローンされたリポジトリを開きます。

1. リポジトリが開いたら、**[ファイル] > [フォルダーを開く]** を選択し、`mslearn-ai-agents/Labfiles/01-build-agent-portal-and-vscode/Python` に移動して、**[フォルダーの選択]** をクリックします。

1. エクスプローラー ペインで、`agent_with_functions.py` ファイルを開きます。 今は空であることがわかります。

1. 次のコードを  ファイルに追加します。

    ```python
    import os
    from azure.ai.projects import AIProjectClient
    from azure.identity import DefaultAzureCredential
    import base64
    from pathlib import Path
    
    
    def save_image(image_data, filename):
        """Save base64 image data to a file."""
        output_dir = Path("agent_outputs")
        output_dir.mkdir(exist_ok=True)
        
        filepath = output_dir / filename
        
        # Decode and save the image
        image_bytes = base64.b64decode(image_data)
        with open(filepath, 'wb') as f:
            f.write(image_bytes)
        
        return str(filepath)
    
    
    def main():
        # Initialize the project client
        project_endpoint = os.environ.get("PROJECT_ENDPOINT")
        agent_name = os.environ.get("AGENT_NAME", "it-support-agent")
        
        if not project_endpoint:
            print("Error: PROJECT_ENDPOINT environment variable not set")
            print("Please set it in your .env file or environment")
            return
        
        print("Connecting to Microsoft Foundry project...")
        credential = DefaultAzureCredential()
        project_client = AIProjectClient(
            credential=credential,
            endpoint=project_endpoint
        )
        
        # Get the OpenAI client for Responses API
        openai_client = project_client.get_openai_client()
        
        # Get the agent created in the portal
        print(f"Loading agent: {agent_name}")
        agent = project_client.agents.get(agent_name=agent_name)
        print(f"Connected to agent: {agent.name} (id: {agent.id})")
        
        # Create a conversation
        conversation = openai_client.conversations.create(items=[])
        print(f"Conversation created (id: {conversation.id})")
        
        # Chat loop
        print("\n" + "="*60)
        print("IT Support Agent Ready!")
        print("Ask questions, request data analysis, or get help.")
        print("Type 'exit' to quit.")
        print("="*60 + "\n")
        
        while True:
            user_input = input("You: ").strip()
            
            if user_input.lower() in ['exit', 'quit', 'bye']:
                print("Goodbye!")
                break
            
            if not user_input:
                continue
            
            # Add user message to conversation
            openai_client.conversations.items.create(
                conversation_id=conversation.id,
                items=[{"type": "message", "role": "user", "content": user_input}]
            )
            
            # Get response from agent
            print("\n[Agent is thinking...]")
            response = openai_client.responses.create(
                conversation=conversation.id,
                extra_body={"agent": {"name": agent.name, "type": "agent_reference"}},
                input=""
            )
            
            # Display response
            if hasattr(response, 'output_text') and response.output_text:
                print(f"\nAgent: {response.output_text}\n")
            elif hasattr(response, 'output') and response.output:
                # Extract text from output items
                image_count = 0
                for item in response.output:
                    if hasattr(item, 'text') and item.text:
                        print(f"\nAgent: {item.text}\n")
                    elif hasattr(item, 'type'):
                        # Handle other output types like images from code interpreter
                        if item.type == 'image':
                            image_count += 1
                            filename = f"chart_{image_count}.png"
                            
                            # Download and save the image
                            if hasattr(item, 'image') and hasattr(item.image, 'data'):
                                filepath = save_image(item.image.data, filename)
                                print(f"\n[Agent generated a chart - saved to: {filepath}]")
                            else:
                                print(f"\n[Agent generated an image]")
                        elif item.type == 'file':
                            print(f"\n[Agent created a file]")
    
    
    if __name__ == "__main__":
        main()
    ```

### 環境を構成してアプリケーションを実行する

1. エクスプローラー ペインに、フォルダーに既に存在する `.env.example` と `requirements.txt` のファイルが表示されます。

1. `.env.example` ファイルを複製し、名前を `.env` に変更します。

1. `.env` ファイルで、`your_project_endpoint_here` を実際のプロジェクト エンドポイントに置き換えます。

    ```
    PROJECT_ENDPOINT=<your_project_endpoint>
    AGENT_NAME=it-support-agent
    ```
    
    **プロジェクト エンドポイントを取得するには:** VS Code で、**[Microsoft Foundry]** 拡張機能を開き、アクティブなプロジェクトを右クリックし、**[エンドポイントのコピー]** を選択します。

1. `.env` ファイルを保存 (**Ctrl + S** または **[ファイル] > [保存]**) します。

1. VS Code でターミナルを開きます (**[ターミナル] > [新しいターミナル]**)。

1. 必要なパッケージをインストールします。

    ```bash
    pip install -r requirements.txt
    ```

1. アプリケーションを実行します。

    ```bash
    python agent_with_functions.py
    ```

### コード インタープリターを使用してエージェントをテストする

エージェントが起動したら、次のプロンプトを試して、さまざまな機能をテストしてください。

1. ファイル検索を使用してポリシー検索をテストする:
    ```
    What's the policy for password resets?
    ```

2. コード インタープリターを使用してデータ分析を要求する:
    ```
    Analyze the system performance data and identify any periods where CPU usage exceeded 80%
    ```

3. 視覚化を要求する:
    ```
    Create a line chart showing memory usage trends over time
    ```

4. 統計分析を依頼する:
    ```
    What are the average, minimum, and maximum values for disk usage in the performance data?
    ```

5. 結合分析:
    ```
    Find any correlation between high CPU usage and memory usage in the performance data
    ```

エージェントがどのようにファイル検索 (ポリシーの質問用) とコード インタープリター (データ分析用) の両方を使用して要求を処理するかを確認してください。 コード インタープリターは CSV データを分析し、計算を実行し、さらに視覚化を生成することもできます。 テストが完了したら、「`exit`」と入力します。

---

## ポータルとコードの比較:各アプローチをどのようなときに使用するか

両方のアプローチを使用したので、それぞれをどのようなときに使用するかに関するガイダンスを示します。

### ポータルは、次の場合に使用します。
- エージェント構成の迅速なプロトタイプ作成とテスト
- 指示とシステム プロンプトの簡易的な調整
- 典拠データと組み込みツールを使用したテスト
- 利害関係者への概念のデモンストレーション
- コードを記述せずに簡易的なエージェントが必要

### VS Code または SDK は、次の場合に使用します。
- 運用アプリケーションの構築
- エージェントと既存のコードおよびシステムの統合
- プログラムによる会話と応答の管理
- バージョン管理と CI/CD パイプライン
- 高度なオーケストレーションとマルチエージェントのシナリオ
- プログラムによる大規模なエージェント管理

### ハイブリッドのアプローチ (ベスト プラクティス):
1. 概念を検証するためのポータルでの**プロトタイプ作成**
2. 運用環境の実装のための VS Code での**開発**
3. 両方のツールを使用した**監視と反復処理**

---

## クリーンアップ

不要な Azure 料金を回避するため、作成したリソースを削除してください。

1. Foundry ポータルで、プロジェクトに移動します
1. **[設定]** > **[プロジェクトの削除]** を選択します
1. または、Azure portal からリソース グループ全体を削除します

---

## トラブルシューティング

### 一般的な問題

**問題点**:"プロジェクト エンドポイントが無効です"
- **解決策**:ポータルから完全なプロジェクト エンドポイントをコピーしたことを確認してください。 `https://` で始まり、プロジェクトの詳細が含まれている必要があります。

**問題点**:"エージェントが見つかりません"
- **解決策**:VS Code 拡張機能で正しいプロジェクトをアクティブに設定していることを確認してください。

**問題点**:"コード インタープリターが視覚化を生成しない"
- **解決策**:CSV ファイルがエージェントに正しくアップロードされ、エージェント設定でコード インタープリターが有効になっていることを確認してください。

---

## まとめ

この演習では、以下のことを行います。

典拠データを含む AI エージェントを Microsoft Foundry ポータルで作成しました  
ファイル検索やコード インタープリターなどの組み込みツールを有効にしました  
VS Code 拡張機能を使用してプロジェクトに接続しました  
Python を使用してプログラムでエージェントと対話しました  
データ分析と視覚化でコード インタープリターを活用しました  
ポータルとコードベースのアプローチをそれぞれどのようなときに使用するかについて学習しました  

ビジュアルとコードベースの両方のワークフローを使用して AI エージェントを構築するための基本的なスキルを取得しました。

## 次のステップ

エージェントの開発スキルを次のレベルに引き上げましょう。 続行:

- **ラボ 2:高度なツール呼び出し** - 動的データ処理に高度なツール呼び出しを使用する方法、高度な非同期関数パターンを実装する方法、バッチ処理を使用したマスター ファイル操作について学習します。

### その他のリソース

- [Azure AI エージェント サービスのドキュメント](https://learn.microsoft.com/azure/ai-services/agents/)
- [Microsoft Foundry VS Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai)
- [Azure AI Projects SDK](https://learn.microsoft.com/python/api/overview/azure/ai-projects-readme)
