# :rocket: Azure Service Connectorを使用する

Azure Service Connectorは、アプリケーションを他のバックエンドサービスに接続するのに役立ちます。Service Connectorは、アプリケーションサービスとターゲットバックエンドサービス間のネットワーク設定と接続情報（環境変数の生成など）を管理プレーンで構成します。開発者は、接続情報を消費するための好みのSDKやライブラリを使用して、ターゲットバックエンドサービスに対してデータプレーン操作を行います。

このラボでは、Azure Service Connectorを使用して、コード内で接続文字列を公開せずにJavaアプリケーションをAzure Database for MySQL Flexible Serverに接続する方法を学びます。しかし、始める前に、Azure SDK内の認証メカニズムの基本を理解しましょう。

## 目的

このモジュールでは、以下の4つの主要な目的に焦点を当てます：

1. :white_check_mark: Azure SDKを使用したAzureリソースのアプリケーション認証の基本を理解する。
2. :bar_chart: マネージドIDとそのサービスプリンシパルに対する利点を理解する。
3. :mag: MySQLデータベースに接続するためにバックエンドサービスをマネージドIDで構成する。
4. :airplane: Service Connectorを使用したパスワードレス接続の概念を理解する。

---

## :book: 基本 - Azureリソースのアプリケーション認証

アプリがAzureリソースにアクセスする必要がある場合、アプリはAzureに認証される必要があります。これは、Azureにデプロイされたアプリ、オンプレミスにデプロイされたアプリ、またはローカル開発者ワークステーションで開発中のアプリのすべてに当てはまります。

![DefaultAzureCredential](images/appauth.png)

Azure��アプリケーションを認証するためのアプローチは3つあります：

1. 開発者アカウント - VSCode、IntelliJ、Azure CLI、またはPowerShellから取得されたID
2. サービスプリンシパル - Azure Entra IDから取得されたID
3. マネージドID - AzureマネージドIDから取得されたID

サービスプリンシパルとマネージドIDのどちらを選択するかは、特定のシナリオに依存します。ただし、一般的な優先順位は、可能なすべてのシナリオでまずマネージドIDを使用し、マネージドIDがサポートされていない場合にサービスプリンシパルに戻ることです。良い例としては、Azure外でホストされているアプリ（たとえば、オンプレミスのアプリ）がAzureサービスに接続する必要がある場合です。

## :book: 基本 - なぜAzureマネージドIDなのか？

サービスプリンシパルは、伝統的にAzureリソースにアプリケーションを認証する一般的な方法でしたが、重大な欠点があります：それは単なるIDとシークレットの組み合わせです。Azure Key Vaultにシークレットを安全に保存することは可能ですが、サービスプリンシパルを介してAzure Key Vaultにアクセスすることはセキュリティリスクを引き起こします。

マネージドIDは、Microsoft Entra ID内で自動的に管理されるIDを提供することでこの問題を軽減します。この機能により、アプリケーションはマネージドIDを使用してMicrosoft Entraトークンを取得し、手動で資格情報を管理する必要がなくなります。

マネージドIDには2つのタイプがあります：

- システム割り当て。仮想マシンなどの一部のAzureリソースでは、リソース上でマネージドIDを有効にすることができます。システム割り当てマネージドIDを有効にすると：
  - 特殊なタイプのサービスプリンシパルがMicrosoft Entra IDに作成されます。このサービスプリンシパルは、そのAzureリソースのライフサイクルに結び付けられています。Azureリソースが削除されると、Azureは自動的にサービスプリンシパルを削除します。
  - 設計上、そのAzureリソースのみがこのIDを使用してMicrosoft Entra IDからトークンを要求できます。
  - マネージドIDに1つ以上のサービスへのアクセス権を付与します。
- ユーザー割り当て。スタンドアロンのAzureリソースとしてマネージドIDを作成することもできます。ユーザー割り当てマネージドIDを作成し、1つ以上のAzureリソースに割り当てることができます。ユーザー割り当てマネージドIDを有効にすると：
  - 特殊なタイプのサービスプリンシパルがMicrosoft Entra IDに作成されます。このサービスプリンシパルは、使用するリソースとは別に管理されます。
  - ユーザー割り当てマネージドIDは複数のリソースで使用できます。
  - マネージドIDに1つ以上のサービスへのアクセス権を付与します。

## :book: 基本 - Spring Cloud Azure（Spring Boot用Azure SDK）

