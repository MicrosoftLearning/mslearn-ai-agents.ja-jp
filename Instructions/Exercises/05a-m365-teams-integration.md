---
lab:
  title: エージェントを Microsoft Teams と Copilot にデプロイする
  description: エンタープライズ アクセスできるように AI エージェントを Microsoft Teams と Microsoft 365 Copilot に対して発行する
  level: 300
  duration: 40
  islab: true
---

# エージェントを Microsoft Teams と Copilot にデプロイする

このラボでは、従業員が既に仕事に使用している場所からアクセスできるように AI エージェントを **Microsoft Teams** と **Microsoft 365 Copilot** に対して発行する方法を学習します。 単純なエージェントを Foundry ポータルで作成し、知識典拠を追加してから、両方のプラットフォームに展開します。

このラボの焦点は、エージェントの開発ではなく**展開と発行のワークフロー**に置いています。

このラボの所要時間は約 **40** 分です。

> **注**:Microsoft 365 Copilot に対して発行するには Copilot のライセンスが必要です。 Teams への展開は標準の Microsoft 365 アカウントで動作します。

## 学習の目的

このラボを終了すると、次のことができるようになります。

1. Microsoft Foundry ポータルで基本的なエージェントを短時間で作成する
2. ファイル検索を使用して知識の典拠を追加する
3. エージェントを Microsoft Teams に対してカスタム アプリとして発行する
4. エージェントを Microsoft 365 Copilot に対して拡張機能として発行する
5. Teams と Copilot の展開の違いを理解する
6. 発行済みエージェントの管理と更新を行う

## 前提条件

このラボを開始する前に、次のものがそろっていることを確認してください。

