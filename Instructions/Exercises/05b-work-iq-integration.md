---
lab:
  title: Work IQ - AI エージェント向けワークプレース インテリジェンス (オプション)
  description: Work IQ とモデル コンテキスト プロトコルを利用して、Microsoft 365 のワークプレース データにアクセスする AI エージェントを作成し、会議の準備、プロジェクトの進捗管理、実施項目の管理を行います。
  level: 300
  duration: 40
  islab: true
---

# Work IQ - AI エージェント向けワークプレース インテリジェンス

このラボでは、モデル コンテキスト プロトコル (MCP) を基盤とする Microsoft のコンテキスト インテリジェンス レイヤーである **Work IQ** を活用して、Microsoft 365 のワークプレース データにアクセスする AI エージェントを作成します。 実際の M365 データを使用して、会議の準備、プロジェクトの追跡、実施項目の抽出、職場の質問への回答を行うことができるワークプレース インテリジェンス エージェントを作成します。

このラボには約 **40** 分かかります。

> **注:**  これは、Microsoft 365 Copilot ライセンスを必要とする**オプションかつ高度なラボ**です。 エンタープライズ学習者、Microsoft 従業員、または M365 Copilot のアクセス権を持つユーザー向けに設計されています。 Copilot が付属していない Standard M365 アカウントでは動作しません。

## 前提条件

このラボを開始する前に、次のことを確認してください:

