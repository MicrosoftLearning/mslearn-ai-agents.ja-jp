---
title: タスク 3 - エージェントを Microsoft 365 Copilot に公開する
lab:
  title: タスク 3 - エージェントを Microsoft 365 Copilot に公開する
  description: Caldova のナレッジ エージェントを Microsoft 365 Copilot に公開し、スタッフが Copilot 内でアクセスできるようにします。
  type: task
  parent: B
  order: 3
  section: optional
  difficulty: 2
  duration: 15
  access: gated
  requires: 'A Microsoft 365 Copilot licence, and permission to publish agents to Copilot in your tenant'
  verify: 'In the Foundry portal, open your agent and select **Publish**. If Microsoft 365 Copilot is greyed out or returns a consent error, you don''t have the rights this task needs.'
  level: 200
  concepts: 'agent publishing, Microsoft 365 Copilot, Copilot agents'
  status: draft
---

# タスク 3 — エージェントを Microsoft 365 Copilot に公開する

これは、**エージェントをエンタープライズ ナレッジおよび Microsoft 365 と統合する**ラボの一部です。初めてご覧になる方は、「[はじめに](B0-getting-started.md)」から開始してください。**

<!-- BEGIN GENERATED: gated-notice - do not edit by hand; run: python tools/generate_lab_blocks.py -->
> ### 始める前にアクセスを確認する
>
> **このタスクに必要なもの:** Microsoft 365 Copilot ライセンス、テナント内で Copilot にエージェントを公開するアクセス許可。
>
> Foundry ポータルで、エージェントを開き **[発行]** を選択します。 Microsoft 365 Copilot が淡色表示されている、または同意についてのエラーが返される場合、このタスクに必要な権利がありません。
>
> **お持ちでない場合** このタスクはスキップしてください。 このラボの他のものはそれに依存していません。手順を読んで、そのしくみを確認することはできます。
<!-- END GENERATED: gated-notice -->

> **設定 (ここから始めます):** このタスクは[タスク 1](B1-create-a-foundry-iq-knowledge-agent.md) の典拠された `caldova-knowledge-agent` を公開します。 まだそのエージェントがない場合は、まずタスク 1 を完了します (または VS Code で開いた `Python` フォルダーの `python ../setup/bootstrap_agent.py` でコードで作成し典拠します)。 このタスクはポータルと Copilot で完全に完了することができます。ローカル コードや `.env` ファイルは必要ありません。

> **前のタスクから続けている場合** [タスク 2](B2-publish-to-microsoft-teams.md) で既に Teams に発行している場合、同じ発行フローで、エージェントを Copilot で使用可能にすることができます。下記の「**Microsoft 365 Copilot に公開する**」に直接進んでください。

---

**Microsoft 365 Copilot** に公開すると、エージェントが、スタッフが Copilot 内で直接アクセスできる **Copilot エージェント**になります。 このタスクは、**公開のワークフロー**に焦点を当てているため、コードは記述しません。

