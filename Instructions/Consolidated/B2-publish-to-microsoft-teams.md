---
title: タスク 2 - エージェントを Microsoft Teams に公開する
lab:
  title: タスク 2 - エージェントを Microsoft Teams に公開する
  description: Caldova のナレッジ エージェントを Microsoft Teams に公開し、スタッフが既に作業している場所でチャットできるようにします。
  type: task
  parent: B
  order: 2
  section: optional
  difficulty: 2
  duration: 20
  access: gated
  requires: 'A Microsoft 365 account with Teams access, and permission to publish agents to Teams in your tenant'
  verify: 'In the Foundry portal, open your agent and select **Publish**. If Teams is greyed out or returns a consent error, you don''t have the rights this task needs.'
  level: 200
  concepts: 'agent publishing, Microsoft Teams, Azure Bot Service'
  status: draft
---

# タスク 2 — エージェントを Microsoft Teams に公開する

これは、**エージェントをエンタープライズ ナレッジおよび Microsoft 365 と統合する**ラボの一部です。初めてご覧になる方は、「[はじめに](B0-getting-started.md)」から開始してください。**

<!-- BEGIN GENERATED: gated-notice - do not edit by hand; run: python tools/generate_lab_blocks.py -->
> ### 始める前にアクセスを確認する
>
> **このタスクに必要なもの:** Teams のアクセス権のある Microsoft 365 アカウント、テナント内の Teams にエージェントを公開する権限。
>
> Foundry ポータルで、エージェントを開き **[発行]** を選択します。 Teams が淡色表示されている、または同意についてのエラーが返される場合、このタスクに必要な権利がありません。
>
> **お持ちでない場合** このタスクはスキップしてください。 このラボの他のものはそれに依存していません。手順を読んで、そのしくみを確認することはできます。
<!-- END GENERATED: gated-notice -->

> **設定 (ここから始めます):** このタスクは[タスク 1](B1-create-a-foundry-iq-knowledge-agent.md) の典拠された `caldova-knowledge-agent` を公開します。 まだそのエージェントがない場合は、まずタスク 1 を完了します (または、最も早い方法として VS Code で開いた `Python` フォルダーの `python ../setup/bootstrap_agent.py` でコードで作成し典拠します)。 このタスクはポータルと Teams で完全に完了することができます。ローカル コードや `.env` ファイルは必要ありません。

> **前のタスクから続けている場合** もしタスク 1 を終えたばかりで、`caldova-knowledge-agent` が Foundry ポータルを典拠として保存されている場合は、準備はできています。下記の「**Microsoft Teams に公開する**」に直接進みます。

---

**Microsoft Teams** に公開すると、Caldova のスタッフは、既に使用しているツールを離れることなく、Teams でナレッジ アシスタントと直接チャットできます。 このタスクは、**デプロイと公開のワークフロー**に焦点を当てているため、コードは記述しません。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>Teams に公開した場合に起きること</summary>
<div class="concept-body" markdown="1">

Teams にエージェントを発行すると、Foundry ポータルにより自動的に **Azure Bot Service が作成され**、**Teams アプリ マニフェスト**が生成され、**アプリ アイコンと構成**がパッケージされ、**ダウンロード可能なアプリ パッケージ**が提供されます。 短いフォームに入力すればポータルでまとめられるため、手動で作成することはありません。

</div>
</details>

## Microsoft Teams に公開する

Foundry ポータルで Teams に対して発行すると、自動的に次のことが行われます。

- Azure Bot Service を作成する
- Teams アプリ マニフェストを生成する
- アプリのアイコンと構成をパッケージ化する
- ダウンロード可能なアプリ パッケージを作成する

### アプリ情報を準備する

発行の前に、次の情報を集めます。

| フィールド | 値 |
|-------|-------|
| **アプリ名** | Caldova Knowledge Assistant |
| **簡単な説明** | Caldova スタッフ向けの AI アシスタント |
| **詳細な説明** | 工場のキャパシティ、現場運営、契約メーカー、サプライヤーに関するスタッフの質問に回答する Enterprise AI アシスタント |
| **開発者名** | 自分の名前または会社名 |
| **Web サイトの URL** | <https://caldova.example> (ラボではプレースホルダーのままでかまいません) |
| **プライバシー ポリシーの URL** | <https://caldova.example/privacy> |
| **使用条件の URL** | <https://caldova.example/terms> |

### アプリのアイコンを作成する

Teams アプリには次の 2 つのアイコンが必要です。

1. **カラー アイコン** (192x192 ピクセル)
   - アプリのロゴのフル カラー バージョン
   - PNG 形式

2. **アウトライン アイコン** (32x32 ピクセル)
   - 透明な背景に白のアウトライン
   - PNG 形式
   - Teams サイド バーで使用される