- AI エージェントとモデル コンテキスト プロトコル (MCP) の基本的な理解
- **Copilot ライセンスが付属する Microsoft 365**
- Work IQ の IT 管理者の承認 (組織アカウントのみ)
- [Node.js 18](https://nodejs.org/en/download/) 以降がインストールされている
- [Python 3.13](https://www.python.org/downloads/) 以降がインストールされている
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) がインストールされている (`az login`で認証されている)
- クエリを実行するためのアクティブな M365 データ (メール、会議、Teams チャット)

> **重要:** Work IQ は、Microsoft 365 Copilot が有効になっているアカウントで**のみ**動作します。 Copilot を使わずにこのラボを完了することはできません。

## Work IQ のインストール

1. ターミナルまたはコマンド ウィンドウを開きます。

2. npm 経由で Work IQ をグローバルにインストールします:

   ```bash
   npm install -g @microsoft/workiq
   ```

3. 使用許諾契約書に同意します:

   ```bash
   workiq accept-eula
   ```

4. Work IQ のインストールをテストします:

   ```bash
   workiq ask -q "What meetings do I have today?"
   ```

5. **テストが成功した場合** - M365 予定表の会議情報が表示されます。 次のタスクに進みます。

6. **"管理者の同意が必要です" と表示される場合:**

   - コマンドに同意のための URL が表示されます
   - 次のメッセージを含めて、この URL を IT 管理者に送信します:"Microsoft Learn AI エージェント ラボには Work IQ アクセスが必要です"
   - 管理者の承認を待ってから、テスト コマンドをもう一度試します

7. **"M365 Copilot ライセンスがありません" と表示される場合:**

   - 残念ながら、Copilot ライセンスなしではこのラボを完了できません
   - 手順をよく読んで、概念を理解することは可能です
   - このラボはオプションとみなし、Copilot へのアクセスを取得後に再開してください

## Visual Studio Code でアプリの開発準備をする

次に、Visual Studio Code を使用してアプリを開発してみましょう。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

1. Visual Studio Code を起動し、[ターミナル] ウィンドウを開きます。
   
2. コマンドを入力して、リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。

   ```bash
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
   ```

3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。

    > **注**: Visual Studio Code に、開こうとしているコードを信頼するかどうかを確認するポップアップ メッセージが表示された場合は、**[はい、作成者を信頼します]** を選択して続行します。

4. 必要に応じて、リポジトリ内の Python コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます (ダイアログが表示された場合)。

    > **注**: ビルドとデバッグに必要なアセットをインストールするように求めるダイアログが表示された場合は、**[今はしない]** を選択します。

5. **[エクスプローラー]** ペインで、**Labfiles/05b-work-iq-integration/Python** フォルダーを展開します。

    提供されたファイルには、アプリケーション コード、構成設定、エージェント クライアント スタート コードが含まれます。

6. ターミナルに次のコマンドを入力して Python 仮想環境を作成します。

   ```bash
   python -m venv venv
   ```

7. 次のコマンドを実行して、仮想環境をアクティブにします。

   **Windows:**

   ```bash
   venv\Scripts\activate
   ```

   **macOS/Linux**:

   ```bash
   source venv/bin/activate
   ```

8. 必要な Python パッケージをインストールします:

   ```bash
   pip install -r requirements.txt
   ```

9. `.env` ファイルを構成します:

   ラボ フォルダーで、`.env` ファイルを開き、それを Foundry プロジェクト エンドポイントで更新します:

   ```env
   PROJECT_ENDPOINT=https://your-project.services.ai.azure.com/api/projects/your-id
   MODEL_DEPLOYMENT_NAME=gpt-4.1
   ```

   > **ヒント:** エンドポイントを取得するには、VS Code で、**[AI Toolkit]** 拡張機能を開き、アクティブなプロジェクトを右クリックし、**[エンドポイントのコピー]** を選択します。 AI Toolkit は、Foundry Toolkit for VS Code 拡張機能に含まれています。

### セットアップの確認

次のことを行っておきます。

- Work IQ がインストールされ、アクセス可能である (`workiq --version` が動作する)
- 管理者の同意が承認されている (または Copilot が付属する個人用 M365 アカウント)
- `workiq_lab.py` - メインの対話型アプリケーション
- `requirements.txt` - Python の依存関係がインストールされている
- プロジェクト エンドポイントで構成された `.env` ファイル

## ワークプレース インテリジェンスのシナリオを確認する

この演習では、1 つの AI エージェントと Work IQ ツールを使用して、5 つのワークプレース インテリジェンス シナリオを示す対話型の統合アプリケーションを実行します。

### ラボ アプリケーションを起動する

1. 仮想環境がアクティブ化された状態のラボ ディレクトリにいることを確認します。

2. ラボ アプリケーションを実行します:

   ```bash
   python workiq_lab.py
   ```

3. アプリケーションによって次の操作が行われます:
   - Work IQ のセットアップを検証する
   - Microsoft Foundry プロジェクトに接続する
   - Work IQ MCP クライアントを初期化する
   - ワークプレース インテリジェンス エージェントを作成する
   - 5 つのシナリオで対話型メニューを表示する

### 会議の準備シナリオ

このシナリオは、関連するコンテキストを収集することで会議の準備を支援します。

1. メイン メニューから **[1 - 会議の準備]** を選択します。

2. メッセージが表示されたら、次のような会議のトピックや時間を入力します:
   - "午後 2 時の会議"
   - "第 4 四半期計画セッション"
   - "チームのスタンドアップ"

3. エージェントは次の内容を実行します。
   - 会議の詳細 (時間、出席者、議題) を見つける
   - トピックに関する最近のメールを検索する
   - このテーマに関する以前の会議を探す
   - 重要なポイントと決定事項の要約
   - ディスカッション ポイントの提案

4. 出力を確認し、次の点に注意を払います:
   - ソースの引用方法 (メール、会議、日付)
   - エージェントが複数のソースから情報を合成する方法
   - 手動による検索と比較して節約できた時間

**熟考:** これは、メールや予定表を手動で検索することとどう違うのか?

### プロジェクトの状態のシナリオ

このシナリオでは、ワークプレース ツール全体のプロジェクトの更新を追跡します。

1. メイン メニューから **[2 - プロジェクトの状態]** を選択します。

2. 作業中のプロジェクト名を入力します。次に例を示します:
   - "Web サイトの再設計"
   - "第 1 四半期の OKR"
   - "顧客オンボーディング"

3. エージェントは次の内容を実行します。
   - プロジェクトに関するメールと Teams メッセージを検索する
   - 関連する会議とその結果を見つける
   - 最近の決定事項と変更内容を特定する
   - メンションされた阻害要因または問題を一覧表示する
   - 次のステップと期限を要約する

4. 結果を分析します。
   - 状態の更新はどの程度包括的なものか?
   - エージェントはどのソースを使用したか?
   - これは従来の API を使用して構築できるか? 開発作業の違いは何か?

### 実施項目のシナリオ

このシナリオでは、さまざまなソースからオープン タスクを抽出します。

1. メイン メニューから **[3 - 実施項目]** を選択します。

2. 時間範囲を選択します (または、"今週" の場合は Enter キーを押します):
   - "今日"
   - "過去 3 日間"
   - "今月"

3. エージェントは次の内容を実行します。
   - 割り当てられた実施項目の会議メモを検索する
   - 自分に送信されたタスク関連のメールを探す
   - 自分がメンションされた Teams メッセージを確認する
   - 期限のある項目を特定する
   - 可能であれば緊急度で優先順位を付ける

4. 出力を確認します:
   - すべての実施項目がキャプチャされているか?
   - 優先順位付けはどの程度正確であるか?
   - 実施項目 (会議、メール、Teams) はどこで見つかったか?

### 統合インテリジェンスのシナリオ

このシナリオでは、Work IQ (ワークプレース データ) と Foundry IQ (ナレッジ ベース) を**両方**一緒に使用する方法を示します。

> **注:**  このシナリオでは、インデックス付きナレッジ ベースを使用して Foundry プロジェクトで構成された Azure AI Search が必要です。

1. メイン メニューから **[4 - 統合インテリジェンス]** を選択します。

2. ワークプレースのディスカッションと公式ドキュメントの両方に存在するトピックを入力します:
   - "リモート ワーク ポリシー"
   - "経費レポート"
   - "セキュリティ ガイドライン"

3. エージェントは次の内容を実行します。
   - ワークプレース データの検索 (Work IQ): メール、会議、Teams ディスカッション
   - ナレッジ ベースの検索 (Foundry IQ): 公式ドキュメント、ポリシー、手順
   - ワークプレースのディスカッションと公式ドキュメントの比較
   - ギャップまたは不整合を特定する
   - ラベル付けされたソースを含む包括的な概要を提供する

4. 2 つの観点を比較します:
   - 公式にドキュメント化されているものと、非公式に説明されているものは何か?
   - 矛盾はないか?
   - 最新のソースはどれか?

**主要な分析情報:**

- **Work IQ** は、人々が実際に何をしているか、何を言っているかを伝えます
- **Foundry IQ** は、公式にドキュメント化されている内容を示します
- **Together** は意思決定のための完全なコンテキストを提供します

### カスタム クエリ シナリオ

このシナリオでは、自分の質問を使用してワークプレースのデータを調査できます。

1. メイン メニューから **[5 - カスタム クエリ]** を選択します。

2. さまざまな種類のワークプレースの質問を試します:

   **メール検索:**

   ```
   Find emails about the budget from my manager
   ```

   **会議の要約:**

   ```
   What was decided in yesterday's standup?
   ```

   **チーム アクティビティ:**

   ```
   What did the engineering team discuss this week?
   ```

   **ドキュメントの検出:**

   ```
   Show me shared documents about security policies
   ```

3. 次の内容を試します:
   - 異なる時間範囲
   - さまざまなデータ ソース (メールと会議と Teams の比較)
   - さまざまなレベルの具体性
   - 結果を絞り込むフォローアップの質問

4. 何が適切に機能するかに注意してください:
   - 通常、具体的なクエリはあいまいなクエリよりも適切に機能します
   - 時間範囲を含めると関連性が向上します
   - 名前とキーワードは結果を絞り込むのに役立ちます

## 探索と実験

すべてのシナリオが完了しました。5 - 10 分間、自由に探索してみましょう。

### エッジ ケースのテスト

1. お持ちでないデータに関するクエリを試してみてください。エージェントはどのように応答しますか?

2. あいまいな質問をしてみてください。エージェントはどのように処理しますか?

3. 非常に古い情報を検索してみてください。どのような制限があるでしょうか?

### さまざまなクエリ スタイルを確認する

1. **非常に具体的**:"1 月 15 日に送信された第 3 四半期の予算に関する John からのメールを見つけてください"

2. **非常に広範**:"最近の開発について教えてください"

3. **比較**:"今週のディスカッションと先週のディスカッションを比較してください"

### Work IQ の機能を表示する

メイン メニューから **[6 - Work IQ 機能の表示]** を選択して確認します:

- ーキテクチャの概要
- 使用可能なデータ ソース
- セキュリティとプライバシー モデル
- Work IQ と Foundry IQ の比較
- 一般的なユース ケース

## コードについて理解する

このラボで使用される主なパターンを見てみましょう。

### パターン 1:Work IQ MCP クライアントの初期化

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

### パターン 2:Work IQ ツールを使用したエージェントの作成

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
    agent_name="workplace-intelligence-agent",
    definition=PromptAgentDefinition(
        model=self.model_deployment,
        instructions="You are a workplace intelligence assistant...",
        tools=workiq_tools  # Work IQ tools added here
    )
)

