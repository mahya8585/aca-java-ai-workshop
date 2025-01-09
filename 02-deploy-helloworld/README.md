# :rocket: 02 - Hello World Spring Bootアプリを作成し、Azure Container Appsにデプロイする

# 目的

このモジュールでは、以下の4つの主要な目的に焦点を当てます：

1. :white_check_mark: HelloWorld Spring Bootアプリケーションを開発し、Azure Container Appsにデプロイする
2. :bar_chart: Azure CLIおよびAzureポータルを通じてログを監視する方法を学ぶ
3. :mag: Azure Container Appsのスケーリング方法を理解する
4. :airplane: レディネスプローブとリビジョン作成の設定に慣れる

## `helloworld`アプリを作成しましょう

Spring Bootアプリケーションを作成する人気の方法は、Spring Initializerを使用することです。これは[https://start.spring.io/](https://start.spring.io/)で見つけることができます。

![Spring Initializr](images/spring-initializr.jpg)

> Codespacesを使用している場合は、以下の手順に従って新しいSpring Bootプロジェクトを作成してください。

```bash
mkdir helloworld
cd helloworld
curl https://start.spring.io/starter.tgz -d dependencies=web,actuator,azure-support -d bootVersion=3.2.11 -d name=helloworld -d type=maven-project | tar -xzvf -
```

> このラボでは、Spring Bootのバージョンを3.2.11に固定し、`com.example.demo`パッケージを使用してデフォルト設定を維持します。

`src/main/java/com/example/demo`ディレクトリに移動し、`DemoApplication.java`ファイルと同じパッケージに`HelloController.java`という名前の新しいファイルを作成し、以下の内容を追加します：

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Azure Container Apps\n";
    }
}
```

![Hello World](images/helloworld.jpg)

## プロジェクトをローカルでテストする

プロジェクトを実行します：

```bash
./mvnw spring-boot:run
```

`/hello`エンドポイントにリクエストを送信すると、「Hello from Azure Container Apps」というメッセージが返されるはずです。

![Hello World](images/helloworld-browser.jpg)

上記の手順により、hello-worldアプリがローカルで問題なく動作していることが確認されます。

## Spring BootをAzure Container Appsに作成してデプロイする

以下のコマンドを使用してCLIからアプリインスタンスを作成します：

```bash
az containerapp create --name helloworld --environment ${ACA_ENVIRONMENT_NAME} --source . --ingress external --target-port 8080 --query properties.configuration.ingress.fqdn
```

このコマンドは、Spring BootプロジェクトをAzure Container Appsにデプロイします。内部的には、Azure Container AppsはOryx Builderを使用しており、Cloud Native Buildpackに依存してコンテナイメージをビルドします。`--query`パラメータは、アプリインスタンスにアクセスするために使用される完全修飾ドメイン名（FQDN）を抽出します。以下は出力の例です：

```bash
Your container app helloworld has been created and deployed! Congrats!

Your app is running image caa9d8d23f12acr.azurecr.io/helloworld:cli-containerapp-20241107003317549357 and listening on port 8080
Browse to your container app at: http://helloworld.yellowgrass-143599e3.southeastasia.azurecontainerapps.io

Stream logs for your container with: az containerapp logs show -n helloworld -g sandbox-rg

See full output using: az containerapp show -n helloworld -g sandbox-rg
```

この出力は、アプリが正常にデプロイされたことを示しています。FQDNは`http://helloworld.yellowgrass-143599e3.southeastasia.azurecontainerapps.io`であり、このインスタンスにアクセスするために使用できます。

![Hello World](images/helloworld-aca.png)

## Azure CLIでログを表示する

ターミナルに戻り、以下のコマンドを実行してアプリインスタンスのログを表示します

