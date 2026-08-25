---
title: タスク 3 – リモートエージェントを A2A で接続する
lab:
  title: タスク 3 – リモートエージェントを A2A で接続する
  description: エージェント間 (A2A) プロトコルを使用して、別々のプロセスで動作しているエージェントを接続します。ルーティング エージェントは転送タイトル エージェントと転送アウトライン エージェントを検出および委任し、両者は協力して Caldova の技術転送を計画します。
  type: task
  parent: C
  order: 3
  section: optional
  difficulty: 4
  duration: 30
  access: open
  level: 400
  concepts: 'A2A protocol, remote agents, multi-agent orchestration'
  status: draft
---

# タスク 3 — リモート エージェントを A2A で接続する

これは、**Agent Framework でマルチエージェント ソリューションを構築する**ラボの一部です。初めてご覧になる方は、「[はじめに](C0-getting-started.md)」から開始してください。**

> **設定 (ここから始めます):** このタスクには Foundry プロジェクトとスタート コードが必要です。 まだ用意していない場合は、「[はじめに](C0-getting-started.md)」を完了して、プロジェクトを作成し、コードをクローンし、`Python/.env` で `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` を設定します。
> 次に、VS Code で開いた `Python` フォルダーから準備ができていることを確認します。

```
python ../setup/check_env.py --task 3
```

> **前のタスクから続けている場合** 同じ `Python` フォルダーで前のタスクを終了したばかりで、プロジェクト、仮想環境、`.env` が既に設定されている場合、下記の「**検出可能なエージェントを作成する**」に直接進んでください。

---

これまでのところ、あなたのエージェントは単一のプロセスの中で動作していました。 実際のシステムはしばしばサービスに分かれており、各エージェントが独立して動作し、ネットワーク上で連携しています。 **エージェント間 (A2A) プロトコル**は、エージェントが自分たちの能力を公開し、互いに作業内容を送り合うための標準的な方法です。
このタスクでは、3 つのリモート エージェントから Caldova 転送計画システムを構築します。**転送タイトル エージェント**は見出しを提案し、**転送アウトライン エージェント**はプランの下書きを行い、**ルーティング エージェント**は両方を検出し、各要求を適切なエージェントに委任します。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>A2A プロトコルとは</summary>
<div class="concept-body" markdown="1">

**Agent-to-Agent (A2A) プロトコル** を使用すると、別のプロセス内のエージェントが互いを検出して呼び出します。 各エージェントは**エージェント カード**を発行します。これは名前、スキル、エンドポイントを記述した小さな文書で、他のエージェントが実行時に確認することができます。 1 つのエージェント (ここではルーティング エージェント) がカードを読み取り、誰が要求を処理するかを決め、HTTP 経由でメッセージを送信します。リモート エージェントが作業を行い、応答を返します。

