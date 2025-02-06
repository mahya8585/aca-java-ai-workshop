# Azure Container AppsでJavaアプリケーションを監視する

監視は、アプリケーションを本番環境で実行する際の重要な部分です。Azure Container Appsは、Javaアプリケーションを監視するためのいくつかのオプションを提供しています。

---

## 目的

このモジュールでは、以下の3つの主要な目的に焦点を当てます：
1. :white_check_mark: バックエンドサービスのJavaメトリクスを有効にする。
2. :bar_chart: Spring Boot Adminダッシュボードにアクセスする��
3. :airplane: Log Analyticsを使用してログを監視する。

## Javaメトリクスの有効化

Java Virtual Machine (JVM)メトリクスは、Javaアプリケーションの健康状態とパフォーマンスを監視するために重要です。
収集されたデータには、メモリ使用量、ガベージコレクション、JVMのスレッド数に関する洞察が含まれます。

バックエンドサービスのJavaメトリクスを有効にしましょう。

```bash
az containerapp update \
  --name vets-service \
  --enable-java-metrics=true
  
az containerapp update \
  --name customers-service \
  --enable-java-metrics=true

az containerapp update \
  --name visits-service \
  --enable-java-metrics=true
  
```

メトリクスが有効になると、Azureポータルでメトリクスを表示できます。Azureポータルに移動し、「メトリクス」にナビゲートします。

![Java Metrics on Azure Container Apps](images/metrics-1.png)

収集されたメトリクスのリストは[こちら](https://learn.microsoft.com/en-us/azure/container-apps/java-metrics?tabs=create&pivots=azure-cli#collected-metrics)で確認できます。

## Spring Boot Admin

Spring用のAdminマネージドコンポーネントは、アクチュエータエンドポイントを公開するSpring Boot Webアプリケーションの管理インターフェースを提供します。

Azure CLIを使用してSpring Boot AdminのURLを見つけましょう。

```bash
az containerapp env java-component admin-for-spring show \
  --environment ${ACA_ENVIRONMENT_NAME} \
  --name admin --query properties.ingress.fqdn
```

ブラウザを開き、Spring Boot AdminのURLにアクセスします。

![Spring Boot Admin](images/bootadmin.png)

> [!IMPORTANT]
> **メニューをナビゲートし、Spring Boot Adminに慣れてください**

## ログストリーミング

Azureポータルでシステムログとコンソールログを表示できます。システムログはコンテナアプリのランタイムによって生成されます。コンソールログはコンテナアプリ自体によって生成されます。

Azureポータルでコンテナアプリに移動します。サイドバーメニューの「監視」セクションで「ログストリーム」を選択します。コンソールログストリームを表示するには、「コンソール」を選択します。複数のリビジョン、レプリカ、またはコンテナがある場合は、ドロップダウンメニューからコンテナを選択できます。アプリにコンテナが1つしかない場合は、このステップをスキップできます。

![Spring Boot Admin](images/logstream.png)

## Log Analyticsを使用したログの監視

Azure Container Appsは、Azure Monitor Log Analyticsと統合されており、コンテナアプリのログを監視および分析できます。
ログ監視ソリューションとして選択された場合、コンテナアプリ環境には、環境内で実行されているすべてのコンテナアプリからのシステムおよびアプリケーションログデータを保存するためのLog Analyticsワークスペースが含まれます。

Log Analyticsは、Azureポータルでログデータを表示および分析するためのツールです。Log Analyticsを使用すると、Kustoクエリを作成し、結果をチャートで視覚化してトレンドを把握し、問題を特定できます。クエリ結果をインタラクティブに操作したり、アラート、ダッシュボード、ワークブックなどの他の機能と組み合わせて使用��ることができます。

コンテナアプリページのサイドバーメニューで「ログ」からLog Analyticsを開始します。Monitor>LogsからもLog Analyticsを開始できます。

カスタムログカテゴリのテーブルタブにリストされているテーブルを使用してログをクエリできます。このカテゴリのテーブルは、`ContainerAppSystemlogs_CL`および`ContainerAppConsoleLogs_CL`テーブルです。

`ContainerAppSystemlogs_CL`テーブルからcustomers-serviceのアプリケーションログを確認します。

```kql
ContainerAppConsoleLogs_CL
| where ContainerAppName_s == 'customers-service'
| project Time=TimeGenerated, AppName=ContainerAppName_s, Revision=RevisionName_s, Message=Log_s
| take 100
```

`ContainerAppConsoleLogs_CL`テーブルは、Azure Container Appsサービス自体からのログを表示します。

```kql
ContainerAppSystemLogs_CL
| where ContainerAppName_s == 'customer-services'
| project Time=TimeGenerated, EnvName=EnvironmentName_s, AppName=ContainerAppName_s, Revision=RevisionName_s, Message=Log_s
| take 100
```

## :notebook_with_decorative_cover: まとめ

この章では、バックエンドサービスのJavaメトリクスを有効にし、Spring Boot Adminダッシュボードにアクセスし、Log Analyticsを使用してログを監視しました。

---

:arrow_forward::️ 次へ : [08 - Azure OpenAIとSpring AIを使用してAIを組み込んだJavaアプリを作成する](../08-AI-with-java/README.md)