# Keep a map of raw tools for lookup during execution
self.raw_tools_map = {tool.name: tool for tool in raw_tools}
```

各 MCP ツールは、`FunctionTool` オブジェクトにラップされ、`PromptAgentDefinition` に渡されます。 未加工のツール マップを使用すると、エージェントが名前でツールを呼び出すときに効率的な検索が可能になります。

### パターン 3:Responses API を使用したクエリの実行

```python
# Create conversation
conversation = self.openai_client.conversations.create(
    items=[{"type": "message", "role": "user", "content": query}]
)

# Create response with agent
response = self.openai_client.responses.create(
    conversation=conversation.id,
    extra_body={"agent_reference": {"name": self.agent.name, "type": "agent_reference"}}
)
```

これには Responses API パターン (従来の Run/Thread パターンではない) が使用され、よりクリーンなエージェント実行を実現できます。

### パターン 4: ツール呼び出しループ

最初の応答の後、エージェントにより 1 回以上の Work IQ ツール呼び出しが要求されることがあります。 会話を続けるには、これらを実行してフィードバックする必要があります。

```python
from openai.types.responses.response_input_param import FunctionCallOutput

while True:
    if response.status == "failed":
        break

    input_list = []
    for item in response.output:
        if item.type == "function_call":
            kwargs = json.loads(item.arguments)

            # Call the Work IQ tool via MCP
            async def _execute():
                async with stdio_client(self.workiq_server_params) as (read, write):
                    async with ClientSession(read, write) as session:
                        await session.initialize()
                        return await session.call_tool(item.name, kwargs)

            result = asyncio.run(_execute())
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=result.content[0].text,
                )
            )

    if input_list:
        # Send tool results back and continue
        response = self.openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": self.agent.name, "type": "agent_reference"}}
        )
    else:
        break  # No more tool calls - final response ready
