---
title: 'タスク 4 - Work IQ: Microsoft 365 のシグナルをエージェントに取り込む'
lab:
  title: 'タスク 4 - Work IQ: Microsoft 365 のシグナルをエージェントに取り込む'
  description: Work IQ とモデル コンテキスト プロトコルを利用して、Microsoft 365 のワークプレース データにアクセスするエージェントをビルドし、会議の準備、プロジェクトの進捗管理、実施項目の管理を行います。
  type: task
  parent: B
  order: 4
  section: optional
  difficulty: 4
  duration: 40
  access: gated
  requires: 'A Microsoft 365 Copilot licence, IT admin consent for Work IQ, and Node.js 18 or later'
  verify: 'Run the command below. If it returns your calendar you''re ready; if it reports missing consent or no Copilot licence, skip this task.'
  verify_command: npm install -g @microsoft/workiq && workiq accept-eula && workiq ask -q "What meetings do I have today?"
  level: 400
  concepts: 'Work IQ, Microsoft 365, Model Context Protocol (MCP), function tools'
  status: draft
---

# タスク 4 — Work IQ: Microsoft 365 のシグナルをエージェントに取り込む

これは、**エージェントをエンタープライズ ナレッジおよび Microsoft 365 と統合する**ラボの一部です。初めてご覧になる方は、「[はじめに](B0-getting-started.md)」から開始してください。**

<!-- BEGIN GENERATED: gated-notice - do not edit by hand; run: python tools/generate_lab_blocks.py -->
> ### 始める前にアクセスを確認する
>
> **このタスクに必要なもの:** Microsoft 365 Copilot ライセンス、Work IQ に対する IT 管理者の同意、Node.js 18 以降。
>
> 次のコマンドを実行します。 予定表が返される場合は、準備ができています。同意がない、または Copilot ライセンスがないと報告された場合は、このタスクをスキップしてください。

```
npm install -g @microsoft/workiq && workiq accept-eula && workiq ask -q "What meetings do I have today?"
```

> **お持ちでない場合** このタスクはスキップしてください。 このラボの他のものはそれに依存していません。手順を読んで、そのしくみを確認することはできます。
<!-- END GENERATED: gated-notice -->

> **設定 (ここから始めます):** このタスクには Foundry プロジェクト (展開済みモデル付き) とスタート コードが必要です。 まだ用意していない場合は、「[はじめに](B0-getting-started.md)」を完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定します。 次に、VS Code で開いた `Python` フォルダーから準備ができていることを確認します。

```
python ../setup/check_env.py --task 4
```

> **前のタスクから続けている場合** もしプロジェクトや仮想環境、`.env` を以前のタスクで既に設定した場合は、Work IQ (下記) をインストールするだけで、そのまま「**ワークプレース インテリジェンスのシナリオを確認する**」に進むことができます。

---

タスク 1-3 ではエージェントが "ドキュメント" を典拠としていたのに対し、このタスクはエージェントを**ライブの Microsoft 365 シグナル** (メール、会議、Teams メッセージ) に **Work IQ** を使って接続します。** 会議の準備、プロジェクトの追跡、実際の M365 データからのアクション項目の抽出を行うことができる Caldova Traders のワークプレース インテリジェンス エージェントを構築します。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>Work IQ とは</summary>
<div class="concept-body" markdown="1">

**Work IQ** は Microsoft 365 向けのコンテキスト インテリジェンス層であり、**モデル コンテキスト プロトコル (MCP)** サーバーとして公開されています。 これにより、ワークプレース データ (メール、予定表、Teams メッセージ、ドキュメント) へのアクセス許可に対応したアクセス権がエージェントに与えられます。これにより、エージェントはユーザーが実際に行っていることや話していることを推論できるようになります。 これは、**Foundry IQ** (キュレーションされた知識) を、**ライブのワークプレース シグナル**で補完します。

</div>
</details>

## Work IQ のインストール

1. ターミナルまたはコマンド ウィンドウを開きます。

2. npm 経由で Work IQ をグローバルにインストールします:

   ```
   npm install -g @microsoft/workiq
   ```

3. 使用許諾契約書に同意します:

   ```
   workiq accept-eula
   ```

4. Work IQ のインストールをテストします:

   ```
   workiq ask -q "What meetings do I have today?"
   ```

5. **テストが成功した場合** - M365 予定表の会議情報が表示されます。 次のセクションに進みます。

6. **"管理者の同意が必要です" と表示される場合:**

   - コマンドに同意のための URL が表示されます
   - 次のメッセージを含めて、この URL を IT 管理者に送信します:"Microsoft Learn AI エージェント ラボには Work IQ アクセスが必要です"
   - 管理者の承認を待ってから、テスト コマンドをもう一度試します