> **このラボ用の簡単な方法**: PowerPoint、ペイント、またはオンライン ツール (例: Canva) を使用して、単純な色付きの正方形を作成してテキストまたはイニシャルを追加します。

### ポータルから発行する

1. Foundry ポータルで、エージェントを開きます (**[ビルド]** → **[エージェント]** → **caldova-knowledge-agent**)

2. ページの上部にある **[発行]** ボタンを選択します

3. **[Teams と Microsoft 365 Copilot に対して発行する]** を選択します。

4. **[続行]** を選択します

### Teams アプリの詳細を構成する

次のとおりに構成フォームに入力します。

**基本情報:**

- **アプリ名**:Caldova Knowledge Assistant
- **簡単な説明**: Caldova スタッフ向けの AI アシスタント
- **詳細な説明**: 工場のキャパシティ、現場運営、契約メーカー、サプライヤーに関するスタッフの質問に答えるエンタープライズ AI アシスタント

**開発者情報:**

- **開発者名**: 名前
- **Web サイト**: <https://caldova.example>
- **プライバシー ポリシー**: <https://caldova.example/privacy>
- **使用条件**: <https://caldova.example/terms>

**アプリ アイコン:**

- **カラー アイコン** (192x192 ピクセル) をアップロードする
- **アウトライン アイコン** (32x32 ピクセル) をアップロードする

**アプリ スコープ:**

- 個人チャットでアクセスできるようにするには **[個人用]** を選択します
- (省略可能) チャネルでアクセスできるようにするには **[チーム]** を選択します

**[エージェントを準備する]** を選択します

### Teams へのデプロイ

エージェント パッケージの準備 (これには 1 分から 2 分ほどかかります) が完了すると、Teams にデプロイすることができます。

1. パッケージの準備ができたら、**[製品内発行フローを続行する]** を選択します

2. 発行スコープを選択します。
   - **[個人の範囲]**: エージェントは Teams エージェント ストアの "エージェント" の下に表示されます。 管理者の承認は必要ありません。 個人でテストする場合に最適です。
   - **[組織の (テナント) 範囲]**: エージェントはすべてのユーザーを対象として "組織による構築" の下に表示されます。 管理者の承認が必要です。

3. このラボでは、**[個人の範囲]** を選択します

4. **[送信]** を選択します

5. 発行が完了するまで待ちます (成功メッセージが表示されます)

> **直接発行が失敗した場合の代替方法**: 発行ダイアログで **400** エラーが返され、Microsoft 365 アカウントにカスタム アプリを発行するアクセス許可がある場合は、代わりに [**ダウンロードとカスタマイズ**] タブを開き、指示に従います。

6. これで、作成したエージェントを Teams で利用できるようになりました。 これは **[アプリ]** → **[エージェント]** の下にあります

### エージェントを Teams でテストする

1. エージェントのチャットがインストール後に開くはずです (開いていない場合は **[アプリ]** → **[エージェント]** で見つけてください)

2. あいさつを送ります。

    ```
    Hello! What can you help me with?
    ```

3. ナレッジ クエリをテストします。

    ```
    How much headroom does Calderwood have?
    ```

4. 別の質問を試します。

    ```
    What are our site core hours?
    ```

5. エージェントは Caldova ナレッジ ベースからの情報を使って応答します。

> ✅ **チェックポイント**: 典拠されているナレッジ エージェントが Microsoft Teams で利用可能になり、エンタープライズ ナレッジ ベースからのスタッフの質問に回答できるようになりました。

### Teams 展開のトラブルシューティング

**エージェントを Teams で見つけられない (直接発行後):**

- Teams の **[アプリ]** → **[エージェント]** セクションを確認します
- 発行後にエージェントが表示されるまで 1 分から 2 分ほど待ちます
- 発行が正常に完了したことを Foundry ポータルで確認します

**アプリをアップロードできない (手動アップロード):**

- manifest.zip ファイルが破損していないことを確認します (必要に応じて再ダウンロードします)
- カスタム アプリのアップロードが Teams 管理者によって無効化されていないことを確認します
- アイコンのサイズが正しいことを確認します (192x192 と 32x32)

**エージェントが応答しない:**

- インストール後にボットが初期化されるまで 30 秒待ちます
- Azure Bot Service が作成されたことを確認します (発行中に表示されます)
- エージェントを最初に Foundry プレイグラウンドでテストします

**応答が一般論的である (知識なし):**

- エージェントで Foundry IQ (またはファイル検索) が有効になっていることを確認する
- ドキュメントがアップロード済みでインデックス作成済みであることを確認します
- Foundry プレイグラウンドでナレッジ クエリをテストします

---

**次へ (任意):** [タスク 3 — Microsoft 365 Copilot に公開する](B3-publish-to-microsoft-365-copilot.md) · [タスク 4 — Work IQ](B4-work-iq-workplace-intelligence.md)
