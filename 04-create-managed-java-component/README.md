# :rocket: Azure Container AppsでマネージドJavaコンポーネントを作成する

Azure Container Appsは、Spring Bootを本番環境で実行するためのいくつかのマネージドコンポーネントを提供しています。これには、Eureka Server、Config Server、およびSpring Boot Adminが含まれます。

## 目的

このモジュールでは、以下の3つの主要な目的に焦点を当てます：

1. :white_check_mark: Spring Bootアプリケーション用のさまざまなマネージドコンポーネントを作成する。
2. :bar_chart: Eureka Serverダッシュボードにアクセスする。
3. :mag: Spring Boot Adminダッシュボードにアクセスする。

---

## マネージドEureka Serverを作成する

クラウドネイティブアプリケーションの基本的な要件の1つは、*サービスディスカバリー*です。これは、さまざまなマイクロサービスを見つけて識別する機能です。このセクションでは、この機能を有効にするために[Spring Cloud Eureka Server](https://spring.io/projects/spring-cloud-netflix)を作成します。

1. Eureka Serverの名前を定義するために環境変数を設定します。

```bash
EUREKA_SERVER_NAME="eurekaserver"
```

2. マネージドEureka Serverを作成します。

```bash
az containerapp env java-component eureka-server-for-spring create \
  --environment ${ACA_ENVIRONMENT_NAME} \
  --name ${EUREKA_SERVER_NAME} \
  --query properties.ingress.fqdn -o tsv
```

3. 上記のコマンドで返されたFQDNを使用してEurekaダッシュボードにアクセスします。
![Spring Boot Admin](images/eureka-web.png)

## マネージドConfig Serverを作成する

クラウドネイティブアプリケーションのもう1つの基本的な要件は、*外部化された構成*です。これは、アプリケーションコードとは別に構成を保存、管理、およびバージョン管理する機能です。このセクションでは、この機能を有効にするために[Spring Cloud Config Server](https://spring.io/projects/spring-cloud-config)を作成および構成します。次のセクションでは、Spring Cloud ConfigがGitリポジトリから構成をアプリケーションに注入する方法を示します。

このショートカットを使用するには：

1. Config Serverの名前と構成ソースとして使用するGitリポジトリのURLを定義するために、次の環境変数を設定します。
   ```bash
   CONFIG_SERVER_NAME="configserver"
   GIT_URL="https://github.com/eggboy/spring-petclinic-microservices-config"
   ```
2. Spring用のマネージドConfig Serverを作成し、その構成ソースをパブリックGitリポジトリとして設定します。
   ```bash
   az containerapp env java-component config-server-for-spring create \
   --environment ${ACA_ENVIRONMENT_NAME} \
   --name ${CONFIG_SERVER_NAME} \
   --configuration spring.cloud.config.server.git.uri=$GIT_URL spring.cloud.config.server.git.refresh-rate=60
   ```

## マネージドSpring Boot Adminを作成する

Spring用のAdminマネージドコンポーネントは、アクチュエータエンドポイントを公開するSpring Boot Webアプリケーションの管理インターフェースを提供します。Azure Container Appsのマネージドコンポーネントとして、コンテナアプリをSpring用のAdminに簡単にバインドして、シームレスな統合と管理を実現できます。

1. Spring Boot Adminの名前を定義するために、次の環境変数を設定します。
   ```bash
   SPRING_ADMIN_NAME="admin"
   ```
2. マネージドSpring Boot Adminを作成します。

```bash
az containerapp env java-component admin-for-spring create \
--environment ${ACA_ENVIRONMENT_NAME}  \
--name ${SPRING_ADMIN_NAME} \
  --min-replicas 1 \
  --max-replicas 1 \
--query properties.ingress.fqdn -o tsv
```
3. 上記のコマンドから返されたFQDNを使用してSpring Boot Adminコンソールにアクセスします。Spring Boot AdminはデフォルトでAzure Entra IDによって保護されています。

![Spring Boot Admin](images/admin-fqdn.png)
![Spring Boot Admin](images/admin-web.png)

## :notebook_with_decorative_cover: まとめ

このモジュールでは、マネージドEureka Server、Config Server、およびSpring Boot Adminを作成しました。さらに、Eureka ServerダッシュボードとSpring Boot Adminダッシュボードを探索しました。次に、PetClinicマイクロサービスをデプロイします。

---

:arrow_forward::️ 次へ : [05 - PetClinicマイクロサービスをデプロイする](../05-deploy-microservices/README.md)