```bash
az containerapp logs show -n helloworld
{"TimeStamp": "2024-11-06T16:37:39.83094", "Log": "Connecting to the container 'helloworld'..."}
{"TimeStamp": "2024-11-06T16:37:39.88611", "Log": "Successfully Connected to container: 'helloworld' [Revision: 'helloworld--84iwngl-65d4f76d4-jlbt8', Replica: 'helloworld--84iwngl']"}
{"TimeStamp": "2024-11-06T16:37:39.8864994Z", "Log": " /\\\\ / ___'_ __ _ _(_)_ __  __ _ \\ \\ \\ \\"}
{"TimeStamp": "2024-11-06T16:37:39.8866142Z", "Log": "( ( )\\___ | '_ | '_| | '_ \\/ _` | \\ \\ \\ \\"}
{"TimeStamp": "2024-11-06T16:37:39.8866782Z", "Log": " \\\\/  ___)| |_)| | | | | || (_| |  ) ) ) )"}
{"TimeStamp": "2024-11-06T16:37:39.8867333Z", "Log": "  '  |____| .__|_| |_|_| |_\\__, | / / / /"}
{"TimeStamp": "2024-11-06T16:37:39.886785Z", "Log": " =========|_|==============|___/=/_/_/_/"}
{"TimeStamp": "2024-11-06T16:37:39.8868402Z", "Log": ""}
{"TimeStamp": "2024-11-06T16:37:39.8868959Z", "Log": " :: Spring Boot ::                (v3.3.5)"}
{"TimeStamp": "2024-11-06T16:37:39.8869479Z", "Log": ""}
{"TimeStamp": "2024-11-06T16:34:24.511+00:00", "Log": "INFO 1 --- [demo] [           main] com.example.demo.DemoApplication         : Starting DemoApplication v0.0.1-SNAPSHOT using Java 17.0.10 with PID 1 (/workspace/BOOT-INF/classes started by cnb in /workspace)"}
{"TimeStamp": "2024-11-06T16:34:24.514+00:00", "Log": "INFO 1 --- [demo] [           main] com.example.demo.DemoApplication         : No active profile set, falling back to 1 default profile: \"default\""}
{"TimeStamp": "2024-11-06T16:34:25.511+00:00", "Log": "INFO 1 --- [demo] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)"}
```

## Azureポータルでログを表示する

コンソール出力をストリーミングすることは、マイクロサービスの現在の状態をより深く理解するのに役立ちます。
ただし、過去のログをさらに調べたり、特定のものをログで探し���りする必要がある場合があります。これはLog Analyticsを使用して簡単に行えます。

[Azureポータルを開く](https://portal.azure.com)し、`helloworld`コンテナアプリに移動します。「ログ」をクリックします。これは、以前に作成されたLog Analyticsワークスペースへのショートカットです。チュートリアルが表示された場合は、今はスキップしても構いません。

このワークスペースでは、集計されたログに対してクエリを実行できます。最も一般的なクエリは、特定のアプリケーションの最新のログを取得することです：

__重要：__ アプリケーションログには専用の`ContainerAppConsoleLogs_CL`タイプがあります。

以下は、デプロイしたマイクロサービスの`ContainerAppConsoleLogs_CL`タイプの最新の50件のログを取得する方法です。以下のクエリをクエリエディタに貼り付けて「実行」をクリックします。

```sql
ContainerAppConsoleLogs_CL
| where ContainerAppName_s == "helloworld"
| project time_t, Log_s
| order by time_t desc
| limit 50
```

![Query logs](images/loganalytics.png)

> 💡 Azure Container Appsマイクロサービスのコンソール出力がLog Analyticsに取り込まれるまでに約1〜2分かかることを覚えておいてください。

## Azure Container Appsのスケーリング

Azure Container Appsでは、要件に応じてコンテナをスケーリングできます。デフォルトでは、ACAは0から10のレプリカにスケーリングするように設定されており、デフォルトのスケーリングルールはHTTPスケーリングを使用します。Azureポータルに移動して、スケールルールの設定を確認しましょう。

![Hello World](images/helloworld-scale.png)

上の画像に示されているように、デフォルトのスケーリングルールは0から10のレプリカにスケーリングするように設定されています。また、現在のレプリカ数が1であることがわかります。自動スケーリングを望まず、レプリカ数を固定の数に設定したい場合は、以下のコマンドを使用してインスタンス数を更新できます。

```shell
az containerapp update --name helloworld --min-replicas 1 --max-replicas 1
```

## リビジョンを作成する

ゼロダウンタイムデプロイメントは、アプリケーションにとって重要な機能です。たとえば、Kubernetesでは、新しいデプロイメントを作成し、サービスを更新して新しいデプロイメントを指すことでこれを実現します。IstioやService Meshがある場合、重み付けルーティングを使用してカナリアデプロイメントを行うこともできます。

Azure Container Appsは、この機能をサポートしており、新しいリビジョンを作成し、その後新しいリビジョンに更新します。デフォルトでは、Azure Container Appsは単一リビジョンモードに設定されています。このモードでは、新しいリビジョンが準備されるまで、既存のアクティブなリビジョンは非アクティブ化されません。イングレスが有効になっている場合、現在のリビジョンは新しいリビジョンが準備されるまで100％のトラフィックを受け取り続けます。

ここでは、単一リビジョンモードがどのように機能するかを確認するための小さなテストを行います。**レディネスプローブを使用して、新しいリビジョンが失敗するように意図的に設定し、古いリビジョンがトラフィックを継続して処理する様子を確認します。**

<details markdown="block">
**<summary>GitHub Copilotを使用する</summary>**

前の`HelloController.java`に移動し、`/hello`エンドポイントのメッセージを変更して、古いリビジョンと新しいリビジョンの違いを確認します。その後、HTTPコード500を返すエンドポイント`/readiness`を追加します。GitHub CopilotにHTTPコード500を返すようにプロンプトを設定します。

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Failure from Azure Container Apps\n";
    }

    // Return HTTP code 500 as return with ResponseEntity for Endpoint /readiness
}
```