[詳細情報 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

`Python` フォルダーを開き、「[はじめに](C0-getting-started.md)」から仮想環境をアクティブ化し (`.\labenv\Scripts\Activate.ps1`)、下記に進みます。

このタスクのスタート コードは、各エージェントに 1 つのフォルダーと、クライアントおよび起動ツールに整理されています。

```output
Python
├── outline_agent/       # remote agent: drafts a transfer plan outline (provided complete)
│   ├── agent.py
│   ├── agent_executor.py
│   └── server.py
├── routing_agent/       # orchestrator that discovers and delegates to the other agents
│   ├── agent.py
│   └── server.py
├── title_agent/         # remote agent: suggests a transfer brief title
│   ├── agent.py
│   ├── agent_executor.py
│   └── server.py
├── client.py            # sends your prompt to the routing agent
└── run_all.py           # launches all three agent servers
```

各エージェント フォルダーには、Foundry エージェント コードと、それをホストするサーバーが含まれています。 **ルーティング エージェント**は、**転送タイトル** エージェントと**転送アウトライン** エージェントを検出して通信します。 **クライアント**を使用すると、ルーティング エージェントにプロンプトを送信できます。 `run_all.py` は、すべてのサーバーを起動します。

> `outline_agent` (転送アウトライン エージェント) は参照用として**完成**の状態で提供されています。`title_agent` で同等のコードを構築し、その後 `routing_agent` を接続します。

### 検出可能なエージェントを作成する

このタスクでは、Caldova の技術転送に関する見出しを提案する転送タイトル エージェントを完成させます。 また、エージェントを検出可能にするために、A2A プロトコルで使用するエージェントのスキルとカードも定義します。

> **ヒント**: コードを追加する際は、コメントとインデントを揃えておきましょう。

1. **title_agent/agent.py** を開きます。

1. コメント "**Create the agents client**" (エージェント クライアントを作成する) を見つけて、Foundry プロジェクトに接続するコードを追加します。

    ```python
    # Create the agents client
    self.client = AgentsClient(
        endpoint=os.environ['PROJECT_ENDPOINT'],
        credential=DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True
        )
    )
    ```

1. コメント "**Create the title agent**" (タイトルエージェントを作成する) を見つけて、エージェントを作成するコードを追加します。

    ```python
    # Create the title agent
    self.agent = self.client.create_agent(
        model=os.environ['MODEL_DEPLOYMENT_NAME'],
        name='transfer-title-agent',
        instructions="""
        You are a helpful planning assistant for Caldova.
        Given a site or capability the planner names, suggest a single clear transfer brief title.
        """,
    )
    ```

1. コメント "**Create a thread for the chat session"** (チャット セッションのスレッドを作成する) を見つけて、下記を追加します。

    ```python
    # Create a thread for the chat session
    thread = self.client.threads.create()
    ```

1. コメント "**Send user message**" (ユーザーメッセージを送信) を見つけて、下記を追加します。

    ```python
    # Send user message
    self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. コメント "**Create and run the agent**" (エージェントを作成して追加する) を見つけて、下記を追加します。

    ```python
    # Create and run the agent
    run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    ファイルの残りの部分にあるコードにより、エージェントの応答が処理され、返されます。

1. ファイルを保存します (**CTRL + S**)。 次に、エージェントのスキルとカードを A2A プロトコルと共有します。

1. **title_agent/server.py** を開きます。

1. コメント "**Define agent skills**" (エージェント スキルを定義する) を見つけて、下記を追加します。

    ```python
    # Define agent skills
    skills = [
        AgentSkill(
            id='generate_trip_title',
            name='Generate Transfer Title',
            description='Generates a transfer brief title based on a site or capability',
            tags=['title'],
            examples=[
                'Can you give me a title for a packaging transfer at Ashford?',
            ],
        ),
    ]
    ```

1. コメント "**Create agent card**" (エージェント カードを作成する) を見つけて、エージェントを検出可能にするメタデータを追加します。

    ```python
    # Create agent card
    agent_card = AgentCard(
        name='Caldova Transfer Title Agent',
        description='An intelligent title generator agent powered by Foundry. '
        'I can help you generate clear titles for Caldova tech transfers.',
        url=f'http://{host}:{port}/',
        version='1.0.0',
        default_input_modes=['text'],
        default_output_modes=['text'],
        capabilities=AgentCapabilities(),
        skills=skills,
    )
    ```

1. コメント "**Create agent executor**" (エージェント エグゼキューターを作成する) を見つけて下記を追加します。

    ```python
    # Create agent executor
    agent_executor = create_foundry_agent_executor(agent_card)
    ```

1. コメント "**Create request handler**" (要求ハンドラーを作成する) を見つけて、下記を追加します。

    ```python
    # Create request handler
    request_handler = DefaultRequestHandler(
        agent_executor=agent_executor, task_store=InMemoryTaskStore()
    )
    ```

1. コメント "**Create A2A application**" (A2A アプリケーションを作成する) を見つけて下記を追加します。

    ```python
    # Create A2A application
    a2a_app = A2AStarletteApplication(
        agent_card=agent_card, http_handler=request_handler
    )
    ```

    これにより、転送タイトル エージェントの情報を共有し、エージェント エグゼキューターを使用して受信要求を処理する A2A サーバーが作成されます。

1. ファイルを保存します (**CTRL + S**)。

### エージェント間のメッセージを有効にする

このタスクでは、A2A プロトコルを使用してルーティング エージェントが他のエージェントにメッセージを送信し、転送タイトル エージェントでエージェント エグゼキューターを完成させてメッセージを受信できるようにします。

1. **routing_agent/agent.py** を開きます。

    ルーティング エージェントはシステムを調整します。ユーザー メッセージが到着すると、スレッドを開始し、`create_and_process` を使用して要求を処理するリモート エージェントを決定し、`send_message` 関数を使用して HTTP 経由でそのエージェントにメッセージをルーティングします。 `send_message` メソッドは非同期であり、実行が完了するまで待機する必要があります。

1. コメント "**Retrieve the remote agent's A2A client using the agent name**" (エージェント名を使ってリモート エージェントの A2A クライアントを取得する) を見つけて、下記を追加します。

    ```python
    # Retrieve the remote agent's A2A client using the agent name 
    client = self.remote_agent_connections[agent_name]
    ```

1. コメント "**Construct the payload to send to the remote agent**" (リモート エージェントに送信するペイロードを構築する) を見つけて、下記を追加します。

    ```python
    # Construct the payload to send to the remote agent
    payload: dict[str, Any] = {
        'message': {
            'role': 'user',
            'parts': [{'kind': 'text', 'text': task}],
            'messageId': message_id,
        },
    }
    ```

1. コメント "**Wrap the payload in a SendMessageRequest object**" (SendMessageRequest オブジェクトでペイロードをラップする) を見つけて、下記を追加します。

    ```python
    # Wrap the payload in a SendMessageRequest object
    message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. コメント "**Send the message to the remote agent client and await the response**" (リモート エージェント クライアントにメッセージを送信し応答を待つ) を見つけて、下記を追加します。

    ```python
    # Send the message to the remote agent client and await the response
    send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```

1. ファイルを保存します (**CTRL + S**)。 ルーティング エージェントはリモート エージェントを検出し、メッセージを送信できるようになりました。
    次に、転送タイトル エージェントのエグゼキューターを完成させ、その受信メッセージを処理できるようにします。

1. **title_agent/agent_executor.py** を開きます。

    `AgentExecutor` クラスは `execute` と `cancel` を実装する必要があります。 `cancel` メソッドが提供されています。 `execute` メソッドは `TaskUpdater` オブジェクトを使ってイベントを管理し、タスクが完了すると信号を送ります。下記の実行ロジックを追加します。

1. `execute` メソッドで、コメント "**Process the request**" (要求を処理する) を見つけて、下記を追加します。

    ```python
    # Process the request
    await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. `_process_request` メソッドで、コメント "**Get the title agent**" (タイトル エージェントを取得する) を見つけて、下記を追加します。

    ```python
    # Get the title agent
    agent = await self._get_or_create_agent()
    ```

1. コメント "**Update the task status**" (タスク ステータスを更新する) を見つけて、下記を追加します。

    ```python
    # Update the task status
    await task_updater.update_status(
        TaskState.working,
        message=new_agent_text_message('Title Agent is processing your request...', context_id=context_id),
    )
    ```

1. コメント "**Run the agent conversation**" (エージェントとの会話を実行する) を見つけて、下記を追加します。

    ```python
    # Run the agent conversation
    responses = await agent.run_conversation(user_message)
    ```

1. コメント "**Update the task with the responses**" (応答でタスクを更新する) を見つけて、下記を追加します。

    ```python
    # Update the task with the responses
    for response in responses:
        await task_updater.update_status(
            TaskState.working,
            message=new_agent_text_message(response, context_id=context_id),
        )
    ```

1. コメント "**Mark the task as complete**" (タスクを完了とマークする) を見つけて、下記を追加します。

    ```python
    # Mark the task as complete
    final_message = responses[-1] if responses else 'Task completed.'
    await task_updater.complete(
        message=new_agent_text_message(final_message, context_id=context_id)
    )
    ```

    転送タイトル エージェントが、A2A プロトコルでメッセージ処理に使用されるエグゼキューターによりラップされるようになりました。

1. ファイルを保存します (**CTRL + S**)。

### 実行してテストする

1. ターミナルでサインインし、3 つのエージェント サーバーすべてを起動します。

    ```
    az login
    ```

    ```
    python run_all.py
    ```

    サーバーが、認証された Azure セッションの使用を開始します。 各サーバーから準備が完了したことが報告されるまで待ちます。

1. **2 番目の** ターミナルで (仮想環境がアクティブ化された状態で)、クライアントを実行します。

    ```
    python client.py
    ```

1. メッセージが表示されたら、次のようなプロンプトを入力します。

    ```
    Create a title and outline for a packaging transfer at Ashford.
    ```

    しばらくすると、ルーティング エージェントが転送タイトルおよび転送アウトライン エージェントに委任され、提案されたタイトルとプランの概要が応答に表示されます。

    > **ヒント**: ポートが既に使用されているためにサーバーの起動に失敗した場合は、それより前に実行されていたものを停止 (`run_all.py` ターミナルで Ctrl + C) してから、もう一度やり直すか、`.env` で `*_PORT` 値を変更します。

1. 終了したら、`run_all.py` ターミナルで **Ctr l + C** を押してすべてのサーバーを停止し、各ターミナルで `deactivate` と入力して仮想環境を終了します。

> ✅ **チェックポイント**: A2A プロトコルを使用して別々のプロセスで実行されているエージェントを接続しました。具体的には、エージェント カードの発行、適切なリモート エージェントへの要求のルーティング、結果の返答を行いました。

---

**次へ (任意):** [タスク4 — サポート チケットを分類しルーティングする](C4-classify-and-route-a-ticket.md)
