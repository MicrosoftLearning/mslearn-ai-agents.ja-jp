---
title: '作業の開始: 環境を設定する'
lab:
  title: '作業の開始: 環境を設定する'
  description: '「Agent Framework を使用してマルチエージェント ソリューションを構築する」ラボの共有セットアップ: Microsoft Foundry プロジェクトを作成し、スタート コードを入手し、環境を構成します。 どのタスクでも、事前にこれを 1 回完了してください。'
  type: task
  parent: C
  order: 0
  section: setup
  access: open
  level: 300
  concepts: 'environment setup, Microsoft Foundry project'
  status: draft
---

# 概要

このページでは、「**Agent Framework を使用してマルチエージェント ソリューションを構築する**」ラボで必要なすべてのものを設定します。 **すべてのタスクはここから始まります**。最初にこのページを完了してください。 各タスクは、単独で実行できるように記述されています。ラボ全体を一気に進める場合は、このセットアップは 1 回だけで済みます。

**シナリオ:** あなたは、製薬会社 **Caldova** で働いており、会社は、前倒しされた製品発売の準備をしているところです。 このラボでは、Caldova の業務を支援する自動化を構築します。まず、1 つのエージェントを構築し、調整されたエージェント チームに拡張します。

> **注**: このラボで使用されるテクノロジの一部は、プレビューまたは開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

開始する前に、次のことを確認します。

- Azure AI リソースをプロビジョニングするための十分なアクセス許可とクォータを持つ [Azure サブスクリプション](https://azure.microsoft.com/free/)
- ローカル コンピューターにインストールされている [Visual Studio Code](https://code.visualstudio.com/)
- [Python 3.13](https://www.python.org/downloads/) がインストールされていること
- ローカル コンピューターに [Git](https://git-scm.com/downloads) がインストールされている
- Python に関する基本的な知識

> \* Python 3.14 はまだサポートされていません。一部の依存関係には 3.14 ビルドがありません。 このラボは Python 3.13.12 でテストされました。

## Microsoft Foundry プロジェクトを作成する

すべてのタスクで Foundry プロジェクトとデプロイ済みのモデルが必要です。 これらはポータルで作成する (既定) か、Azure Developer CLI (`azd`) で 1 つのコマンドでプロビジョニングすることができます。

### オプション A - ポータルでプロジェクトを作成する (既定)

Microsoft Foundry では、プロジェクトを使用して、モデル、リソース、データ、その他の資産を整理します。

1. Web ブラウザーで、[Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 ヒントやクイック スタートのペインをすべて閉じ、必要な場合は、左上にある **Foundry** のロゴを使ってホーム ページに移動します。

    > **重要**: このラボでは、**新しい** Foundry エクスペリエンスを使用しています。

1. 上部のバナーで、**[構築の開始]** を選択します。

1. プロンプトが表示されたら、**新規**プロジェクトを作成し、有効な名前 (`agents-lab-project` など) を入力します。

1. **[詳細オプション]** を展開して、次のように指定します。
    - **Microsoft Foundry リソース**:"Foundry リソースの有効な名前"**
    - **リージョン**: 近くから利用可能なものを選択します**\*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]**: *リソース グループを選択または作成します*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 後でクォータ制限に達した場合、別の Azure リージョンで別のリソースを作成することが必要になる場合があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。 促されたら、ウェルカムダイアログを進めます。

1. モデルのデプロイを求められたら、**gpt-4o** モデル (または別の利用可能なチャット モデル) をデプロイします。 **デプロイ名**をメモします。これを `.env` で `MODEL_DEPLOYMENT_NAME` として設定します。

1. プロジェクトの概要から**プロジェクト エンドポイント** をコピーします。これを `PROJECT_ENDPOINT` として設定します。

### オプション B - azd を使用してプロビジョニングする (省略可能、1 つのコマンド)

ポータルを何度もクリックしたくない場合、ラボには Foundry リソース、プロジェクト、モデル デプロイを作成するオプションの `azd` テンプレートが付属しています。

1. [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd) をインストールします。

1. `Labfiles/C-build-multi-agent-solutions-with-agent-framework` フォルダーから、次を実行します。

    ```
    azd auth login
    azd up
    ```

1. プロンプト (環境名、Azure リージョン) に回答します。 完了すると、`azd` によって `PROJECT_ENDPOINT` と `MODEL_DEPLOYMENT_NAME` が `Python/.env` に書き込まれます。

    > **注**: ラボが完了したら、`azd down` を実行して、作成されたものをすべて削除します。

## スタート コードを取得する

1. VS Code で、コマンド パレットを開き (**Ctrl + Shift + P** キー)、**Git: Clone** を実行して、次のように入力します。

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. クローンされたリポジトリを開き、**[ファイル] > [フォルダーを開く]** を選択し、`mslearn-ai-agents/Labfiles/C-build-multi-agent-solutions-with-agent-framework/Python` を選択します。 この 1 つのフォルダーに、このラボの**すべて**のタスクのスタート コードが入っています。全体を通して 1 つの仮想環境と 1 つの `.env` を使用します。

1. **requirements.txt** を右クリックし、**[統合ターミナルで開く]** を選択します。 その後、仮想環境を作成し、パッケージをインストールします。

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

1. **.env** ファイルを開いて、`PROJECT_ENDPOINT` をお使いのプロジェクト エンドポイントに設定し、`MODEL_DEPLOYMENT_NAME` をお使いのモデル デプロイ名に設定します  ファイルを保存します。 (`azd up` を使用した場合、これらは既に入力されています。)

    > **ヒント**: Foundry Toolkit VS Code 拡張機能で、プロジェクトのデプロイを右クリックし、**[プロジェクト エンドポイントのコピー]** を選択してエンドポイント URL を取得します。

    > `.env` には、`SERVER_URL` と 3 つの `*_PORT` 値が事前に入力されており、これらは**タスク 3** (リモート エージェント) でのみ使用され、通常、変更する必要はありません。

## タスクの準備ができていることを確認する

各タスクには、`.env` 内に固有の値が必要です。 タスクを開始する前に、VS Code で開いた `Python` フォルダーからプレフライト チェックを実行します。これにより、`.env` が読み取られ、不足しているものがある場合は指示されます。

```
python ../setup/check_env.py --task 1
```

`1` をこれから始めるタスク番号に入れ替えます。

> **ヒント**: プレフライト チェックは Python 標準ライブラリのみを使用するため、`pip install` の前や仮想環境がアクティブでない場合でも安全に実行できます。

以上です。次の任意のタスクに進んでください。

| タスク | ページ |
| --- | --- |
| タスク 1 - ツールを使用するエージェントを構築する | [C1](C1-create-an-agent-with-a-tool.md) |
| タスク 2 - 複数のエージェントを順番にオーケストレーションする | [C2](C2-orchestrate-multiple-agents.md) |
| タスク 3 - リモート エージェントを A2A で接続する | [C3](C3-connect-remote-agents-with-a2a.md) |
| タスク 4 - サポート チケットを分類してルーティングする | [C4](C4-classify-and-route-a-ticket.md) |