> GitHub Copilotが正しいコードを提供しない場合があることを覚えておいてください。
> プロンプトがどのように機能するかを理解し、Copilotを正しい方向に導くことが重要です。
> たとえば、`public ResponseEntity`と入力を開始して、GitHub Copilotを正しい方向に促すことができます。

</details>

<details markdown="block">
**<summary>自分でコードを書く</summary>**

```java
package com.example.demo;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Azure Container Apps\n";
    }

    // Return HTTP code 500 as return with ResponseEntity for Endpoint /readiness
    @GetMapping("/readiness")
    public ResponseEntity<String> readiness() {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Not ready yet\n");
    }
}

```

</details>

次に、`helloworld`アプリを更新し、リビネスプローブを`/readiness`エンドポイントに設定します。これにより、新しいリビジョンが意図的に失敗します。以下のコマンドを使用してアプリを更新します

```shell
az containerapp up --name helloworld --environment ${ACA_ENVIRONMENT_NAME} --source . --ingress external --target-port 8080 --query properties.configuration.ingress.fqdn
```

Azureポータルでリビネスプローブの設定を変更します。[Azureポータル](https://portal.azure.com)に移動し、`helloworld` Azure Container Appsに移動します。「アプリケーション」をクリックし、「コンテナ」をクリックします。上部の「編集してデプロイ」をクリックし、コンテナイメージをクリックします。

![Readiness Probe](images/readiness.png)

下部の「作成」ボタンをクリックすると、新しいリビジョンが作成されます。次に、「アプリケーション」>「リビジョンとレプリカ」に移動します。そこには2つのリビジョンがあり、そのうちの1つは「アクティブ化中」と表示されます。

![Revisions](images/revision-1.png)

新しいリビジョンは、意図的にリビネスプローブが失敗するように設定されているため、アクティブ化されることはありません。システムログを確認しましょう。「詳細を表示」をクリックし、「システムログストリームを表示」をクリックします。

![Revisions](images/revision-2.png)

システムログには、新しく作成されたリビジョンに対して実行されるプローブが失敗していることが表示されます。

![Revisions](images/revision-3.png)

次のステップに進む前に、リビネスプローブの設定を削除して行った変更を元に戻す必要があります。失敗しているリビジョンを見つけ、「編集してデプロイ」をクリックし、リビネスプローブを削除します。

## :notebook_with_decorative_cover: まとめ

おめでとうございます。最初のSpring BootアプリをAzure Container Appsにデプロイしました！次に、KEDAを使用してAzure Container Appsを自動スケーリングする方法を学びます。

---

➡️
次へ : [03 - KEDA(Kubernetes-based Event Driven Autoscaler)を使用した自動スケーリング](../03-use-keda-autoscaling/README.md)
