---
title: タスク 6 - アシスタントをホステッド エージェントに昇格する
lab:
  title: タスク 6 - アシスタントをホステッド エージェントに昇格する
  description: プロンプト エージェントとして構築した Caldova アシスタントを、Azure Developer CLI を使用して、ホステッド エージェント (Foundry マネージド コンテナーで実行される独自のコード) としてデプロイします。
  type: task
  parent: A
  order: 6
  section: optional
  difficulty: 3
  duration: 30
  access: open
  level: 300
  concepts: 'hosted agents, deployment, Azure Developer CLI'
  status: draft
---

# タスク 6 — アシスタントをホステッド エージェントに昇格する

これは、**AI エージェントを構築および拡張する**ラボの一部です。初めてご覧になる方は、「[はじめに](A0-getting-started.md)」から開始してください。**

> **設定 (ここから始めます):** このタスクはコードを展開するため、Foundry プロジェクト、展開済みモデル、**Azure Developer CLI (`azd`)** が必要です。 「[はじめに](A0-getting-started.md)」を完了していない場合は完了し、プロジェクトを作成して、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定します。 次に、VS Code で開いた `Python` フォルダーから下記を確認します。

```
python ../setup/check_env.py --task 6
```

> また、**`azd` 1.25.3 以降**と Foundry 拡張機能も必要です。 下記を使用してインストールします。
>
> ```
> azd ext install microsoft.foundry
> ```

> **前のタスクから続けている場合** ホステッド エージェントは独自のフォルダー (`Python/hosted_agent/`) にあり、依存関係があるため、共有 `labenv` を再利用しません。 必要な情報はすべて下記にあります — 前のタスクを終えていなくても、ここから始めることができます。

---

**目標**: ビルドしたのと**同じ Caldova アシスタント**を、**ホストテッド エージェント**として実行します。独自のコードをパッケージ化して Foundry Agent Service に展開し、先ほどビルドしたプロンプト エージェントと同様に、参照によって呼び出せるようにします。

**プロンプトエージェントとホステッド エージェント** — これまでのすべてのタスクで "プロンプト エージェント" をビルドしました。エージェントを `PromptAgentDefinition` (モデル、指示、ツール) で記述し、Foundry に実行させました。** これは高速で宣言型ですが、ロジックはプロンプトとツールの定義に適合する必要があります。 "ホステッド エージェント" は、**独自のコード**つまり、フレームワークや単純な Python をコンテナーとしてパッケージし、Foundry が管理するインフラストラクチャ上で動作するものです。** ランタイムの動作を選択すると、プラットフォームではスケーリング、セッション状態、ID、エンドポイントが処理されます。 これは、アシスタントがプロンプト定義を超えたときの自然な次のステップであり、その後、Teams または Microsoft 365 向けにアシスタントを運用化する場合の手順です。

このタスクでは、**Responses プロトコル**を使用します。そのため、ホステッド エージェントは、OpenAI 互換性を維持します。つまり、プロンプト エージェントを呼び出したのと同じクライアント コードで、これを呼び出すことができます。

## エージェント コードを確認する

1. `Labfiles/A-build-and-extend-ai-agents/Python/hosted_agent` フォルダーで **main.py** を開きます。
    `azure-ai-agentserver-responses` ホスティング ライブラリは Web サーバー、正常性チェック、会話履歴を管理します。1 ターンを答える**ハンドラー**を記述するだけで済みます。

1. 2 つのブロックが `TODO` としてマークされています。Responses クライアントの作成と、ハンドラーからのモデルの呼び出しです。 これらを記述します。

> **まず試してみてください**: ハンドラーは既にユーザーのメッセージ (`user_input`) と会話履歴を `input_items` にまとめています。 それらをモデル デプロイに送って返信を受け取るようにする方法はありますか? (ヒント: Responses クライアントの `create(...)` は同期的であるため、サンプルでは、サーバーをブロックしないように `run_in_executor` を使用してイベント ループの外から実行されます。)**

<details markdown="1">
<summary>ソリューションを表示する</summary>

ファイルの上部近くに Responses クライアントを作成します。

