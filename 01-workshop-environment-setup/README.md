# :rocket: 01 - ワークショップのための環境をセットアップする

ワークショップをスムーズに進行するためには、環境を正しくセットアップする必要があります。これには、提供されたAzureパスを使用してAzureサブスクリプションを作成し、必要なツールを使用してコード環境を準備することが含まれます。

## Azure Passを引き換える

Azure Passは、Azureで使用するための100ドルのクレジットを提供し、このワークショップの一環としてさまざまなリソースを作成することができます。以下の手順に従ってパスを引き換えてください：

1. 新しいプライベートブラウザウィンドウを開きます。これにより、Azureパスを既存のMicrosoftアカウントや職場のアカウントに誤ってリンクすることを防ぎます。[Microsoft Azure Pass](https://www.microsoftazurepass.com/)のウェブサイトにアクセスし、「Start」ボタンをクリックします。

   ![Azure Pass](images/image01.png "Azure Pass")

2. 新しいAzureアカウントを作成します。

   ![Azure Pass](images/image02.png "Azure Pass")
   ![Azure Pass](images/image03.png "Azure Pass")

3. 提供されたプロモーションコードを入力します。

   ![Azure Pass](images/image04.png "Azure Pass")

パスの引き換えが成功すると、新しいAzureサブスクリプションが設定されます。

## ワークショップのためのコード環境を準備する

ワークショップのためのコード環境を準備するには、以下のツールをインストールする必要があります。

1. [JDK 17](https://docs.microsoft.com/java/openjdk/download?WT.mc_id=azurespringcloud-github-judubois#openjdk-17)
2. VSCode、または IntelliJ for GitHub Copilot
3. [Azure CLIバージョン2.64.0以上](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。現在のAzure CLIインストールのバージョンを確認するには、以下のコマンドを実行します：

    ```bash
    az --version
    ```

4. [Azure Developer CLI](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd?tabs=winget-windows%2Cbrew-mac%2Cscript-linux&pivots=os-windows)

## Azure CLIでログインする

次のコマンドを使用して CLI から Azure にログインし、プロンプトに従って認証します。
```bash
az login
```

次に、本ワークショップに必要なAzure CLI拡張機能をインストールまたは更新します。

```bash
az extension add --name containerapp --upgrade --allow-preview true
az extension add --name serviceconnector-passwordless --upgrade --allow-preview true
```

Azureリソースプロバイダーは、特定のAzureサービスの機能を有効にするためのものです。一部のリソースプロバイダーはデフォルトで登録されています。デフォルトで登録されているリソースプロバイダーのリストについては、[Azureサービスのリソースプロバイダー](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-services-resource-providers)を参照してください。

本ワークショップに必要なAzureリソースプロバイダーを登録します。

```bash
az config set extension.use_dynamic_install=yes_without_prompt
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights
az provider register --namespace Microsoft.ServiceLinker
```

## Azure環境を準備する

このワークショップでは、Azureに必要なリソースをデプロイするためのBicepテンプレートを提供しています。ご自身のパソコンを使用している場合は、まずプロジェクトをクローンしてください。GitHub Codespacesを使用している場合は、プロジェクトはすでにクローンされているので、何もする必要はありません。

```bash
git clone https://github.com/eggboy/aca-java-ai-workshop
cd aca-java-ai-workshop
```

以下のコマンドを実行して、Azureに必要なリソースをデプロイします。以下は出力の例です。

```bash
azd up
? Select an Azure location to use:  1. (Asia Pacific) East Asia (eastasia)

Packaging services (azd package)


Provisioning Azure resources (azd provision)
Provisioning Azure resources can take some time.

Subscription: ... (xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
Location: East Asia

  You can view detailed progress in the Azure Portal:
  https://portal.azure.com/#view/...

  (✓) Done: Resource group: aca-labs (7.168s)
  (✓) Done: Log Analytics workspace: log-vyhjztie4zgue (4.19s)
  (✓) Done: Service Bus Namespace: sb-vyhjztie4zgue (22.363s)
  (✓) Done: Azure OpenAI: cog-vyhjztie4zgue (29.06s)
  (✓) Done: Azure AI Services Model Deployment: cog-vyhjztie4zgue/gpt-4o (30.675s)
  (✓) Done: Container Apps Environment: cae-vyhjztie4zgue (1m46.019s)
  (✓) Done: Azure Database for MySQL flexible server: mysql-vyhjztie4zgue (7m14.054s)

Deploying services (azd deploy)


SUCCESS: Your up workflow to provision and deploy to Azure completed in 8 minutes 3 seconds.
```

これにより、Azureサブスクリプションに以下のリソースが作成されます。
1. リソースグループ
2. Log Analyticsワークスペース
3. Azure OpenAIエンドポイント
4. Azure Container Apps環境
5. Azure Database for MySQL Flexible Server

`azd up`は、Azureサブスクリプションに作成されたリソースの名前を返します。[Azure Portal](https://portal.azure.com)にアクセスして、リソースグループ`aca-labs`内のリソースを確認してください。

## Azure Container Appsのデフォルト設定を構成する

> [!IMPORTANT] Azure Container Appsへの簡単なアクセスを確保するために、デフォルト設定を構成してください。
> ```bash
> az configure --defaults location=koreacentral group=aca-labs
> ```

---

➡️
次へ : [02 - Hello World Spring Bootアプリを作成し、Azure Container Appsにデプロイする](../02-deploy-helloworld/README.md)
