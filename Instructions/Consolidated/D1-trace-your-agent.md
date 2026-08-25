---
title: タスク 1 – エージェントをトレースする
lab:
  title: タスク 1 – エージェントをトレースする
  description: OpenTelemetry でエージェントをインストルメント化し、トレースを Azure Monitor にエクスポートし、自分のスパンや属性を追加し、Foundry ポータルで結果を読み取ります。
  type: task
  parent: D
  order: 1
  section: core
  difficulty: 3
  duration: 25
  access: open
  level: 300
  concepts: 'tracing, OpenTelemetry, Azure Monitor, Application Insights'
  islab: true
  status: draft
---

# タスク 1 — エージェントをトレースする

*「**エージェントを観察し、評価し、安全に保つ**」のラボの一部です。初めての場合は、まずは「[はじめに](D0-getting-started.md)」から開始してください。*

> **「セットアップ」(ここから開始する):** このタスクには Foundry プロジェクト、**それに接続する Application Insights リソース**、そしてスタート コードが必要です。 まだの場合は、「[はじめに](D0-getting-started.md)」を完了してプロジェクトを作成し、Application Insights を接続し、コードをクローンし、`PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を `Python/.env` で設定してください。 次に、VS Code で開いた `Python` フォルダーから、次の準備ができているかを確認してください。

```
python ../setup/check_env.py --task 1
```

> **前のタスクから続けていますか?** 同じ `Python` フォルダーの別のタスクを終えたばかりで、プロジェクト、仮想環境、`.env` が既に設定されているなら、下の「**エージェントをインストルメント化する**」に直接アクセスしてください。

---

月曜日の朝、Caldova の Ashford 拠点でのことです。 プランナーが会議の合間にアシスタントに質問を浴びせ、計画リードは一部の応答に「時間がかかる」と言っています。 どの応答なのか、なぜそうなるのか、まったくわかりません。ターミナルは応答を示しますが、どうやってその応答にたどり着いたかは何も示さないのです。

**トレース**が、それを解決します。 あなたのコードは、作業記録が時間と名前付きで入れ子になった**スパン**を出力し、それを **Application Insights** に送ります。そこでは Foundry ポータルがウォーターフォール形式でレンダリングするので、順番に確認することができます。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>OpenTelemetry とは何ですか?</summary>
<div class="concept-body" markdown="1">

**OpenTelemetry** は、トレース、指標、ログの送信に関するベンダー中立の標準です。 **スパン**は、開始、終了、属性を含む 1 つの作業単位です。スパンは入れ子になって操作全体の**トレース**を形成します。 これは、標準であるため、Azure SDK、OpenAI クライアント、独自のコードはすべて同じウォーターフォールに並ぶスパンを生成します。そのため、インストルメンテーションを書き直さずに明日は別のバックエンドに指定することができます。

</div>
</details>

> **サーバー側のトレースは無料です。** Application Insights がプロジェクトに接続されたことで、Foundry は既にホストするエージェントのトレースを記録しているため、コードは不要です。 ここで追加するのは**クライアント側の**インストルメンテーション、つまり*あなたの*コードを囲むスパンなので、あなたのロジックとエージェントの作業を 1 つのタイムラインで確認できます。

`Python` フォルダーを開き、「[はじめに](D0-getting-started.md)」から仮想環境をアクティブ化し (`.\labenv\Scripts\Activate.ps1`)、下のページを進めてください。

### エージェントをインストルメント化する

**traced_agent.py** を開き、コメント付きの各プレースホルダーにコードを追加します。

> **ヒント**: コードを追加する際は、コメントとインデントを揃えておきましょう。

1. **参照を追加する**:

    ```python
    # Add references
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.projects.models import PromptAgentDefinition
    from azure.monitor.opentelemetry import configure_azure_monitor
    from opentelemetry import trace
    ```

1. **生成 AI トレースをオンにする** — モデル呼び出しをキャプチャするスパンは既定ではオフになり、メッセージ内容も別途オフにされます。プロンプトには個人データが含まれている可能性があるためです。 このラボでは両方ともオンにしてください。

    ```python
    # Turn on GenAI tracing
    os.environ.setdefault("AZURE_EXPERIMENTAL_ENABLE_GENAI_TRACING", "true")
    os.environ.setdefault("OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT", "true")
    ```

    > これらはクライアントが作成される**前**に設定しなければならいため、ファイルの上部に表示されます。 運用環境では、メッセージ内容をオンにする前によく考えてください。

1. **プロジェクトに接続する**:

    ```python
    # Connect to the project
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
        project_client.get_openai_client() as openai_client,
    ):
    ```

1. **Application Insights の接続文字列を読み、トレースのエクスポートを開始する** — プロジェクトはセットアップ時に接続したリソースの接続文字列を渡し、`configure_azure_monitor` がエクスポーターを接続します。

    ```python
    # Read the Application Insights connection string and start exporting traces
    try:
        connection_string = project_client.telemetry.get_application_insights_connection_string()
    except Exception as error:
        raise SystemExit(
            "Could not read an Application Insights connection string from this project.\n"
            "In the Foundry portal, open your project, select Agents > Traces, and select\n"
            f"Connect to create or connect an Application Insights resource.\n\nDetails: {error}"
        )
    configure_azure_monitor(connection_string=connection_string)
    ```

1. **このスクリプトのトレーサーを入手する** — トレーサーとは、自分のスパンを作成するためのものです。

    ```python
    # Get a tracer for this script
    tracer = trace.get_tracer(__name__)
    ```

1. **スタッフが話しかけるエージェントを作成する**:

    ```python
    # Create the agent staff are talking to
    agent = project_client.agents.create_version(
        agent_name=AGENT_NAME,
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions=INSTRUCTIONS,
        ),
    )
    print(f"Agent created (name: {agent.name}, version: {agent.version})")
    ```

1. **自分のスパン内でそれぞれの質問を投げかける** — この部分が役に立ちます。 外側のスパンはレビューを表します。各質問は子スパンを取得し、あなたが選択した属性がタグ付けされているので、ポータル内で区別できます。

    ```python
    # Ask each question inside its own span
    with tracer.start_as_current_span("morning-planning-review") as shift_span:
        shift_span.set_attribute("caldova.site", "ashford")
        conversation = openai_client.conversations.create()

        for number, question in enumerate(QUESTIONS, start=1):
            with tracer.start_as_current_span("planner-question") as question_span:
                question_span.set_attribute("caldova.question_number", number)
                response = openai_client.responses.create(
                    conversation=conversation.id,
                    input=question,
                    extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
                )
                question_span.set_attribute("caldova.answer_length", len(response.output_text))
                print(f"\nQ{number}: {question}")
                print(f"A{number}: {response.output_text}")
    ```

1. **エージェントのバージョンをクリーンアップして**、テスト エージェントを忘れないようにしましょう。

    ```python
    # Clean up resources by deleting the agent version
    project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
    print("\nAgent deleted")
    ```

1. ファイルを保存します (**Ctrl キーを押しながら S キーを押します**)。

### 実行してテストする

1. ターミナルにサインインし、アプリを実行します。

    ```
    az login
    ```

    ```
    python traced_agent.py
    ```

1. 3 つの応答が印刷された後、エージェントが自身を削除するのを確認できるはずです。

    ```
    Agent created (name: caldova-planning-assistant, version: 1)

    Q1: How long does review take for a capacity request with a complete brief?
    A1: ...

    Agent deleted
    ```

    > 接続文字列に関するエラーが表示された場合は、Application Insights がまだプロジェクトに接続されていません。「[はじめに](D0-getting-started.md)」に戻って接続してください。

### トレースを読み取る

1. [Foundry ポータル](https://ai.azure.com)でプロジェクトを開き、**[エージェント]**、**[トレース]** の順に選択します。

1. 最新のトレースを探して選択します。 テレメトリは到着までに 1 〜 2 分かかります。表示されていない場合は、少し待ってから更新してください。

1. スパンをステップ実行します。 上部の `morning-planning-review` スパンに `planner-question` の 3 つの子と、それぞれの内部に SDK が出力したモデル呼び出しが表示されているはずです。

1. `planner-question` スパンを選択し、その属性を確認します。 `caldova.question_number` および `caldova.answer_length` は標準の生成 AI の属性とともに存在します。

1. 3 つの質問の時間の長さを比較してください。 それが計画リードの苦情であり、データで答えが出ました。スパンの内訳から、*どの部分が*特に遅いかがわかります。

> **お試しください**: 4 問目として、より難しい質問を `QUESTIONS` に追加してもう一度試してみてください。 追加時間はモデル呼び出しで表示されるでしょうか、それとも別の場所で表示されるでしょうか?

> ✅**チェックポイント**: 実行中のエージェントの内部を見ることができます — SDK の自分のスパンと、ご自分で選んだ属性を持つカスタム スパンの両方が、すべて 1 つのタイムラインに収まっています。

完了したら、`deactivate` と入力して、仮想環境を終了します。

---

**次:** [タスク 2 — 応答の質を評価する](D2-evaluate-answer-quality.md)
