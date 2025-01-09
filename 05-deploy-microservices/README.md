# :rocket: PetClinicマイクロサービスをデプロイする

![PetClinic Application](images/frontend-1.png)

このラボでは、Spring PetClinicアプリケーションのバックエンドマイクロサービスである`customers-service`、`visits-service`、および`vets-service`をデプロイします。これらのサービスはすべてSpring Bootを使用して構築されています。フロントエンドマイクロサービスはAngularとSpring Cloud Gatewayを使用して構築された`api-gateway`です。

![PetClinic Spring Apps](images/petclinic.png)

デフォルトでは、Spring PetClinicアプリケーションはインメモリデータベースを使用しますが、これを変更してAzure Database for MySQL Flexible Serverを使用するようにします（第6章で説明します）。各サービスはEureka Serverにバインドされるため、UI+API GatewayはEureka Serverを通じてサービスを発見できます。また、すべてのマイクロサービスをConfig Serverにバインドして、Config Serverから構成を取得します。

## 目的

このモジュールでは、以下の2つの主要な目的に焦点を当てます：

1. :white_check_mark: Spring PetClinicバックエンドサービスをAzure Container Appsにデプロイする。
2. :bar_chart: すべてのサービスをEureka ServerおよびConfig Serverにバインドする。

---

## バックエンドサービスのデプロイ

最初のステップとして、Azure Container Appsに3つのマイクロサービス`customers-service`、`visits-service`、および`vets-service`をデプロイします。今回は、各サービスをアーティファクトとともにデプロイし、Eureka ServerおよびConfig Serverにバインドします。Javaコンポーネントをデプロイ中またはデプロイ後にバインド/アンバインドできます。ここでは、例としてデプロイ後にバインドします。

### `customers-service`のデプロイ

まず、Mavenを使用してプロジェクトをビルドし、`az containerapp create`コマンドでデプロイします。このアプリでは、デプロイとバインドのステップを分けて、デプロイ後にJavaコンポーネントをContainer Appsにバインドする方法を示します。

```bash
cd ~/spring-petclinic-customers-service
mvn clean package
az containerapp create \
  --name customers-service \
  --environment ${ACA_ENVIRONMENT_NAME} \
  --artifact target/customers-service-3.2.11.jar \
  --ingress external \
  --target-port 8080 \
  --query properties.configuration.ingress.fqdn \
  --min-replicas 1
```

Eureka Server、Config Server、およびSpring Boot Adminをcustomers-serviceにバインドします。

```bash
az containerapp update \
    --name customers-service \
    --bind eurekaserver configserver admin
```

### `visits-service`のデプロイ

次に、`visits-service`をアーティファクトとともにデプロイし、すべてのマネージドJavaコンポーネントにバインドします。

```bash
cd ~/spring-petclinic-visits-service
mvn clean package
az containerapp create \
  --name visits-service \
  --environment ${ACA_ENVIRONMENT_NAME} \
  --artifact target/visits-service-3.2.11.jar \
  --ingress external \
  --target-port 8080 \
  --query properties.configuration.ingress.fqdn \
  --min-replicas 1 \
  --bind eurekaserver configserver admin
```

### `vets-service`のデプロイ

上記と同じ手順を繰り返して、`vets-service`をデプロイします。

```bash
cd ~/spring-petclinic-vets-service
mvn clean package
az containerapp create \
  --name vets-service \
  --environment ${ACA_ENVIRONMENT_NAME} \
  --artifact target/vets-service-3.2.11.jar \
  --ingress external \
  --target-port 8080 \
  --query properties.configuration.ingress.fqdn \
  --min-replicas 1 \
  --bind eurekaserver configserver admin
```

## フロントエンドサービスのデプロイ

Mavenを使用してプロジェクトをビルドし、`az containerapp create`コマンドでデプロイします。

```bash
cd spring-petclinic-api-gateway
mvn clean package
az containerapp create \
  --name frontend-service \
  --environment ${ACA_ENVIRONMENT_NAME} \
  --artifact target/api-gateway-3.2.11.jar \
  --ingress external \
  --target-port 8080 \
  --query properties.configuration.ingress.fqdn \
  --min-replicas 1 \
  --bind eurekaserver configserver admin
```

## Javaコンポーネントバインディングの探索

JavaコンポーネントをEureka ServerおよびConfig Serverにバインドしたとき、JavaコンポーネントをContainer Appsに接続しました。これにより、さまざまな環境変数がContainer Appsに注入されます。

たとえば、Eureka Serverバインディングは以下の環境変数を設定します：

![Eureka Env Variables](images/eureka-1.png)

同様に、Config Serverバインディングは以下の環境変数を設定します：

![Config Server Env Variables](images/config-1.png)

## PetClinicアプリケーションのテスト

バックエンドサービスとフロントエンドサービスをデプロイしたので、アプリケーションをテストできます。ブラウザを開き、フロントエンドサービスのURLにアクセスします。

1. `Owners`をクリックして、オーナーのリストを表示します。
2. `Veterinarians`をクリックして、獣医のリストを表示します。
3. `Owners`に戻り、新しいオーナーを登録します。
4. 新しいオーナーに移動し、新しいペットを追加します。

![Test PetClinic](images/frontend-2.png)

## :notebook_with_decorative_cover: まとめ

この章では、Spring PetClinicバックエンドサービスをAzure Container Appsにデプロイしました。また、フロントエンドサービスもデプロイしました。バックエンドサービスをEureka ServerおよびConfig Serverにバインドしました。JavaコンポーネントのContainer Appsへのバインディングを探索しました。次に、Azure Service Connectorを使用して、バックエンドサービスをAzure Database for MySQL Flexible Serverに接続します。

---

:arrow_forward::️ 次へ : [06 - Azure Service Connectorを使用する](../06-use-service-connector/README.md)
