<properties 
    pageTitle="Azure 自动化中的源代码管理集成 | Microsoft Azure"
    description="本文介绍 Azure 自动化中源代码管理与 GitHub 的集成。"
    services="automation"
    documentationCenter=""
    authors="mgoedtel"
    manager="stevenka"
    editor="tysonn" />    
<tags 
    ms.service="automation"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/23/2016"
    ms.author="magoedte;sngun" />

# Azure 自动化中的源代码管理集成

源代码管理集成可让你将自动化帐户中的 Runbook 关联到 GitHub 源代码管理存储库。源代码管理可让你轻松与团队协作、跟踪更改，以及回滚到旧版 Runbook。例如，源代码管理可让你将源代码管理中的不同分支同步到开发、测试或生产自动化帐户，以轻松地将已在开发环境中测试过的代码提升到生产自动化帐户。

源代码管理可让你将代码从 Azure 自动化推送到源代码管理，或将 Runbook 从源代码管理提取到 Azure 自动化。本文介绍如何在 Azure 自动化环境中设置源代码管理。我们首先将配置 Azure 自动化以访问 GitHub 存储库，并演练可使用源代码管理集成完成的不同操作。


>[AZURE.NOTE] 源代码管理支持提取和推送 [PowerShell 工作流 Runbook](automation-runbook-types.md#powershell-workflow-runbooks) 以及 [PowerShell Runbook](automation-runbook-types.md#powershell-runbooks)。目前不支持[图形 Runbook](automation-runbook-types.md#graphical-runbooks)。<br><br>


为自动化帐户设置源代码管理时，需要执行两个简单的步骤；如果你已有 GitHub 帐户，则只需要执行一个步骤。它们具有以下特点：
## 步骤 1 – 创建 GitHub 存储库

如果你已有想要链接到 Azure 自动化的 GitHub 帐户和存储库，请登录现有帐户并从下面的步骤 2 开始。否则，请导航到 [GitHub](https://github.com/)，注册新帐户并[创建新的存储库](https://help.github.com/articles/create-a-repo/)。


## 步骤 2 – 在 Azure 自动化中设置源代码管理

1. 在 Azure 门户中的“自动化帐户”边栏选项卡上，单击“设置源代码管理”。 
 
    ![设置源代码管理](media/automation-source-control-integration/automation_01_SetUpSourceControl.png)

2. “源代码管理”边栏选项卡随即打开，你可以在其中配置 GitHub 帐户详细信息。下面是要配置的参数的列表：

    |**参数** |**说明** |
    |:---|:---| 
    |选择源 | 选择源。目前仅支持 **GitHub**。 |
    |授权 | 单击“授权”按钮，授予 GitHub 存储库的 Azure 自动化访问权限。如果你已在不同的窗口中登录 GitHub 帐户，将使用该帐户的凭据。成功授权之后，边栏选项卡在“授权属性”之下显示你的 GitHub 用户名。 |
    |选择存储库 | 从可用存储库列表中选择 GitHub 存储库。 |
    |选择分支 | 从可用分支列表中选择分支。如果你尚未创建任何分支，则只显示 **master** 分支。 |
    |Runbook 文件夹路径 | Runbook 文件夹路径可指定 GitHub 存储库中的路径，以便从中推送或提取代码。必须以 **/foldername/subfoldername** 格式输入路径。只有 Runbook 文件夹路径中的 Runbook 才同步到自动化帐户。Runbook 文件夹路径的子文件夹中的 Runbook **不会**同步。使用 **/** 来同步存储库下的所有 Runbook。 |


3. 例如，如果你有名为 **PowerShellScripts** 的存储库，其中包含名为 **RootFolder** 的文件夹，而该文件夹包含名为 **SubFolder** 的文件夹。那么，可以使用以下字符串来同步每个文件夹级别：

    1. 若要从**存储库**同步 Runbook，则 Runbook 文件夹路径为 */*
    2. 若要从 **RootFolder** 同步 Runbook，则 Runbook 文件夹路径为 */RootFolder*
    3. 若要从 **SubFolder** 同步 Runbook，则 Runbook 文件夹路径为 */RootFolder/SubFolder*。
  

4. 配置参数后，它们将显示在“设置源代码管理”边栏选项卡上。
 
    ![配置边栏选项卡](media/automation-source-control-integration/automation_02_SourceControlConfigure.png)


5. 单击“确定”后，随即会针对自动化帐户配置源代码管理集成，并且应会使用 GitHub 信息更新该集成。现在，你可以单击此部分来查看所有源代码管理同步作业历史记录。

    ![存储库值](media/automation-source-control-integration/automation_03_RepoValues.png)

6. 设置源代码管理之后，将在你的自动化帐户中创建以下自动化资源：  
 创建两个[变量资产](automation-variables.md)。
      
    * 变量 **Microsoft.Azure.Automation.SourceControl.Connection** 包含连接字符串的值，如下所示。  

    |**参数** |**值** |
    |:---|:---|
    | Name | Microsoft.Azure.Automation.SourceControl.Connection |
    | 类型 | String |
    | 值 | {"Branch":<*Your branch name*>,"RunbookFolderPath":<*Runbook folder path*>,"ProviderType":<*has a value 1 for GitHub*>,"Repository":<*Name of your repository*>,"Username":<*Your GitHub user name*>} | <br>


    * 变量 **Microsoft.Azure.Automation.SourceControl.OAuthToken** 包含 OAuthToken 的安全加密值。  

    |**参数** |**值** |
    |:---|:---|
    | Name | Microsoft.Azure.Automation.SourceControl.OAuthToken |
    | 类型 | Unknown(Encrypted) |
    | 值 | <*Encrypted OAuthToken*> |  

    ![变量](media/automation-source-control-integration/automation_04_Variables.png)

    * **自动化源代码管理**已作为已授权的应用程序添加到 GitHub 帐户。若要查看应用程序，请从 GitHub 主页导航到“配置文件”>“设置”>“应用程序”。此应用程序可让 Azure 自动化将 GitHub 存储库同步到自动化帐户。  

    ![Git 应用程序](media/automation-source-control-integration/automation_05_GitApplication.png)


## 在自动化中使用源代码管理


### 从 Azure 自动化将 Runbook 签入到源代码管理

Runbook 签入可让你将对 Azure 自动化中的 Runbook 所做的更改推送到源代码管理存储库。以下是签入 Runbook 的步骤：

1. 从自动化帐户[创建新的文本 Runbook](automation-first-runbook-textual.md)，或[编辑现有的文本 Runbook](automation-edit-textual-runbook.md)。此 Runbook 可以是 PowerShell 工作流或 PowerShell 脚本 Runbook。  

2. 编辑 Runbook 之后，将其保存，然后单击“编辑”边栏选项卡中的“签入”。

    ![签入按钮](media/automation-source-control-integration/automation_06_CheckinButton.png)


     >[AZURE.NOTE] 从 Azure 自动化签入会覆盖源代码管理中当前存在的代码。用于签入的 Git 等效命令行指令为 **git add + git commit + git push**

3. 当你单击“签入”时，将出现一条确认消息。请单击“是”继续。

    ![签入消息](media/automation-source-control-integration/automation_07_CheckinMessage.png)

4. 签入启动源代码管理 Runbook：**Sync-MicrosoftAzureAutomationAccountToGitHubV1**。此 Runbook 将连接到 GitHub 并将 Azure 自动化中的更改推送到存储库。若要查看签入作业历史记录，请返回到“源代码管理集成”选项卡，单击以打开“存储库同步”边栏选项卡。此边栏选项卡显示了所有源代码管理作业。选择要查看的作业，然后单击以查看详细信息。

    ![签入 Runbook](media/automation-source-control-integration/automation_08_CheckinRunbook.png)

    >[AZURE.NOTE] 源代码管理 Runbook 是特殊的自动化 Runbook，无法查看或编辑。虽然它们不出现在 Runbook 列表上，但你会看到同步作业显示在作业列表中。
 
5. 修改后的 Runbook 名称将作为输入参数发送到签入 Runbook。在“存储库同步”边栏选项卡中展开 Runbook，即可[查看作业详细信息](automation-runbook-execution.md#viewing-job-status-using-the-azure-management-portal)。

    ![签入输入](media/automation-source-control-integration/automation_09_CheckinInput.png)

6. 在作业完成时刷新 GitHub 存储库可以查看更改。存储库中应有一个提交项，其提交消息为：“已在 Azure 自动化中更新Runbook 名称”。



### 将源代码管理中的 Runbook 同步到 Azure 自动化 

“存储库同步”边栏选项卡上的“同步”按钮可让你将存储库的 Runbook 文件夹路径中的所有 Runbook 提取到自动化帐户。同一个存储库可以同步到多个自动化帐户。以下是同步 Runbook 的步骤：

1. 从设置源代码管理的自动化帐户，打开“源代码管理集成/存储库同步”边栏选项卡并单击“同步”，然后在显示确认消息时，单击“是”继续。  

    ![同步按钮](media/automation-source-control-integration/automation_10_SyncButtonwithMessage.png)

2. 同步启动 Runbook：**Sync-MicrosoftAzureAutomationAccountFromGitHubV1**。此 Runbook 将连接到 GitHub 并将存储库中的更改提取到 Azure 自动化。你应会在此操作的“存储库同步”边栏选项卡中看到新作业。若要查看同步作业的详细信息，请单击以打开作业详细信息边栏选项卡。
 
    ![同步 Runbook](media/automation-source-control-integration/automation_11_SyncRunbook.png)

 
    >[AZURE.NOTE] 从源代码管理进行的同步针对当前在源代码管理中的**所有** Runbook，覆盖当前存在于自动化帐户中的 Runbook 草稿版本。用于同步的 Git 等效命令行指令为 **git pull**


## 排查源代码管理问题

签入或同步作业如有任何错误，作业状态应为“暂停”，你可以在作业边栏选项卡中查看更多错误详细信息。“所有日志”部分显示与该作业关联的所有 PowerShell 流。这可以提供帮助你解决任何签入或同步问题的细节。此外，还会显示同步或签入 Runbook 时发生的操作序列。

![AllLogs 图像](media/automation-source-control-integration/automation_13_AllLogs.png)

## 断开连接源代码管理

若要断开与 GitHub 帐户的连接，请打开“存储库同步”边栏选项卡，然后单击“断开连接”。断开与源代码管理的连接后，前面同步的 Runbook 仍会保留在自动化帐户中，但不启用“存储库同步”边栏选项卡。

  ![断开连接按钮](media/automation-source-control-integration/automation_12_Disconnect.png)



## 后续步骤

有关源代码管理集成的详细信息，请参阅以下资源：
- [Azure Automation: Source Control Integration in Azure Automation（Azure 自动化：Azure 自动化中的源代码管理集成）](https://azure.microsoft.com/blog/azure-automation-source-control-13/)  
- [Vote for your favorite source control system（为你喜爱的源代码管理系统投票）](https://www.surveymonkey.com/r/?sm=2dVjdcrCPFdT0dFFI8nUdQ%3d%3d)  
- [Azure Automation: Integrating Runbook Source Control using Visual Studio Team Services（Azure 自动化：使用 Visual Studio Team Services 集成 Runbook 源代码管理）](https://azure.microsoft.com/blog/azure-automation-integrating-runbook-source-control-using-visual-studio-online/)  

<!---HONumber=Mooncake_0411_2016-->