```python
_responses_client = (
    AIProjectClient(endpoint=_endpoint, credential=DefaultAzureCredential())
    .get_openai_client()
    .responses
)
```

次に、Caldova システム プロンプトでモデルを呼び出すハンドラーを完成させ、返信を返します:

```python
response = await asyncio.get_running_loop().run_in_executor(
    None,
    lambda: _responses_client.create(
        model=_model,
        instructions=_SYSTEM_PROMPT,
        input=input_items,
        store=False,
    ),
)
return TextResponse(context, request, text=response.output_text)
```

完全なファイルは `Solution/Python/hosted_agent/main.py` にあります。

</details>

## モデルを構成する

1. `hosted_agent/.env.example` を `hosted_agent/.env` にコピーし、`AZURE_AI_MODEL_DEPLOYMENT_NAME` を展開したモデル名 (例:`gpt-4o`) に設定します。 ホストされたコンテナーには `FOUNDRY_PROJECT_ENDPOINT` が挿入されます。ローカルでテストすると `azd ai agent run` が自動的にこれを設定します。

## azd プロジェクトを初期化する

1. `hosted_agent` フォルダーからエージェントの定義をスキャフォールドします。 これにより、ホステッド **`azure.ai.agent`** サービスを記述する `azure.yaml` が生成されます。

    ```
    azd ai agent init --protocol responses --deploy-mode code
    ```

    プロンプトに応答する: **エージェント名** (例: `caldova-hosted-agent`) を選び、**[既存の Foundry プロジェクトを使用する]** (「はじめに」にあるもの) を選択し、サブスクリプションと所在地を選択します。

> 完成した `azure.yaml` は `Solution/Python/hosted_agent/` に含まれているので、ツールで作成されるものを確認できます。 `--deploy-mode code` は、Foundry によりコンテナーがビルドされることを意味し (**リモート ビルド**)、Docker をローカルにインストールする必要はありません。

## ローカルでプロビジョニングとテストを行う

1. サポート リソース (Application Insights など) をプロビジョニングします。

    ```
    azd provision
    ```

1. エージェントをローカルで実行します。 これにより仮想環境が作成され、`requirements.txt` がインストールされ、ハンドラーが起動し、ブラウザーでエージェント インスペクターが開かれます。

    ```
    azd ai agent run
    ```

1. インスペクターでエージェントとチャットするか、2 つ目の端末から呼び出してみてください。

    ```
    azd ai agent invoke --local "How long does review take for a capacity request?"
    ```

## Foundry Agent Service への展開

1. コンテナーをビルドして Foundry にデプロイします。

    ```
    azd deploy
    ```

    終了すると、出力に**エージェント プレイグラウンド**のリンクと**エージェント エンドポイント**が含まれています。 ホステッド エージェントが専用のエンドポイントとアイデンティティを保持するようになりました。

1. 展開されたエージェントを呼び出します。

    ```
    azd ai agent invoke "A planner needs five weeks of premium contract capacity. What will it cost at expedited priority?"
    ```

> **同じ参照で、ご自分のコードを使用しています**: ホステッド エージェントは、ビルドしたプロンプト エージェントとまったく同じように、つまり、OpenAI 互換クライアントを経由し名前を指定して呼び出されます。 プロンプト エージェントに対して `agent_reference` を使用したアプリは、代わりにこのホステッド エージェントをポイントすることができます。違いは、各ターンの応答するロジックが、プロンプト定義ではなく、コンテナーで実行される**ご自分のコード**になったことです。

> ✅ **チェックポイント**: Caldova アシスタントをプロンプト エージェントからホステッド エージェントに昇格し、`azd ai agent run` を使用してローカルでテストし、`azd deploy` を使用して展開し、展開されたエージェントを名前で呼び出しました。

## クリーンアップ

作業が終わったら、このタスクで作成したすべてのものを削除します。

```
azd down
```

> **警告**: `azd down` は Foundry プロジェクトやホステッド エージェントを含めて、リソース グループ内のすべてのリソースを削除します。 グループに他のリソースが含まれている場合は、それらも削除されます。

---

[ラボの概要](A-build-and-extend-ai-agents.md)に**戻ります**。
