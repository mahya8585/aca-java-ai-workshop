# :rocket: KEDA(Kubernetesベースのイベント駆動型オートスケーラー)を使用したオートスケーリング

## 目的

このモジュールでは、以下の3つの主要な目的に焦点を当てます：

1. :white_check_mark: KEDA (Kubernetes Event-Driven Autoscaler)について学ぶ
2. :bar_chart: 異なるスケーリングルールを作成する
3. :mag: Azure Container Appsのスケーリングをテストする

## Azure Container Appsのオートスケーリングオプション

Azure Container Appsでは、3つの異なるカテゴリの[スケーリングトリガー](https://learn.microsoft.com/ja-jp/azure/container-apps/scale-app?pivots=azure-cli)があります。

1. HTTP: リビジョンへの同時HTTPリクエストの数に基づいてスケーリングします。
2. TCP: リビジョンへの同時TCP接続の数に基づいてスケーリングします。
3. カスタム(KEDAベース): CPU、メモリ、または以下のようなサポートされているイベント駆動型データソースに基づいてスケーリングします：

- Azure Service Bus
- Azure Event Hubs
- Apache Kafka
- Redis
- その他多数

カスタムスケーリングトリガーは、内部でKEDAを使用します。これについては、次の章で詳しく説明します。

> :warning: **スケーリングルールを追加または変更すると、コンテナアプリの新しいリビジョンが作成されることに注意してください。**

## HTTPスケーリングルール

HTTPスケーリングルールでは、コンテナアプリのリビジョンがスケーリングされる同時HTTPリクエストのしきい値を制御できます。15秒ごとに、過去15秒間のリクエスト数を15で割った値として同時リクエスト数が計算されます。ワークショップのために、同時実行レベルを1に設定し、ブラウザで簡単にスケーリングをシミュレートできるようにしましょう。

スケーリングルールは、Azure CLIまたはAzureポータルを使用して更新できます。CLIを使用する方法は次のとおりです：

```bash
az containerapp update \
  --name helloworld \
  --min-replicas 1 \
  --max-replicas 5 \
  --scale-rule-name azure-http-rule \
  --scale-rule-type http \
  --scale-rule-http-concurrency 1
```

ターミナルを開き、`az containerapp logs`を実行してレプリカの増加を確認します。

```bash
az containerapp logs show \
  --name helloworld \
  --type=system \
  --follow=true
```

![システムログに表示されるレプリカの変更](images/http-1.png)

[Azureポータル](https://portal.azure.com)に移動し、`Revisions and replicas`を確認します。`Replicas`では、複数のレプリカが表示されるはずです。

![システムログに表示されるレプリカの変更](images/http-2.png)

メトリクスでは、レプリカごとの総リクエスト数を確認できます。

![システムログに表示されるレプリカの変更](images/http-3.png)

## KEDAとは何か？

**KEDA**は、Kubernetesベースのイベント駆動型オートスケーラーです。KEDAを使用すると、処理する必要のあるイベントの数に基づいてKubernetes内の任意のコンテナのスケーリングを駆動できます。

**KEDA**は、単一目的の軽量コンポーネントであり、任意のKubernetesクラスターに追加できます。KEDAは、Horizontal Pod Autoscalerなどの標準的なKubernetesコンポーネントと連携し、機能を拡張できます。KEDAを使用すると、イベント駆動型スケーリングを使用するアプリを明示的にマッピングでき、他のアプリは引き続き機能します。これにより、KEDAは他のKubernetesアプリケーションやフレームワークと並行して実行するための柔軟で安全なオプションとなります。

## Azure Service Busスケーリングルール

次に、Azure Service Busを使用してカスタムスケーリングルールをテストします。まず、前のモジュールで作成したAzure Service Busの接続文字列とキュー名を見つける必要があります。キュー名は`keda`であり、接続文字列はAzureポータルで確認できます。

![Service Bus Managed Key](images/servicebus-1.png)

**ここで、どのシナリオを選択するかを決定します。シナリオ1は、GitHub Copilotを使用してAzure Service Busパブリッシャーを作成するシナリオです。このシナリオを選択する場合は、シナリオ2も確認することをお勧めします。シナリオ2を選択する場合は、まずシナリオを確認してください。GitHub Copilotが開発プロセスでどのように役立つかを確認したい場合は、シナリオ1も確認してください。**

<details markdown="block">
<summary>:rocket: シナリオ1 - GitHub Copilotを使用してAzure Service Busパブリッシャーを作成する</summary>

最初のステップは、`pom.xml`ファイルにAzure Service Busの依存関係を追加することです。`pom.xml`ファイルを開き、次の内容を追加します：

```xml
		<dependency>
			<groupId>com.azure</groupId>
			<artifactId>azure-messaging-servicebus</artifactId>
			<version>7.17.6</version>
		</dependency>

```

GitHub Copilot Chatを使用してコードを生成します。以下のスクリーンキャプチャを参照してください。

![Copilot Chat with Prompt](images/ghcp-1.png)

ここで重要な詳細は次の2つです：
1. `pom.xml`と`HelloController.java`の2つのタブが開かれています。
2. GitHub Copilot Chatの実際のプロンプトは次のとおりです：
```plaintext
1. Create an RESTful endpoint "/message" and read the Path Variable following the endpoint. Endpoint should be GET method.
2. Send received message to Azure Service Bus Queue using client.
3. Have everything in the single file for simplicity.
4. Suggest properties in application.properties for Azure Service Bus client.
```

生成されたコードは次のようになります。**注意**：GitHub Copilotはプロンプトに基づいてコードを合成するため、コードは異なる場合があります。

```java
package com.example.demo;

import com.azure.messaging.servicebus.ServiceBusClientBuilder;
import com.azure.messaging.servicebus.ServiceBusSenderClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Value("${azure.servicebus.connection-string}")
    private String connectionString;

    @Value("${azure.servicebus.queue-name}")
    private String queueName;

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Azure Container Apps\n";
    }

    @GetMapping("/message/{msg}")
    public String sendMessage(@PathVariable String msg) {
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
            .connectionString(connectionString)
            .sender()
            .queueName(queueName)
            .buildClient();

        senderClient.sendMessage(new com.azure.messaging.servicebus.ServiceBusMessage(msg));
        senderClient.close();

        return "Message sent to Azure Service Bus Queue: " + msg;
    }
}
```

GitHub Copilotは、`application.properties`ファイルに次のプロパティを追加するように提案するはずです。前のステップでメモした値を使用してください。

```properties
azure.servicebus.connection-string=YOUR_SERVICE_BUS_CONNECTION_STRING
azure.servicebus.queue-name=YOUR_QUEUE_NAME
```

次に、アプリケーションを実行し、新しいRESTエンドポイントを次のようにテストします。正しく呼び出された場合、次のようなメッセージが表示されるはずです。

![New REST endpoint](images/servicebus-2.png)

Azure Service Busキューにエンキューされたメッセージを確認します。Azureポータルでエンキューされたメッセージを表示できます。Azureポータルに移動し、Service Busキューを見つけ、`Service Bus Explorer`をクリックします。`Peek Mode`を使用してキュー内のメッセージを表示できます。`Peek from start`をクリックして、キュー内のすべてのメッセージを表示します。

![Peek Mode](images/servicebus-3.png)
</details>

<details markdown="block">
<summary>:rocket: シナリオ2 - 手動でAzure Service Busキューにメッセージを公開する</summary>

Azureポータルで、Service Busキューに移動し、`Service Bus Explorer`をクリックします。上部にある`Send Messages`ボタンをクリックして、キューにメッセージを送信します。`Repeat Send`ボタンを使用して、一度に複数のメッセージを送信できます。

![Sens Messages](images/servicebus-4.png)

</details>

Azure Service Busキューにメッセージを公開する準備が整いました。KEDAはAzure Service Busに接続するために接続文字列を必要とします。理想的には、セキュリティリスクを最小限に抑えるためにマネージドIDを使用するべきです。しかし、簡単のために接続文字列を使用します。マネージドIDについては、次のモジュールで詳��く説明します。

Azure Container Appsに接続文字列を安全に保存するためのシークレットを作成します。次のコマンドに正しい接続文字列を挿入します。

```bash
az containerapp secret set \
        --name helloworld \
        --secrets service-bus-connection-string="Endpoint=..."
````

コンテナアプリを再起動して、新しいシークレットをAzure Container Appsに適用します。まず、アクティブなリビジョンを見つけます。

```bash
az containerapp revision list -n helloworld -o table
````

次に、アクティブなリビジョンを再起動します。例：

```bash
az containerapp revision restart -n helloworld --revision helloworld--50kr6mp
```

コンテナアプリが再起動されたら、Azure CLIを使用してスケーリングルールを作成できます。

```bash
az containerapp update \
  --name helloworld \
  --min-replicas 0 \
  --max-replicas 5 \
  --scale-rule-name azure-servicebus-queue-rule \
  --scale-rule-type azure-servicebus \
  --scale-rule-metadata "queueName=keda" \
                        "namespace=service-bus-namespace" \
                        "messageCount=1" \
  --scale-rule-auth "connection=service-bus-connection-string"
```

スケーリングルールが作成されました。次に、Azure Service Busキューにメッセージを公開してみてください。コンテナアプリのスケーリングが確認できるはずです。また、キュー内のメッセージを削除してレプリカの減少を確認します。

```bash
az containerapp revision list -n helloworld -o table
CreatedTime                Active    Replicas    TrafficWeight    HealthState    ProvisioningState    Name
-------------------------  --------  ----------  ---------------  -------------  -------------------  -------------------
2024-11-17T01:06:02+00:00  True      4           100              Healthy        Provisioned          helloworld--az242pu
```

> 💡 __注意:__ スケーリングが期待通りに動作しない場合は、ログを確認して原因を特定してください。Azureポータルに移動し、`Logs`セクションでログを確認します。
> ```kusto
> ContainerAppSystemLogs_CL
> | where Reason_s == "KEDAScalerFailed"
>| project TimeGenerated, ContainerAppName_s, ReplicaName_s, Log_s, Reason_s
>```

## :notebook_with_decorative_cover: まとめ

このモジュールでは、KEDAについて学び、Azure CLIを使用してスケーリングルールを作成する方法を学びました。Azure Service Busを使用してスケーリングのテストを行いました。次に、Azure Container AppsでマネージドJavaコンポーネントを作成する方法を学びます。

---

:arrow_forward:
次へ : [04 - Azure Container AppsでマネージドJavaコンポーネントを作成する](../04-create-managed-java-component/README.md)
