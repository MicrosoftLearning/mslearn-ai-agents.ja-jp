---
title: タスク 2 – 応答の質を評価する
lab:
  title: タスク 2 – 応答の質を評価する
  description: グラウンディングされたエージェントを、組み込みの根拠性、関連性、類似性のエバリュエータを使用してグラウンド トゥルースのデータセットと比較してスコアリングし、行レベルの結果を読み取ります。
  type: task
  parent: D
  order: 2
  section: core
  difficulty: 3
  duration: 35
  access: open
  level: 300
  concepts: 'evaluation, groundedness, relevance, similarity, ground truth'
  islab: true
  status: draft
---

# タスク 2 — 応答の質を評価する

*「**エージェントを観察し、評価し、安全に保つ**」のラボの一部です。初めての場合は、まずは「[はじめに](D0-getting-started.md)」から開始してください。*

> **「セットアップ」(ここから開始する):** このタスクには Foundry プロジェクト、スタート コード、そして**測定用のグラウンディングされたエージェント**が必要です。 「[はじめに](D0-getting-started.md)」がまだ完了していない場合は完了して、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定し、[ラボ B](B-integrate-agents-with-enterprise-knowledge-and-m365.md) で `AGENT_NAME` を指定するか、`python ../setup/bootstrap_agent.py` を使用して作成してください。 次に、VS Code で開いた `Python` フォルダーから、次の準備ができているかを確認してください。

```
python ../setup/check_env.py --task 2
```

> **前のタスクから続けていますか?** 同じ `Python` フォルダーで別のタスクを終えたばかりの場合、プロジェクト、仮想環境、`.env` は既に設定済みですが、タスク 1 では `AGENT_NAME` が必要なかったため、開始する前に設定されているかを確認してください。

---

Caldova のナレッジ エージェントは、キャパシティ、契約メーカー、サプライヤーに関する質問に答えます。 毎回自信を持っているように聞こえます。 問題は、10 件の応答を読んで問題がないと思っても、11 回目で存在しない返品期間を捏造するかどうかがわからないということです。

**評価**はその感覚を数字に置き換えます。 既に正しい応答を知っている質問のセットをエージェントを通じて実行し、その応答を別のモデルに評価してもらいます。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>根拠性、関連性、類似性とは何でしょうか?</summary>
<div class="concept-body" markdown="1">

応答の間違い方は 3 とおり考えられるので、それぞれ別々に測定されます。

- **根拠性** — 応答が与えられた**文脈**によって裏付けられているか? スコアが低い場合、(たとえそれがたまたま真実であったとしても) そのエージェントが作り話をしたことを意味します。
- **関連性** — 応答が実際に**質問**に答えているか? 応答が完璧にグラウンディングされていても、質問の内容に合っていないことがあります。
- **類似性** — 応答が作成した**グラウンド トゥルース**にどの程度近いか? これは人間が書いた正しい応答を必要とするものです。

それぞれ、判断するモデルが 1 点から 5 点でスコアリングします。 根拠性と関連性では `reason` も返し、これは通常、スコアよりも有用です。

</div>
</details>

`Python` フォルダーを開き、「[はじめに](D0-getting-started.md)」から仮想環境をアクティブ化し (`.\labenv\Scripts\Activate.ps1`)、下のページを進めてください。

### まずはデータセットを見る

**data/caldova_eval.jsonl** を開きます。 各行が 1 つのテスト ケースであり、評価の良し悪しは次のファイルに左右されます。

```json
{"query": "A planner wants to raise a capacity request with a complete program brief. How long does review take?", "context": "Standard Request Window: Requests with a complete program brief: reviewed within 5 business days ...", "ground_truth": "Five business days for a request with a complete program brief ..."}
```

- `query` — あなたがエージェントに尋ねる内容。
- `context` — 応答が根拠とすべき資料。 根拠性はこれに対して評価されます。
- `ground_truth` — 知識のある人間が答えるであろう応答。 類似性はこれに対して評価されます。

エージェント自体の `response` はファイルには含まれていません。評価時に、評価を**対象**で指定して生成します。

**agent_target.py** を開いて読みます — 編集はしません。 これは、1 つの `query` を取得し、エージェントから `{"response": ...}` を返す呼び出し可能なクラスです。 これが、対象が守らなければならない契約のすべてです。

### 評価を記述する

**evaluate_agent.py** を開き、コメント付きの各プレースホルダーにコードを追加します。

1. **参照を追加する**:

    ```python
    # Add references
    from azure.ai.evaluation import (
        AzureOpenAIModelConfiguration,
        GroundednessEvaluator,
        RelevanceEvaluator,
        SimilarityEvaluator,
        evaluate,
    )
    from agent_target import CaldovaAgentTarget
    ```

