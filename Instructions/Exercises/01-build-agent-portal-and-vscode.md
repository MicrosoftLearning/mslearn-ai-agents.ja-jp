---
lab:
  title: ポータルと VS Code を使用して AI エージェントを構築する
  description: Microsoft Foundry ポータルと Foundry Toolkit VS Code 拡張機能の両方を使用して、ファイル検索やコード インタープリターなどの組み込みツールを含む AI エージェントを作成します。
  level: 300
  duration: 45
  islab: true
---

# ポータルと VS Code を使用して AI エージェントを構築する

この演習では、Microsoft Foundry ポータルと Foundry Toolkit VS Code 拡張機能の両方を使用して、完全な AI エージェント ソリューションを構築します。 まず、ポータルで典拠データと組み込みツールを含む基本的なエージェントを作成してから、VS Code を使用してプログラムで操作し、コード インタープリターなどの高度な機能をデータ分析に使用します。

この演習は約 **45** 分かかります。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

この演習を開始するには、以下のものが必要です。

- Azure AI リソースをプロビジョニングするための十分なアクセス許可とクォータを持つ [Azure サブスクリプション](https://azure.microsoft.com/free/)
- ローカル コンピューターにインストールされている [Visual Studio Code](https://code.visualstudio.com/)
- [Python 3.13](https://www.python.org/downloads/) 以降がインストールされている
- ローカル コンピューターにインストールされている [Git](https://git-scm.com/downloads)
- Azure AI サービスと Python プログラミングに関する基本的な知識

> \* Python 3.13 を使用できますが、一部の依存関係がそのリリース用にまだコンパイルされていません。 このラボでは Python 3.13.12 でテストが正常に終了しました。

## Microsoft Foundry プロジェクトを作成する

Microsoft Foundry では "プロジェクト" を使って、AI ソリューションの開発に使われるモデル、リソース、データ、その他の資産を整理します。

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインすると開くヒントやクイック スタートのペインをすべて閉じ、必要な場合は、左上にある **Foundry** のロゴを使ってホーム ページに移動します。

    > **重要**: このラボでは、**新しい** Foundry エクスペリエンスを使用しています。

1. 上部のバナーで **[構築の開始]** を選択して、新しい Microsoft Foundry エクスペリエンスをお試しください。

1. プロンプトが表示されたら、**新規**プロジェクトを作成し、プロジェクトに有効な名前 (`it-support-agent-project`など) を入力します。

1. **[詳細オプション]** を展開し、次の設定を指定します。
    - **Microsoft Foundry リソース**:"Foundry リソースの有効な名前"**
    - **リージョン**: 近くから利用可能なものを選択します**\**
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *お手持ちのリソース グループを選択するか、新しいリソース グループを作成します*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。

2. プロジェクトが作成されると、ウェルカム ダイアログが表示される場合があります。 **[次へ]** を選択してウェルカム メッセージを読み、**[エージェントの作成]** を選択します。

    ホーム ページで **[ビルドの開始]** を選択し、ドロップダウン メニューから **[エージェントの作成]** を選択することもできます。

3. **[エージェント名]** を `it-support-agent` に設定し、エージェントを作成します。

新しく作成したエージェントのプレイグラウンドが開きます。 使用可能なデプロイ済みモデルが既に選択されていることがわかります。

## エージェントに指示と典拠データを構成する

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

1. エージェントのプレイグラウンドに戻ります。 **[ツール]** セクションで **[追加]** を選択し、**[ファイル検索]** と **[</> コード インタープリター]** の両方を追加します。

1. **[追加]** の右側にある **[ファイルのアップロード]** を選択します。 **[ファイルの添付]** で、ダウンロードした `IT_Policy.txt` ファイルを参照してアップロードし、**[添付]** を選択します。

1. ファイルのインデックスが作成されるまで待ちます。 準備ができたら確認が表示されます。

1. 次に、分析するコード インタープリターのパフォーマンス データをいくつか追加しましょう。 次の場所からシステム パフォーマンス データ ファイルをダウンロードします。

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/system_performance.csv
    ```

    このファイルをローカル コンピューターに保存します。

1. **[</> コード インタープリター]** の右側にある **[+ ファイル]** を選択し、ダウンロードした `system_performance.csv` ファイルをアップロードします。

    > **注**:この CSV ファイルには、エージェントが分析できるシミュレート済みの経時的システム メトリック (CPU、メモリ、ディスク使用量) が含まれています。

1. エージェントを保存します。

## エージェントをテストする

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

## VS Code を使用してエージェントと対話する

開発者は、Foundry ポータルである程度の時間作業しますが、Visual Studio Code でも多くの時間を費やす可能性があります。 Foundry Toolkit for VS Code 拡張機能は、開発環境を離れることなく Foundry プロジェクト リソースを操作するための便利な方法を提供します。

### VS Code 拡張機能をインストールして構成する

Foundry Toolkit 拡張機能を既にインストールしている場合は、このセクションをスキップできます。

1. Visual Studio Code を開きます。

2. 左側のペインから **[拡張機能]** を選択します (または **Ctrl + Shift + X** キーを押します)。

3. 拡張機能マーケットプレースで Microsoft の `Foundry Toolkit for VS Code` 拡張機能を検索し、**[インストール]** を選択します。

    Foundry Toolkit 拡張機能をインストールすると、AI Toolkit 拡張機能が VS Code に追加されます。

    > **注**: 拡張機能は現在、**Foundry Toolkit** として登録されていますが、一部の VS Code のラベル、コマンド、または以前のスクリーンショットでは、引き続き **AI Toolkit** と表示されている場合があります。 このラボでは、これらの名前が同じ拡張機能エクスペリエンスを表しているものとして扱います。

4. 拡張機能をインストールしたら、サイド バーで AI Toolkit アイコンを選択します。 

    Azure アカウントへのサインインがまだの場合は、プロンプトが表示されるはずです。

### VS Code でエージェントをテストする

コードを記述する前に、拡張機能インターフェイスでエージェントを直接操作できます。

1. **[Microsoft Foundry リソース]** で、**[既定のプロジェクトの設定]** を選択します

    既定のプロジェクトが既にアクティブな場合、そのプロジェクト名はリソースの一覧に表示されます。 別のプロジェクトを選択するには、同じ **[プロジェクトの選択]** アイコンを選択します。

2. [プロジェクト] セクションを展開します。 **[プロンプト エージェント]** の下に、ポータルで作成した `it-support-agent` が表示されるはずです。 エージェント名を選択して、エージェント ビルダー インターフェイスを開きます。

    エージェント プレイグラウンドがエージェント ビルダー インターフェイスに表示されるため、VS Code から離れることなくエージェントを操作し、その設定を構成できるようになります。

3. プレイグラウンドの [チャット] ペインで、次のような質問を入力します。

    ```
    What is the policy for reporting a lost or stolen device?
    ```

4. エージェントの応答を確認します。 これは、前にアップロードしたグラウンディング データを使用して、関連する IT ポリシー情報を提供するものである必要があります。

    > **ヒント**: この組み込みのプレイグラウンドを使用すると、コードを記述することなく、エージェントの指示とナレッジをすばやくテストできます。

## エージェントと対話するクライアント アプリケーションを作成する

次に、エージェントとプログラムで対話するクライアント アプリケーションを作成しましょう。

1. VS Code で、コマンド パレット (**Ctrl + Shift + P** または **[表示] > [コマンド パレット]**) を開きます。

1. 「**Git:Clone**」と入力してこれを一覧から選択します。

1. リポジトリの URL を入力します。

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. リポジトリをクローンするローカル コンピューター上の場所を選択します。

1. プロンプトが表示されたら **[開く]** を選択して、VS Code でクローンされたリポジトリを開きます。

1. リポジトリが開いたら、**[ファイル] > [フォルダーを開く]** を選択し、`mslearn-ai-agents/Labfiles/01-build-agent-portal-and-vscode/Python` に移動して、**[フォルダーの選択]** を選択します。

1. エクスプローラー ペインで、`agent_with_functions.py` ファイルを開きます。 ファイルが空の場合は、その内容を次のコードに置き換えます。

1. 次のコードを使用します。

    ```python
    import base64
    import os
    from pathlib import Path

    from azure.ai.projects import AIProjectClient
    from azure.identity import DefaultAzureCredential
    from dotenv import load_dotenv
    
    
    OUTPUT_DIR = Path("agent_outputs")
    
    
    def get_output_path(filename):
        """Create a unique path for generated files."""
        OUTPUT_DIR.mkdir(exist_ok=True)
        file_name = Path(filename).name
        stem = Path(file_name).stem or "output"
        suffix = Path(file_name).suffix
        output_path = OUTPUT_DIR / file_name

        counter = 1
        while output_path.exists():
            output_path = OUTPUT_DIR / f"{stem}_{counter}{suffix}"
            counter += 1

        return output_path


    def save_bytes(file_bytes, filename):
        """Save binary content to a local file."""
        output_path = get_output_path(filename)
        with open(output_path, "wb") as file_handle:
            file_handle.write(file_bytes)
        return output_path


    def save_image(image_data, filename):
        """Save base64 image data to a file."""
        return save_bytes(base64.b64decode(image_data), filename)


    def download_container_file(openai_client, annotation, downloaded_files):
        """Download a cited container file once and return its local path."""
        cache_key = (annotation.container_id, annotation.file_id)
        if cache_key in downloaded_files:
            return downloaded_files[cache_key]

        file_content = openai_client.containers.files.content.retrieve(
            file_id=annotation.file_id,
            container_id=annotation.container_id,
        )
        output_path = save_bytes(
            file_content.read(),
            annotation.filename or f"{annotation.file_id}.bin",
        )
        downloaded_files[cache_key] = output_path
        return output_path


    def format_output_text(content_item, openai_client, downloaded_files):
        """Replace sandbox file citations with local file paths."""
        text = content_item.text or ""
        replacements = []
        referenced_files = set()

        for annotation in content_item.annotations or []:
            if getattr(annotation, "type", "") != "container_file_citation":
                continue

            output_path = download_container_file(openai_client, annotation, downloaded_files)
            replacement_text = f"{annotation.filename} (saved to {output_path})"
            referenced_files.add(output_path)

            start_index = getattr(annotation, "start_index", None)
            end_index = getattr(annotation, "end_index", None)
            if start_index is not None and end_index is not None:
                replacements.append((start_index, end_index, replacement_text))
                continue

            annotated_text = getattr(annotation, "text", "")
            if annotated_text:
                text = text.replace(annotated_text, replacement_text)

        for start_index, end_index, replacement_text in sorted(replacements, reverse=True):
            text = f"{text[:start_index]}{replacement_text}{text[end_index:]}"

        return text, referenced_files
    
    
    def main():
        # Initialize the project client
        load_dotenv()
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
                extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
                input=""
            )

            # Display response and save any generated files locally
            handled_output = False
            downloaded_files = {}
            referenced_files = set()
            image_count = 0

            if hasattr(response, "output") and response.output:
                for item in response.output:
                    item_type = getattr(item, "type", "")

                    if item_type == "message" and getattr(item, "content", None):
                        for content_item in item.content:
                            if getattr(content_item, "type", "") != "output_text":
                                continue

                            formatted_text, message_files = format_output_text(
                                content_item,
                                openai_client,
                                downloaded_files,
                            )
                            referenced_files.update(message_files)

                            if formatted_text:
                                print(f"\nAgent: {formatted_text}\n")
                                handled_output = True

                    elif hasattr(item, "text") and item.text:
                        print(f"\nAgent: {item.text}\n")
                        handled_output = True

                    elif item_type == "image":
                        image_count += 1
                        filename = f"chart_{image_count}.png"

                        if hasattr(item, "image") and hasattr(item.image, "data"):
                            file_path = save_image(item.image.data, filename)
                            print(f"\n[Agent generated a chart - saved to: {file_path}]")
                        else:
                            print("\n[Agent generated an image]")
                        handled_output = True

                for file_path in downloaded_files.values():
                    if file_path not in referenced_files:
                        print(f"\n[Agent generated a file - saved to: {file_path}]")
                        handled_output = True

            if not handled_output and hasattr(response, "output_text") and response.output_text:
                print(f"\nAgent: {response.output_text}\n")

    if __name__ == "__main__":
        main()
    ```
    
1. `agent_with_functions.py` ファイルを保存 (**Ctrl + S** または **[ファイル] > [保存]**) します。

### 環境を構成してアプリケーションを実行する

1. エクスプローラー ペインに、フォルダーに既に存在する `.env.example` と `requirements.txt` のファイルが表示されます。

1. `.env.example` ファイルを複製し、名前を `.env` に変更します。

1. `.env` ファイルで、`your_project_endpoint_here` を実際のプロジェクト エンドポイントに置き換えます。

    ```
    PROJECT_ENDPOINT=<your_project_endpoint>
    AGENT_NAME=it-support-agent
    ```

    **プロジェクト エンドポイントを取得するには:** VS Code で、**[Foundry Toolkit]** 拡張機能を開き、アクティブなプロジェクトを右クリックし、**[エンドポイントのコピー]** を選択します。 インストールされているバージョンの Foundry Toolkit で **[エンドポイントのコピー]** を使用できない場合は、代わりに、Microsoft Foundry ポータルを開いてプロジェクトに移動し、プロジェクトの概要ページからプロジェクト エンドポイントをコピーします。

1. `.env` ファイルを保存 (**Ctrl + S** または **[ファイル] > [保存]**) します。

1. VS Code でターミナルを開きます (**[ターミナル] > [新しいターミナル]**)。

1. 必要なパッケージをインストールして、ログインします。

    ```bash
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

    ```bash
    az login
    ```

1. アプリケーションを実行します。

    ```bash
    python agent_with_functions.py
    ```

## クライアント アプリケーションをテストする

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

    アプリケーションにより、生成されたグラフと引用ファイルが `agent_outputs` フォルダーに保存され、ターミナルにローカル ファイル パスが出力されます。

4. 統計分析を依頼する:

    ```
    What are the average, minimum, and maximum values for disk usage in the performance data?
    ```

5. 結合分析:

    ```
    Find any correlation between high CPU usage and memory usage in the performance data
    ```

エージェントがどのようにファイル検索 (ポリシーの質問用) とコード インタープリター (データ分析用) の両方を使用して要求を処理するかを確認してください。 コード インタープリターは CSV データを分析し、計算を実行し、さらに視覚化を生成することもできます。 テストが完了したら、「`exit`」と入力します。

## クリーンアップ

不要な Azure 料金を回避するため、作成したリソースを削除してください。

1. Foundry ポータルで、プロジェクトに移動します
1. **[設定]** > **[プロジェクトの削除]** を選択します
1. または、Azure portal からリソース グループ全体を削除します