7. **"M365 Copilot ライセンスがありません" と表示される場合:**

   - 残念ながら、Copilot ライセンスなしではこのタスクを完了できません
   - 手順をよく読んで、概念を理解することは可能です

## アプリの準備

Work IQ アプリはスタート コードに**完成した形**で提供されており、そのまま実行します。

1. `Python` フォルダーを開き、「[はじめに](B0-getting-started.md)」の仮想環境をアクティブ化します。

    ```
    .\labenv\Scripts\Activate.ps1
    ```

1. `.env` に `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` が設定されていることを確認します (Work IQ では、モデル デプロイを使用してエージェントを実行します)。

1. **workiq_lab.py** を確認します。 次のことが行われます。
    - Work IQ のインストールを検証する
    - Microsoft Foundry プロジェクトに接続する
    - Work IQ MCP クライアントを初期化する (`npx -y @microsoft/workiq mcp`)
    - Work IQ ツールで `caldova-workplace-agent` を作成する
    - 5 つのシナリオで対話型メニューを表示する

## ワークプレース インテリジェンスのシナリオを確認する

1. Azure にサインインしてアプリを実行します。

    ```
    az login
    ```

    ```
    python workiq_lab.py
    ```

アプリケーションは Work IQ とお使いの Foundry プロジェクトに接続され、5 つのシナリオのメニューを表示します。

### 会議の準備シナリオ

1. メイン メニューから **[1 - 会議の準備]** を選択します。

2. メッセージが表示されたら、次のような会議のトピックや時間を入力します:
   - "午後 2 時の会議"
   - "春のカタログ企画セッション"
   - "現場業務ミーティング"

3. エージェントはあなたの会議の詳細を見つけ、そのトピックに関する最近のメールを検索し、過去の会議を探し、重要なポイントをまとめ、議論のポイントを提案します。

4. 出力を見直し、情報源の引用方法 (メール、会議、日付) やエージェントが複数の情報源から情報を統合する方法を確認しましょう。

### プロジェクトの状態のシナリオ

1. メイン メニューから **[2 - プロジェクトの状態]** を選択します。

2. 作業中のプロジェクト名を入力します。次に例を示します:
   - "春のカタログ発売"
   - "キャパシティ レビュー"
   - "サプライヤーのオンボーディング"

3. 担当者はメールや Teams メッセージを検索し、関連する会議を見つけ、最近の決定事項や障害を特定し、次のステップや締め切りをまとめます。

### 実施項目のシナリオ

1. メイン メニューから **[3 - 実施項目]** を選択します。

2. 時間の範囲 (または "今週" で Enter キーを押します)、"今日"、"直近 3 日間"、"今月" を選びます。

3. エージェントは会議ノート、タスク関連のメール、Teams のメンションを検索し、締め切りのある項目を特定し、緊急性で優先順位をつけます。

### 統合インテリジェンスのシナリオ

このシナリオでは、Work IQ (ワークプレース データ) と Foundry IQ (ナレッジ ベース) を**両方**一緒に使用する方法を示します。

> **注**: このシナリオでは、インデックス付きナレッジ ベースを含む Foundry IQ (Azure AI 検索) がプロジェクトに構成されている必要があります。たとえば、[タスク 1](B1-create-a-foundry-iq-knowledge-agent.md) の Caldova ナレッジ ベースです。

1. メイン メニューから **[4 - 統合インテリジェンス]** を選択します。

2. ワークプレースのディスカッションと公式ドキュメントの両方に存在するトピックを入力します:
   - "キャパシティ要求および転送ポリシー"
   - "サプライヤーのリード タイム"
   - "契約製造の転送"

3. エージェントは、ワークプレース データ (Work IQ)、**および**ナレッジ ベース (Foundry IQ) を検索し、日常的なミーティングと正式なドキュメントを比較し、ギャップを特定し、ラベル付けされたソースを含む包括的な概要を提供します。

**主要な分析情報:**

- **Work IQ** は、人々が実際に何をしているか、何を言っているかを伝えます
- **Foundry IQ** は、公式にドキュメント化されている内容を示します
- **Together** は意思決定のための完全なコンテキストを提供します

### カスタム クエリ シナリオ

1. メイン メニューから **[5 - カスタム クエリ]** を選択します。

2. さまざまな種類のワークプレースの質問を試します:

    ```
    Find emails about the spring catalog from my manager
    ```

    ```
    What was decided in yesterday's site operations standup?
    ```

    ```
    Show me shared documents about supplier lead times
    ```