```

このループは、エージェントにより保留中の関数呼び出しがない応答を生成するまで続き、その時点で `response.output_text` に最終的な応答が格納されます。

## クリーンアップ

ラボでは、終了時にエージェントが自動的にクリーンアップされます:

```python
self.openai_client.agents.delete_version(
    agent_name=self.agent.name,
    version=self.agent.version
)
```

このラボには Azure リソースは作成されません (Work IQ では M365 ライセンスが使用されます)。そのため、追加のクリーンアップは必要ありません。

## トラブルシューティング

### "Work IQ コマンドが見つかりません"

**解決方法:** Work IQ をインストールします:

```bash
npm install -g @microsoft/workiq
```

### "管理者の同意が必要です"

**解決方法:**

1. `workiq mcp` を実行して同意 URL を取得します
2. IT 管理者に送信して承認を得ます
3. または、Copilot で個人用 M365 アカウントを使用します

### "M365 Copilot ライセンスがありません"

**解決方法:** このラボには Copilot が必要です。 次のいずれか:

- M365 Copilot ライセンスを購入します ($30/月)
- Copilot で組織アカウントを使用します
- ラボを読んで、実際の操作なしで概念を理解します

### "MCP サーバーが応答していません"

**解決方法:** Work IQ を直接テストします:

```bash
workiq ask -q "What meetings do I have?"
```

これが失敗した場合は、再インストールします:

```bash
npm install -g @microsoft/workiq
```

### "データが返されませんでした"

**解決方法:**

- M365 アカウントにメール、会議、Teams アクティビティがあることを確認します
- より広範なクエリを試します
- クエリが実際のデータと一致するかどうかを確認します