Spring Cloud Azureは、AzureとのシームレスなSpring統合を提供するオープンソースプロジェクトです。認証に関しては、Spring Cloud Azureは`DefaultAzureCredential`を使用します。これは、開発環境と本番環境のための簡素化された認証メカニズムを提供することを目的としています。これは、順番に試行される資格情報のチェーンであり、最初に利用可能な資格情報が認証に使用されます。このアプローチにより、アプリは環境固有のコードを実装することなく、異なる環境（ローカル開発と本番環境）で異なる認証方法を使用できます。

![DefaultAzureCredential](images/DefaultAzureCredential.png)

Spring Cloud AzureでマネージドIDを構成するには、`application.properties`ファイルに次のプロパティを設定する必要があります：

```properties
spring.cloud.azure.credential.managed-identity-enabled=true
spring.cloud.azure.credential.client-id=<マネージドIDのクライアントID>
```

## :book: 基本 - Service Connectorを使用したパスワードレス接続

パスワードレス接続は、マネージドIDを使用してAzureサービスにアクセスします。このアプローチでは、マネージドIDのシークレットを手動で追跡および管理する必要がありません。これらのタスクはAzureによって内部的に安全に処理されます。

Service Connectorは、Azure Spring Apps、Azure App Service、およびAzure Container AppsなどのアプリホスティングサービスでマネージドIDを有効にします。Service Connectorは、Azure Database for PostgreSQL、Azure Database for MySQL、およびAzure SQL Databaseなどのデータベースサービスを構成して、マネージドIDを受け入れるようにします。

### Service Connectorの作成

以前にデプロイされたバックエンドサービスはインメモリデータベースを使用しています。この章では、Service Connectorを作成して、バックエンドサービスをデータベースとしてAzure Database for MySQL Flexible Serverに接続します。

まず、Azure CLIを使用してMySQLデータベースのためのマネージドIDを作成します。

```bash
az identity create --name aca-mysql-mi
```

次に、[Azureポータル](https://portal.azure.com/)からService Connectorの作成を開始します。まず、`vets-service`から始めます。
以下の画面に従って接続を作成します。

![DefaultAzureCredential](images/serviceconnector-1.png)

前のステップで作成したマネージドIDを選択します。

![DefaultAzureCredential](images/serviceconnector-2.png)

`Revew + Create`ページで、Service Connectorを作成するコマンドが表示されます。以下のようになります。

```bash
az containerapp connection create mysql-flexible \
--connection mysql_a4696 \
--source-id /subscriptions/../resourceGroups/.../providers/Microsoft.App/containerApps/vets-service \
--target-id /subscriptions/.../resourceGroups/.../providers/Microsoft.DBforMySQL/flexibleServers/jay-aca-java-labs/databases/aca-labs \
--client-type springBoot \
--user-identity client-id=28237d6c.. subs-id=6535fca9.. mysql-identity-id=/subscriptions/.../resourcegroups/.../providers/Microsoft.ManagedIdentity/userAssignedIdentities/aca-mysql-mi 
-c vets-service-build42982
```

コマンドを実行すると、Service ConnectorはMySQLをパスワードなしの接続で構成し、ファイアウォールルールを開き、上記のCLIコマンドで指定された`springBoot`サービスタイプのためにいくつかの環境変数を注入します。Service Connectorは、指定されたサービスタイプとターゲットサービスに応じて異なる環境変数を注入します。例として[Azure Service Bus](https://learn.microsoft.com/en-us/azure/service-connector/how-to-integrate-service-bus?tabs=dotnet)を参照してください。

Azureポータルに戻り、新しく作成されたService Connectorを確認します。

注入された環境変数`spring.datasource.azure.passwordless-enabled=true`に注意してください。これにより、Spring Cloud AzureがマネージドIDを使用してMySQLデータベースに接続できるようになります。

![DefaultAzureCredential](images/serviceconnector-3.png)

最後に、`vets-service`でSPRING_ACTIVE_PROFILEを`passwordless`に指定します：

```bash
az containerapp update --name vets-service --set-env-vars SPRING_PROFILES_ACTIVE=passwordless
```

> 💡 [!NOTE]
> `passwordless`プロファイルはMySQLにテーブルを自動的に作成します。MySQL Workbenchなどのクライアントツールを使用してテーブルを確認してください。

> [!IMPORTANT]
> **同じ手順をcustomers-serviceおよびvisits-serviceにも繰り返してください**



## :notebook_with_decorative_cover: まとめ

---

➡️
:arrow_forward::️ 次へ : [07 - Azure Container AppsでJavaアプリケーションを監視する](../07-monitoring-java-aca/README.md)