<style> /* "Ask Anton" just-in-time concept blocks */ details.concept { margin:.6rem 0 1rem; } details.concept > summary { display:inline-block; cursor:pointer; list-style:none; font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112; border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; } details.concept > summary::-webkit-details-marker { display:none; } details.concept > summary::before { content:"Ask Anton: "; font-weight:700; padding-left:1.5em; background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; } details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; } details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; } details.concept .concept-body { border:1px solid #6b4ba133; border-top:none; border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; } </style>

<details markdown="1" class="concept">
<summary>Microsoft 365 Copilot エージェントとは</summary>
<div class="concept-body" markdown="1">

Copilot に対して発行されたエージェントは、**Copilot エージェント** (別名、拡張機能または宣言型エージェント) になります。 スタッフは、これを Copilot の **@mentions** で呼び出し、Copilot 独自の機能に加えて、その知識にアクセスし、Copilot とエージェントをシームレスに切り替えることができます。

</div>
</details>

## Microsoft 365 Copilot に対して発行する

Copilot に公開すると、ユーザーは次のことを行うことができます。

- そのエージェントを Copilot で @mentions を使用して呼び出す
- Copilot の能力に加えてそのエージェントの知識にアクセスする
- Copilot とエージェントをシームレスに切り替える

### ポータルから発行する

1. Foundry ポータルに戻ります (**<https://ai.azure.com>**)

2. エージェントに移動します (**[ビルド]** → **[エージェント]** → **caldova-knowledge-agent**)

3. **[発行]** ボタンを選択します

4. **[Teams と Microsoft 365 Copilot に対して発行する]** を選択します

5. **[続行]** を選択します

> **注**:この発行フローは、Teams の場合に使用されるものと同じです。 1 つの発行プロセスを通じてエージェントが Teams と Copilot の両方で使用可能になります。

### 発行の詳細を構成する

このエージェントをまだ発行していない場合は、次のとおりに構成の情報を入力します (Teams のセクションと同じです)。

- **名前**: Caldova Knowledge Assistant
- **説明**: Caldova スタッフ向けの AI アシスタント
- **アイコン**: 192x192 と 32x32 のアイコンをアップロードする
- **発行元の情報**: 自分の名前とプレースホルダー URL

### 発行スコープを選択する

配布のスコープを選択します。

| 範囲 | 視程 | 管理者の承認 | 最適な用途 |
|-------|-----------|----------------|----------|
| **共有** | エージェント ストアの "あなたのエージェント" の下 | 必須ではない | 個人でのテスト、小規模チーム |
| **組織** | "組織による構築" の下 (すべてのユーザーが対象) | 必須 | 組織全体への配布 |

このラボでは、管理者の承認なしですぐにアクセスできるように **[共有スコープ]** を選択します。

### 発行を完了する

1. **[エージェントを準備する]** を選択してパッケージ化の完了を待ちます (1 分から 2 分ほど)

2. **[製品内発行フローを続行する]** を選択します

3. 範囲の選択を確認して **[発行]** を選択します

4. 発行が完了するまで待ちます

### Microsoft 365 Copilot でのアクセス

共有スコープを指定して発行されたエージェントは、即座に使用可能になります。

1. **Microsoft 365 Copilot** を開きます (copilot.microsoft.com で、または Microsoft 365 のアプリの中で)

2. エージェント ストアまたは **[拡張機能]** パネルを探します

3. 作成したエージェントを **"あなたのエージェント"** の下で見つけます (共有スコープの場合)

4. 会話を開始します。

    ```
    @Caldova Knowledge Assistant How much headroom does Calderwood have?
    ```

5. または、エージェントを選択して直接質問します。

    ```
    When should we reorder sterile vials, and who is our component supplier?
    ```

6. クエリが Copilot からエージェントに転送されて、Caldova ナレッジ ベースからの情報が返されます

> **注**:**組織スコープ**の場合は、最初に管理者がアプリを [Microsoft 365 管理センター](https://admin.cloud.microsoft/?#/agents/all/requested)の **[要求]** の下で承認する必要があります。 承認されると、エージェントはすべてのユーザーを対象として **"組織による構築"** の下に表示されます。

> ✅ **チェックポイント **: あなたの典拠されたナレッジ エージェントが Microsoft 365 Copilot 内でアクセス可能になり、エンタープライズ ナレッジ ベースからスタッフの質問に答えます。

## クリーンアップ

不要な料金発生を回避するために、終了したらリソースをクリーンアップします。

### エージェントを削除する

1. Foundry ポータルで、**[ビルド]** → **[エージェント]** に移動します

2. **caldova-knowledge-agent** を見つけます

3. **[...]** メニュー → **[削除]** の順に選択します

4. 削除の確定

この操作で、次のものも削除されます。

- Azure Bot Service では、
- 関連付けられた構成
- 発行済みの展開

### Teams からアンインストールする

1. Microsoft Teams を開きます

2. **[アプリ]** → **[アプリの管理]** に移動します

3. **Caldova Knowledge Assistant** を見つけます

4. **[...]** → **[アンインストール]** の順に選択します

5. アンインストールを確認します

### Copilot エージェントを削除する

Copilot に対して発行した場合:

1. 基になるエージェントが削除されると、エージェントは非アクティブになります
2. ユーザーが使用しようとするとエラーが表示されます
3. 管理者が組織カタログからこれを削除する必要がある場合があります

---

**次へ (任意):** [タスク 4 — Work IQ: Microsoft 365 のシグナルをエージェントに取り込む](B4-work-iq-workplace-intelligence.md)