3. 異なる時間の範囲、データ ソース、フォローアップの質問を試して結果を絞り込んでみましょう。

### Work IQ の機能を表示する

メイン メニューから **[6 - Work IQ の機能を表示する]** を選択して、アーキテクチャ、データ ソース、セキュリティ モデル、Work IQ と Foundry IQ の比較を確認できます。 **[0]** を選択して終了します。アプリで、退出時に `caldova-workplace-agent` バージョンが削除されます。

## コードについて

`workiq_lab.py` で使用される主なパターンを確認しましょう。

### パターン 1: Work IQ MCP クライアントの初期化

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

# Store server parameters for reuse
self.workiq_server_params = StdioServerParameters(
    command="npx",
    args=["-y", "@microsoft/workiq", "mcp"]
)

# Fetch available tools from Work IQ MCP server
async def _fetch():
    async with stdio_client(self.workiq_server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            tools_result = await session.list_tools()
            return tools_result.tools

raw_tools = asyncio.run(_fetch())
```

永続的な接続を維持するのではなく、操作ごとに新しい MCP セッションが開かれます。 `StdioServerParameters` には、Work IQ MCP サーバー サブプロセスを毎回起動するために使用されるコマンドと引数が格納されます。

### パターン 2: Work IQ ツールを使用したエージェントの作成

```python
from azure.ai.projects.models import PromptAgentDefinition, FunctionTool

# Convert MCP tools to FunctionTool objects
workiq_tools = [
    FunctionTool(
        name=tool.name,
        description=tool.description,
        parameters=tool.inputSchema,
    )
    for tool in raw_tools
]

# Create agent with Work IQ tools
self.agent = self.project_client.agents.create_version(
    agent_name="caldova-workplace-agent",
    definition=PromptAgentDefinition(
        model=self.model_deployment,
        instructions="You are a workplace intelligence assistant for Caldova staff...",
        tools=workiq_tools  # Work IQ tools added here
    )
)
```

各 MCP ツールは、`FunctionTool` にラップされ、`PromptAgentDefinition` に渡されます。

### パターン 3: ツール呼び出しループ

最初の応答の後、エージェントにより 1 回以上の Work IQ ツール呼び出しが要求されることがあります。 会話を続けるには、これらを実行してフィードバックします。

```python
from openai.types.responses.response_input_param import FunctionCallOutput

while True:
    if response.status == "failed":
        break

    input_list = []
    for item in response.output:
        if item.type == "function_call":
            kwargs = json.loads(item.arguments)
            result = self._call_workiq_tool(item.name, kwargs)
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=result.content[0].text,
                )
            )

    if input_list:
        response = self.openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": self.agent.name, "type": "agent_reference"}}
        )
    else:
        break  # No more tool calls - final response ready
```

このループは、エージェントにより保留中の関数呼び出しがない応答を生成するまで続き、その時点で `response.output_text` に最終的な応答が格納されます。

> ✅ **チェックポイント**: Work IQ を通じて **ライブ Microsoft 365 シグナル**を推論に取り込むエージェントを構築し、タスク 1 のドキュメントを典拠として作成されたエージェントを補完する方法を確認しました。

## クリーンアップ

アプリによって、退出時に `caldova-workplace-agent` バージョンが削除されます。 Work IQ では Azure リソースを作成する代わりに M365 ライセンスが使用されるので、このタスクで削除されるものは他にありません。 完了したら、ターミナルに「`deactivate`」と入力して、仮想環境を終了します。

## トラブルシューティング

**"Work IQ コマンドが見つかりません"** — Work IQ のインストール: `npm install -g @microsoft/workiq`

**"管理者の同意が必要です"** — `workiq mcp` を実行して同意書の URL を取得し、IT 管理者に送信するか、Copilot 対応の個人の M365 アカウントを使用します。

**"M365 Copilot ライセンスがありません"** — このタスクには Copilot が必要です。 M365 Copilot ライセンスがあるアカウントを使うか、ラボの文書を読んで概念を理解しましょう。

**"MCP サーバーが応答しません"** — `workiq ask -q "What meetings do I have?"` で Work IQ を直接テストしてください。 もし失敗したら、`npm install -g @microsoft/workiq` で再インストールしてください。

**"データが返されません"** — お使いの M365 アカウントにメール、会議、Teams のアクティビティがあることを確認し、より広範なクエリも試してみてください。

---

**[ラボの概要](B-integrate-agents-with-enterprise-knowledge-and-m365.md)に戻ります。**