1. **応答を評価するモデルを設定する** — エバリュエータ自体がモデル呼び出しなので、実行するためのデプロイが必要です。 エージェントが使用しているのと同じものを再利用することになります。

    ```python
    # Configure the model that grades the answers
    model_config = AzureOpenAIModelConfiguration(
        azure_endpoint=evaluator_endpoint(),
        azure_deployment=model_deployment,
        api_version=os.getenv("AZURE_OPENAI_API_VERSION", "2024-10-21"),
    )
    ```

    > API キーなし: `az login` が終わると、エバリュエータがあなたとして認証します。 `evaluator_endpoint()` がファイル上部に指定されている — `PROJECT_ENDPOINT` からリソース エンドポイントが派生するか、`AZURE_OPENAI_ENDPOINT` が設定されている場合は使用されます。

1. **エバリュエータを選択する**:

    ```python
    # Create the evaluators
    groundedness = GroundednessEvaluator(model_config)
    relevance = RelevanceEvaluator(model_config)
    similarity = SimilarityEvaluator(model_config)
    ```

1. **評価を実行する** — `evaluate()` がデータセットを読み取り、行ごとに 1 回対象を呼び出し、各エバリュエータに必要な列を正確に渡します。 `column_mapping` によって、どの列がどれかが決まる: `${data.x}` はファイルから発生し、`${outputs.x}` が対象から戻ってきます。

    ```python
    # Run the evaluation
    result = evaluate(
        data=str(DATASET),
        target=CaldovaAgentTarget(),
        evaluators={
            "groundedness": groundedness,
            "relevance": relevance,
            "similarity": similarity,
        },
        evaluator_config={
            "groundedness": {
                "column_mapping": {
                    "query": "${data.query}",
                    "context": "${data.context}",
                    "response": "${outputs.response}",
                }
            },
            "relevance": {
                "column_mapping": {
                    "query": "${data.query}",
                    "response": "${outputs.response}",
                }
            },
            "similarity": {
                "column_mapping": {
                    "query": "${data.query}",
                    "ground_truth": "${data.ground_truth}",
                    "response": "${outputs.response}",
                }
            },
        },
        azure_ai_project=project_endpoint,
        output_path=str(OUTPUT),
    )
    ```

    > `azure_ai_project` はオプションです。 これを渡すと、実行がプロジェクトにアップロードされ、結果をポータルで他のものと一緒に確認できます。

1. **集計スコアを印刷する**:

    ```python
    # Print the aggregate scores
    print("\nAggregate scores (1-5, higher is better):")
    print(json.dumps(result["metrics"], indent=2))
    print(f"\nRow-level detail: {OUTPUT.resolve()}")
    if result.get("studio_url"):
        print(f"View in the Foundry portal: {result['studio_url']}")
    ```

1. ファイルを保存します (**Ctrl キーを押しながら S キーを押します**)。

### 実行してテストする

1. ターミナルでサインインし、評価を実行します。

    ```
    az login
    ```

    ```
    python evaluate_agent.py
    ```

1. エージェントに 10 問すべてを質問して、それぞれの応答を 3 とおりの方法で評価するので、数分お待ちください。 次のような結果が表示されます。

    ```
    Aggregate scores (1-5, higher is better):
    {
      "groundedness.groundedness": 4.6,
      "relevance.relevance": 4.4,
      "similarity.similarity": 4.1
    }

    Row-level detail: ...\eval_results.json
    ```

    > 数字は人によって異なります。 値が正確であることよりも、変更後にそれを*再現*できることの方が重要です。

1. **eval_results.json** を開いて、最もスコアの低い行を探します。 その `groundedness_reason` や `relevance_reason` を読む — 評価器が自ら説明しており、その説明があなたの行動の根拠となります。

1. スコアがエージェントの誤りであるかどうかを判断してください。 類似度スコアが低い場合、グラウンド トゥルースよりもエージェントの方が*良い*応答をしていたり、`context` の内容が質問に対して不十分だったりすることがあります。 評価はエージェントと同じくらいあなたのデータセットを評価します。

### 変化を起こし、それを証明する

これが評価の本当の目的です。

1. **data/caldova_eval.jsonl** の最後に悪いテスト ケースを追加する — `context` が `ground_truth` を裏付けていない質問では、エージェントが応答の根拠とするものがありません。

    ```json
    {"query": "What is the fast-track window for Halden Biologics?", "context": "Planning Desk Hours: Monday-Friday 8:00 AM - 6:00 PM.", "ground_truth": "Four months to first commercial batch."}
    ```

1. もう一度 `python evaluate_agent.py` を実行して、その列を見てください。 エージェントの応答が正しい場合でも、その応答が与えられた文脈によって裏付けられていないため、根拠性は急激に低下するはずです。 この違いが、まさに指標で見られるハルシネーションです。

1. 完了したら、その列をもう一度削除してください。

> ✅**チェックポイント**: エージェントの応答に再現可能なスコアが設定され、スコアごとに行レベルの理由ができ、明日のプロンプトの変更によって状況が良くなるか悪くなるかを判断する方法ができました。 これがこのラボの核心です。残りのタスクはオプションです。

完了したら、`deactivate` と入力して、仮想環境を終了します。

---

**次 (オプション):** [タスク 3 — エージェントをレッド チーミングする](D3-red-team-your-agent.md)
