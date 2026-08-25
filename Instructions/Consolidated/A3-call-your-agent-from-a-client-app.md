---
title: タスク 3 - クライアント アプリからエージェントを呼び出す
lab:
  title: タスク 3 - クライアント アプリからエージェントを呼び出す
  description: Foundry SDK と Responses API を使用して小さな Web チャット アプリから、グラウンディングされたポータル エージェントを駆動します。これにはインライン チャートも含まれます。
  type: task
  parent: A
  order: 3
  section: optional
  difficulty: 3
  duration: 20
  access: open
  level: 300
  concepts: 'Foundry SDK, Responses API, code interpreter'
  status: draft
---

# タスク 3 - クライアント アプリからエージェントを呼び出す

"「**AI エージェントを構築および拡張する**」ラボの一部です。ここから始める場合は、まず「[作業の開始](A0-getting-started.md)」を完了してください。"**

> **設定 (ここから始める場合):** このタスクには Foundry プロジェクトとスタート コードが必要です。 「[作業の開始](A0-getting-started.md)」をまだ完了していない場合は完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` を設定してください。

このタスクでは、**グラウンディングされたエージェント**を駆動します。 これを入手する最も簡単な方法は、コードで作成することです。VS Code で開いた `Python` フォルダーから、次のコマンドを実行します。

```
python ../setup/bootstrap_agent.py
```

これにより、`caldova-agent` が作成され、グラウンディングされます。出力データが既に添付されている**コード インタープリター** ツールも含まれます。さらに、`AGENT_NAME` が `.env` に書き込まれます。 この後、準備ができていることを確認します。

```
python ../setup/check_env.py --task 3
```

> **[タスク 1](A1-create-and-ground-an-agent.md) でエージェントを既に作成している場合**  スクリプトの代わりにそれを使用します。ポータルで `caldova-agent` を開き、出力データが添付された**コード インタープリター** ツールを追加し (下記の手順 1)、`.env` で `AGENT_NAME=caldova-agent` を設定します。

---

**目標**: プレイグラウンドではなく、小さな **Web チャット アプリ**から、グラウンディングされたポータル エージェントを操作します。これには、エージェントによって (コード インタープリターから) 生成されたチャートも含まれ、それらはチャット ウィンドウ内に**インライン**でレンダリングされます。

**概念の強化**: Foundry SDK を使用してプログラムでエージェントを使用し、既存のエージェントを名前で読み込み、Responses API を使用して駆動します。 提供された UI シェル (`caldova_ui.py`) によって、エージェントはブラウザー チャット アプリに変わるため、インターフェイスではなくエージェント コードに集中できます。

**設定:**

上記の `python ../setup/bootstrap_agent.py` を実行した場合は、エージェント、その**コード インタープリター** ツール、`AGENT_NAME` は既に構成されています。仮想環境 (`.\labenv\Scripts\Activate.ps1`) をアクティブにし、「**まず試してみる**」に進みます。

**タスク 1 でエージェントを自分で構築した場合**、その接続を完了します。

1. ポータルで `caldova-agent` を開き、**コード インタープリター** ツールを追加し、分析対象となるデータ ファイルをアップロードします。 ダウンロードして添付します。

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/A-build-and-extend-ai-agents/Python/weekly_output.csv
    ```

    エージェントを保存します。

1. `Labfiles/A-build-and-extend-ai-agents/Python` フォルダーで、仮想環境 (`.\labenv\Scripts\Activate.ps1`) をアクティブにします。 次に、**.env** を開き、既に設定した `PROJECT_ENDPOINT` と共に `AGENT_NAME=caldova-agent` を追加します。 ファイルを保存します。

> **まず試してみる**: `agent_with_functions.py` ファイルには既に、Web チャット ウィンドウを起動する完全なクライアントが含まれています。 それを実行する前に、既存のポータル エージェントを "名前で" 読み込む SDK 呼び出し、** そのエージェントを使用するようにクライアントから Responses API に指示する方法、 `respond()` 関数によって、1 つのメッセージが UI で表示できる返答に変換される方法を予測します。

<details markdown="1">
<summary>ソリューションを表示する</summary>

提供された `agent_with_functions.py` には既にクライアントが実装されており、その `respond()` 関数が共有の `run_chat_app()` シェルに渡されます。 重要な行は次のとおりです。

1. **ポータル エージェントを名前で読み込みます** (**.env** の `AGENT_NAME` を使用)。

    ```python
    agent = project_client.agents.get(agent_name=agent_name)
    ```

2. Responses API (`respond()`内) を使用して、**各要求をそのエージェントにルーティングします**。

    ```python
    response = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        input="",
    )
    ```

3. **インライン チャート**: ヘルパー関数は、画像出力や `container_file_citation` 注釈を検出し、それらを `agent_outputs/` の下に保存し、`AgentReply` で返して、UI がチャット内にそれらを**インライン**でレンダリングできるようにします。

4. **アプリを起動します**: ファイルは、ブラウザー チャット ウィンドウを起動して終了します。

    ```python
    run_chat_app(respond, title="Caldova Supply Chain Assistant")
    ```

サインインして実行します。

```
az login
python agent_with_functions.py
```

ブラウザーで、`http://localhost:7860` のチャット ウィンドウが開きます。 コード インタープリターを使用する作業を依頼します。

```
Analyze the weekly sales data and create a chart of revenue over time.
```

エージェントの分析がチャットに表示され、**チャートがインラインで表示されます**。 ブラウザー タブを閉じ、ターミナルで **Ctrl + C** キーを押してアプリを停止します。

</details>

**拡張**: 各応答後にエージェントのトークン使用状況を表示します。

---

**次 (省略可能):** [タスク 4 - カスタム関数ツールを追加する](A4-add-custom-function-tools.md)