- [Azure サブスクリプション](https://azure.microsoft.com/free/): AI リソース作成のためのアクセス許可が付与されていること
- **Microsoft 365 アカウント**: Teams にアクセスできること
- **Microsoft 365 Copilot ライセンス** (Copilot 展開の場合に必要)
- Microsoft Foundry ポータルに関する基本的な知識

## シナリオ

ここで展開する **Enterprise Knowledge Agent** には、次の特徴があります。

- 会社のポリシーに関する質問に回答する
- アップロードされたドキュメントを典拠として使用する
- Microsoft Teams チャットを介してアクセスできる
- Copilot 拡張機能として使用できる (オプション)

---

## ポータルでエージェントを作成する

最初に、Microsoft Foundry ポータルを使用してエージェントを短時間で作成します。 この所要時間は約 5 分です。

> **重要**: 更新されたユーザー インターフェイスを使うには、このラボで **[新しい Foundry]** トグルが "オン" になっていることを確認します。**

### Foundry ポータルを開く

1. ブラウザーを開いて Foundry ポータル (`https://ai.azure.com`) に移動し、まだしていない場合はサインインします。
1. **[新しい Foundry]** に切り替えると、プロジェクトを選択するように求められます。 ドロップダウンで **[新しいプロジェクトの作成]** を選びます。
1. **[プロジェクトの作成]** ダイアログで、プロジェクトの有効な名前 (例: *m365-lab*) を入力します。
1. プロジェクトで次の設定を確認または構成します。
    - **Foundry リソース**: *新しい Foundry リソースを作成するか、既存のリソースを選択します*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **[場所]**: *使用できるリージョンを選択する*\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択して、プロジェクトが作成されるまで待ちます。 これには数分かかることがあります。
1. プロジェクトが作成されると、プロジェクトのホーム ページが表示されます。

### 新しいエージェントを作成する

1. ホーム ページの **[ビルドの開始]** で、**[エージェントの作成]** を選びます。
1. エージェントの名前を指定します (例: `enterprise-knowledge-agent`)。
1. **［作成］** を選択します

エージェントの作成時には、既定のモデル (例: `gpt-4.1`) がデプロイされます。 エージェントが作成されると、その既定のモデルが自動的に選ばれたエージェント プレイグラウンドが表示されます。

1. **[手順]** を次の値に設定します。

    ```
    You are an Enterprise Knowledge Assistant for Contoso Corporation.
    
    Your role:
    - Answer questions about company policies and procedures
    - Provide accurate information from uploaded documents
    - Be professional, helpful, and concise
    - If you don't know the answer, say so and suggest who to contact
    
    Always cite your sources when referencing specific policies.
    ```

1. **[保存]** を選んで、現在のエージェント構成を保存します。

### クイック テスト

1. チャット パネルで、次のテスト メッセージを送信します。

    ```
    Hello! What can you help me with?
    ```

2. エージェントが応答して、自分は Enterprise Knowledge Assistant であると説明するはずです

エージェントは動作しますが、会社についての知識はまだ何もありません。 次はそれを追加します。

---

## ファイル検索を使用して知識を追加する

ここでは、エージェントが実際の情報を使って質問に回答できるようにするために会社のドキュメントを追加します。

### ファイル検索を有効にする

1. まず、サンプル ポリシー ドキュメントをダウンロードします。 新しいブラウザー タブを開いて次の各ファイルを保存してください。

    **IT セキュリティ ポリシー:**

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/05a-m365-teams-integration/Python/sample_documents/it_security_policy.txt
    ```

    **リモート ワーク ポリシー:**

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/05a-m365-teams-integration/Python/sample_documents/remote_work_policy.txt

1. Back to your agent's configuration, scroll to the **Tools** section

1. Select **Add** and then **Browse all tools** and **Add tool**.

1. A pop up to attach files will show up, attach the files previously downloaded.

1. Once they complete, select **Attach**.

### Test with knowledge queries

1. In the playground, ask a question about IT security:

    ```
    What are the password requirements for my laptop?
    ```

2. The agent should provide specific information from the IT security policy (minimum 12 characters, uppercase, lowercase, numbers, special characters, etc.)

3. Try a question about remote work:

    ```
    What are the core hours for remote employees?
    ```

4. The agent should respond with information from the remote work policy (9 AM - 3 PM)

5. Try another query:

    ```
    What encryption is required on company laptops?
    ```

6. Notice how the agent finds the right document and provides accurate answers about BitLocker requirements

**Your agent now has knowledge grounding!** It can answer questions based on your company documents.

1. Select **Save**.

---

## Publish to Microsoft Teams

Now you'll publish your agent to Microsoft Teams so employees can chat with it directly in Teams.

### What gets created

When you publish to Teams, the Foundry portal automatically:

- Creates an Azure Bot Service
- Generates a Teams app manifest
- Packages app icons and configuration
- Provides a downloadable app package

### Prepare app information

Before publishing, gather this information:

| Field | Value |
|-------|-------|
| **App Name** | Enterprise Knowledge Agent |
| **Short Description** | AI assistant for company policies |
| **Full Description** | Enterprise AI assistant that answers questions about company policies, IT procedures, and employee resources |
| **Developer Name** | Your name or company name |
| **Website URL** | <https://contoso.com> (placeholder is fine for lab) |
| **Privacy Policy URL** | <https://contoso.com/privacy> |
| **Terms of Use URL** | <https://contoso.com/terms> |

### Create app icons

You'll need two icons for the Teams app:

1. **Color icon** (192x192 pixels)
   - Full color version of your app logo
   - PNG format

2. **Outline icon** (32x32 pixels)
   - White outline on transparent background
   - PNG format
   - Used in the Teams sidebar

> **Quick option for this lab**: Create a simple colored square with text or initials using PowerPoint, Paint, or an online tool like Canva.

### Publish from the portal

1. In the Foundry portal, open your agent (**Build** → **Agents** → **enterprise-knowledge-agent**)

2. Click the **Publish** button at the top of the page

3. Select **Publish to Teams and Microsoft 365 Copilot**.

4. Click **Continue**

### Configure Teams app details

Fill in the configuration form:

**Basic Information:**

- **App Name**: Enterprise Knowledge Agent
- **Short Description**: AI assistant for company policies
- **Full Description**: Enterprise AI assistant that answers questions about company policies, IT procedures, and employee resources

**Developer Information:**

- **Developer Name**: Your name
- **Website**: <https://contoso.com>
- **Privacy Policy**: <https://contoso.com/privacy>
- **Terms of Use**: <https://contoso.com/terms>

**App Icons:**

- Upload your **color icon** (192x192 px)
- Upload your **outline icon** (32x32 px)

**App Scope:**

- Select **Personal** for individual chat access
- Optionally select **Team** for channel access

Click **Prepare Agent**

### Deploy to Teams

After the agent package is prepared (this takes 1-2 minutes), you have two options:

#### Option A: Direct publish (recommended)

This option publishes directly to Teams without manually uploading a package:

1. When the package is ready, select **Continue the in-product publishing flow**

2. Choose your publish scope:
   - **Individual scope**: Agent appears under "Your agents" in the Teams agent store. No admin approval required. Best for personal testing.
   - **Organization (tenant) scope**: Agent appears under "Built by your org" for all users. Requires admin approval.

3. For this lab, select **Individual scope**

4. Click **Submit**

5. Wait for publishing to complete (you'll see a success message)

6. Your agent is now available in Teams! Find it under **Apps** → **Your agents**

#### Option B: Download and manually upload

This option gives you a package to upload manually, useful for testing or when direct publishing isn't available:

1. When the package is ready, click **Download zip**

2. Save the `manifest.zip` file to your computer

3. Open **Microsoft Teams** (desktop app or <https://teams.microsoft.com>)

4. Click **Apps** in the left sidebar

5. Click **Manage your apps** at the bottom left

6. Click **Upload an app** → **Upload a custom app**

7. Browse and select your downloaded `manifest.zip`

8. Review the app details and click **Add**

The app will install and open automatically.

### Test your agent in Teams

1. The agent chat should open after installation (or find it under **Apps** → **Your agents**)

2. Send a greeting:

    ```
    こんにちは。 What can you help me with?
    ```

3. Test a knowledge query:

    ```
    What are the laptop password requirements?
    ```

4. Try another question:

    ```
    What MFA methods are supported?
    ```

5. The agent should respond with information from the IT security policy document!

**🎉 Congratulations!** Your agent is now available in Microsoft Teams!

### Sharing with others

**For personal use:**

- The app is already installed for you

**For team-wide access:**

1. Go to a Team channel
2. Click **+** to add a tab or app
3. Search for your app name
4. Add it to the channel

**For organization-wide access:**

1. Contact your Teams administrator
2. They can publish the app to the organization's app catalog
3. All employees can then find and install it

### Troubleshooting Teams deployment

**Can't find the agent in Teams (after direct publish):**

- Check the **Apps** → **Your agents** section in Teams
- Wait 1-2 minutes for the agent to appear after publishing
- Verify publishing completed successfully in the Foundry portal

**Can't upload the app (manual upload):**

- Ensure the manifest.zip file isn't corrupted (re-download if needed)
- Check that your Teams admin hasn't disabled custom app uploads
- Verify the icons are the correct sizes (192x192 and 32x32)

**Agent doesn't respond:**

- Wait 30 seconds after installation for the bot to initialize
- Check that the Azure Bot Service was created (shown during publishing)
- Test the agent in the Foundry playground first

**Responses are generic (no knowledge):**

- Verify file search is enabled on the agent
- Confirm documents were uploaded and indexed
- Test knowledge queries in the Foundry playground

---

## Publish to Microsoft 365 Copilot

Now you'll publish your agent as a Microsoft 365 Copilot extension, allowing users to access it directly within Copilot.

> **Note**: This section requires a Microsoft 365 Copilot license. If you don't have one, you can read through the steps to understand the process.

### Understanding Copilot extensions

When you publish to Copilot, your agent becomes a **Copilot extension** (also called a plugin or declarative agent). Users can:

- Invoke your agent using @mentions in Copilot
- Access your agent's knowledge alongside Copilot's capabilities
- Switch between Copilot and your agent seamlessly

### Differences: Teams vs Copilot

| Aspect | Teams App | Copilot Extension |
|--------|-----------|-------------------|
| **Access** | Standalone chat in Teams | Within Microsoft 365 Copilot |
| **Invocation** | Open the app directly | @mention or select from extensions |
| **Context** | Isolated conversation | Can combine with Copilot's context |
| **License** | Standard M365 | Requires Copilot license |
| **Discovery** | Teams app store | Copilot extensions panel |

### Publish from the portal

1. Return to the Foundry portal (**<https://ai.azure.com>**)

2. Navigate to your agent (**Build** → **Agents** → **enterprise-knowledge-agent**)

3. Click the **Publish** button

4. Select **Publish to Teams and Microsoft 365 Copilot**

5. Click **Continue**

> **Note**: This is the same publishing flow used for Teams. The agent becomes available in both Teams and Copilot through a single publishing process.

### Configure publishing details

If you haven't already published this agent, fill in the configuration (same as the Teams section):

- **Name**: Enterprise Knowledge Agent
- **Description**: AI assistant for company IT policies
- **Icons**: Upload your 192x192 and 32x32 icons
- **Publisher information**: Your name and placeholder URLs

### Choose publish scope

Select your distribution scope:

| Scope | Visibility | Admin Approval | Best For |
|-------|-----------|----------------|----------|
| **Shared** | Under "Your agents" in agent store | Not required | Personal testing, small teams |
| **Organization** | Under "Built by your org" for all users | Required | Organization-wide distribution |

For this lab, select **Shared scope** for immediate access without admin approval.

### Complete publishing

1. Click **Prepare Agent** and wait for packaging (1-2 minutes)

2. Select **Continue the in-product publishing flow**

3. Confirm your scope selection and click **Publish**

4. Wait for publishing to complete

### Access in Microsoft 365 Copilot

Once published with shared scope, your agent is immediately available:

1. Open **Microsoft 365 Copilot** (copilot.microsoft.com or in Microsoft 365 apps)

2. Look for the agent store or **Extensions** panel

3. Find your agent under **Your agents** (for shared scope)

4. Start a conversation:

    ```
    @Enterprise Knowledge Agent What are the laptop security requirements?
    ```

5. Or select your agent and ask directly:

    ```
    What MFA methods are supported for company systems?
    ```

6. Copilot routes the query to your agent and returns information from the IT security policy

> **Note**: For **organization scope**, an admin must first approve the app in the [Microsoft 365 admin center](https://admin.cloud.microsoft/?#/agents/all/requested) under **Requests**. Once approved, the agent appears under **Built by your org** for all users.

### Managing your published agent

**Update the agent:**

1. Make changes to your agent in the Foundry portal (instructions, documents, tools)
2. Minor changes take effect automatically
3. Major changes may require re-publishing

**Monitor usage:**

1. Check analytics in the Foundry portal
2. Review conversation logs
3. Monitor for errors or issues

**Unpublish:**

1. In the Foundry portal, go to your agent's details
2. Find the publish status section
3. Click **Unpublish** to remove access from Teams and Copilot

---

## Update Published Agents

After publishing, you may need to update your agent. Here's how updates work.

### Making changes

1. In the Foundry portal, open your agent

2. Make your changes:
   - Update instructions
   - Add or remove documents
   - Modify tool settings

3. Click **Save**

### Propagating updates

**For Teams apps:**

- Instruction and document changes take effect immediately
- No need to re-upload the manifest
- Users see updated responses in their next conversation

**For Copilot extensions:**

- Minor changes (instructions, documents) may take effect automatically
- Major changes may require re-submission for approval
- Check the publish status for any pending reviews

### Version management

Best practices for managing agent versions:

1. **Document changes**: Keep a changelog of updates
2. **Test before publishing**: Always test in the playground first
3. **Communicate updates**: Let users know about significant changes
4. **Monitor after updates**: Watch for issues after deploying changes

---

## Cleanup

To avoid unnecessary charges, clean up resources when done.

### Delete the agent

1. In the Foundry portal, go to **Build** → **Agents**

2. Find **enterprise-knowledge-agent**

3. Click the **...** menu → **Delete**

4. Confirm deletion

This also removes:

- The Azure Bot Service
- Associated configurations
- Published deployments

### Uninstall from Teams

1. Open Microsoft Teams

2. Go to **Apps** → **Manage your apps**

3. Find **Enterprise Knowledge Agent**

4. Click **...** → **Uninstall**

5. Confirm uninstallation

### Remove Copilot extension

If you published to Copilot:

1. The extension becomes inactive when the agent is deleted
2. Users will see an error if they try to use it
3. Admin may need to remove it from the organization catalog

---

## Summary

**Congratulations!** 🎉 You've completed this lab!

### What You Accomplished

| Task | Status |
|------|--------|
| Created an agent in the Foundry portal | ✅ |
| Added knowledge with file search | ✅ |
| Published to Microsoft Teams | ✅ |
| Published to Microsoft 365 Copilot | ✅ |
| Learned to update published agents | ✅ |

### Key Takeaways

**Teams deployment:**

- Quick to set up using the Foundry portal
- Creates Azure Bot Service automatically
- Users access via Teams app
- Good for standalone chat experiences

**Copilot deployment:**

- Integrates with Microsoft 365 Copilot
- Users invoke via @mention or selection
- Requires Copilot license
- Good for contextual assistance within Copilot

**Best practices:**

- Test thoroughly in playground before publishing
- Keep documents up to date for accurate responses
- Monitor usage and feedback
- Update instructions based on user needs

### When to Use Each Platform

| Use Case | Recommended Platform |
|----------|---------------------|
| Dedicated support chat | Teams |
| Quick policy lookups | Copilot |
| Team-specific assistant | Teams (channel) |
| Organization-wide knowledge | Both |
| Integration with M365 workflow | Copilot |
| Standalone conversational experience | Teams |

### Next Steps

To build on this lab:

1. **Add more documents** for comprehensive knowledge coverage
2. **Customize instructions** for specific use cases
3. **Add tools** like code interpreter for advanced capabilities
4. **Implement authentication** for sensitive information
5. **Set up monitoring** to track usage and quality

---

## Troubleshooting Reference

### Teams Issues

| Issue | Solution |
|-------|----------|
| Can't upload custom app | Check Teams admin settings for custom app policy |
| App won't install | Verify manifest.zip isn't corrupted; check icon sizes |
| Agent not responding | Test in Foundry playground first; wait 30 seconds after install |
| Generic responses | Verify file search enabled and documents indexed |
| "Bot not found" error | Check Bot Service is running in Azure portal |

### Copilot Issues

| Issue | Solution |
|-------|----------|
| Extension not appearing | Check approval status; may be pending admin review |
| @mention not working | Ensure extension is enabled in your Copilot settings |
| Wrong responses | Test agent in Foundry playground; check document content |
| "Extension unavailable" | Verify agent is running and not deleted |
| Approval rejected | Review rejection reason; update and resubmit |

### General Issues

| Issue | Solution |
|-------|----------|
| Agent not saving | Check browser connection; try refreshing |
| Documents not indexing | Wait a few minutes; try re-uploading |
| Slow responses | Large documents take longer; consider chunking |
| Incorrect citations | Review document content and formatting |

---

## Additional Resources

**Microsoft Documentation:**

- [Publish agents to Teams](https://learn.microsoft.com/azure/ai-services/agents/how-to/publish-to-teams)
- [Copilot extensibility](https://learn.microsoft.com/microsoft-365-copilot/extensibility/)
- [Microsoft Foundry](https://learn.microsoft.com/azure/ai-foundry/)

**Tools:**

- [Foundry Portal](https://ai.azure.com)
- [Microsoft Teams](https://teams.microsoft.com)
- [Microsoft 365 Copilot](https://copilot.microsoft.com)
- [Adaptive Cards Designer](https://adaptivecards.io/designer/)

---

**Lab Complete!** 🎉

You've successfully deployed an AI agent to both Microsoft Teams and Microsoft 365 Copilot!
