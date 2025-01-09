# Azure OpenAIとSpring AIを使用してAIを組み込んだJavaアプリを作成する

この章では、Azure OpenAIとSpring AIを使用してAIを組み込んだJavaアプリケーションを作成する方法を探ります。まず、Spring BootアプリケーションをAzure OpenAIに接続するように構成します。次に、Azure OpenAIのGPT-4oモデルを使用してシンプルなチャットボットを実装し、AIをJavaアプリケーションにシームレスに統合する方法を示します。さらに、Spring AIを使用した関数呼び出しをカバーし、AIモデルを活用して複雑なタスクを実行するアプリケーションを作成します。

---

このモジュールでは、以下の2つの主要な目的に焦点を当てます：
1. :white_check_mark: Azure OpenAIをチャットボットに統合する。
2. :bar_chart: 関数呼び出しを使用してSpring AIで基本的なRAGを実装する。

## チャットボットとのAzure OpenAI統合

まず、Azure OpenAIをSpring Bootアプリケーションに統合してチャットボットを作成します。最初のラボで`azd up`を実行した際に作成されたBicepは、`gpt-4o`という名前のAzure OpenAIデプロイメントを作成します。このデプロイメントを使用して、PetClinicアプリケーションにチャット体験を提供します。

`spring-petclinic-chats-service`に移動し、`application.yml`ファイルを開きます。${AZURE_OPENAI_API_KEY}と${AZURE_OPENAI_ENDPOINT}をAzure OpenAI APIキーとエンドポイントに置き換えます。

```yaml
spring:
  application:
    name: chats-service
  config:
    import: optional:configserver:${CONFIG_SERVER_URL:http://localhost:8888/}
  ai:
    azure:
      openai:
        api-key: ${AZURE_OPENAI_API_KEY}
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            model: gpt-4o
            deployment-name: gpt-4o
            temperature: 0.5

```

### GitHub Copilotを使用したプロンプトエンジニアリング

GitHub Copilotの背後にあるのは、特別に訓練されたLLMモデルであり、コンテキストに基づいてコードスニペットを生成できます。Spring AIは非常に新しいプロジェクトであり、Copilotの最新の知識カットオフ日は2023年10月であるため、Spring AIについての知識がない可能性があります。この制限を克服するために、プロンプトエンジニアリングを使用してCopilotにコンテキストを提供できます。そのために、Spring AIとAzure Open AIの基本的な使用法を説明する`CONTEXT.md`というマークダウンファイルがあります。このファイルをプロンプトの一部として提供するだけです。

VSCodeの場合、ファイルを手動で追加できます：
![VS Code](images/vscode-1.png)

JetBrains IntelliJ IDEAの場合、まずタブとしてファイルを開いてから追加する必要があります：
![IntelliJ IDEA](images/jetbrains-1.png)

次に、GitHub CopilotにAzure OpenAIエンドポイントと統合するようにプロンプトを設定する必要があります。以下は使用できるサンプルプロンプトです：

```text
1. '/chat'でチャット完了を提供するPOSTエンドポイントを作成する
2. AzureOpenAiChatModelを使用してAzure OpenAIエンドポイントでチャット完了を行う
3. ユーザープロンプトとシステムプロンプトを使用する
```

> :information_source: プロンプトを設定する前に、適切なファイル（`CONTEXT.md`と`ChatController.java`）を追加してください。

![GitHub Copilot Prompt](images/prompt-1.png)

システムプロンプトを少し変更して、アプリケーションであるPetClinicに特化させます。以下は更新されたプロンプトです：

```text
あなたはSpring Petclinicという獣医ペットクリニックの管理を支援するために設計されたフレンドリーなAIアシスタントです。
あなたの仕事は、ユーザーのリクエストに応じて質問に答えたり、ユーザーの代わりにアクションを実行したりすることです。主に獣医、オーナー、オーナーのペット、オーナーの訪問に関するものです。
プロフェッショナルな態度で回答する必要があります。答えがわからない場合は、丁寧にユーザーに答えがわからないことを伝え、次にユーザーにフォローアップの質問をして、彼らが尋ねている質問を明確にしようとします。
答えがわかる場合は、答えを提供しますが、追加のフォローアップの質問は提供しません。
獣医に関しては、ユーザーが返された結果に不確かである場合、返されなかった追加のデータがある可能性があることを説明します。
すべての獣医の総数についてユーザーが尋ねている場合のみ、たくさんいると答え、追加の基準を求めます。
オーナー、ペット、または訪問に関しては、正しいデータを提供します。
```

完全なソースコードは以下のようになります：

<details markdown="block">

### 完成したソースコード

```java
package org.springframework.samples.petclinic.chats;

import org.springframework.ai.azure.openai.AzureOpenAiChatModel;
import org.springframework.ai.chat.messages.Message;
import org.springframework.ai.chat.messages.UserMessage;
import org.springframework.ai.chat.model.ChatResponse;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.chat.prompt.SystemPromptTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class ChatController {

    private final AzureOpenAiChatModel chatClient;

    @Autowired
    public ChatController(AzureOpenAiChatModel chatClient) {
        this.chatClient = chatClient;
    }

    @PostMapping("/chatclient")
    public String chat(@RequestBody String userPrompt) {
        Message systemMessage = new SystemPromptTemplate("""
         あなたはSpring Petclinicという獣医ペットクリニックの管理を支援するために設計されたフレンドリーなAIアシスタントです。
         あなたの仕事は、ユーザーのリクエストに応じて質問に答えたり、ユーザーの代わりにアクションを実行したりすることです。主に獣医、オーナー、オーナーのペット、オーナーの訪問に関するものです。
         プロフェッショナルな態度で回答する必要があります。答えがわからない場合は、丁寧にユーザーに答えがわからないことを伝え、次にユーザーにフォローアップの質問をして、彼らが尋ねている質問を明確にしようとします。
         答えがわかる場合は、答えを提供しますが、追加のフォローアップの質問は提供しません。
         獣医に関しては、ユーザーが返された結果に不確かである場合、返されなかった追加のデータがある可能性があることを説明します。
         すべての獣医の総数についてユーザーが尋ねている場合のみ、たくさんいると答え、追加の基準を求めます。
         オーナー、ペット、または訪問に関しては、正しいデータを提供します。
        """).createMessage();

        UserMessage userMessage = new UserMessage(userPrompt);

        Prompt prompt = new Prompt(List.of(systemMessage, userMessage));
        ChatResponse response = chatClient.call(prompt);

        return response.getResult().getOutput().getContent();
    }
}
```
</details>

### チャットボットのテスト

```bash
cd ~/spring-petclinic-chats-service
mvn clean package
az containerapp up \
  --name chats-service \
  --environment ${ACA_ENVIRONMENT_NAME} \
  --artifact target/chats-service-3.2.11.jar \
  --ingress external \
  --target-port 8080 \
  --query properties.configuration.ingress.fqdn
```

chats-serviceが実行されている場合、アプリケーションのチャットボックスからチャットボットをテストできるはずです。

![PetClinic Chat](images/chat-1.png)

## SpringAIを使用した関数呼び出し

このセクションでは、Spring AIを使用して基本的なRAG（Retrieval-Augmented Generation）パターンを実装します。Retrieval-Augmented Generation（RAG）パターンは、特定のデータやプロプライエタリデータを使用して大規模言語モデルを活用するための業界標準のアプローチです。これは、前のステップで統合したAzure Open AIモデルがPetClinicアプリケーションについて何も知らないため、重要です。

RAGを実装する方法は複数ありますが、最も簡単な方法を使用します。Spring AIの`FunctionCalling`クラスを使用して、Azure OpenAIモデルを呼び出します。`FunctionCalling`クラスは、指定された入力と出力を持つ関数を呼び出すためのユーティリティクラスです。このクラスを使用して、指定された入力と出力を持つAzure OpenAIモデルを呼び出します。

### オーナーリスト関数の作成

PetClinicアプリケーション内のすべてのオーナーをリストする関数を実装します。この関数は、Azure OpenAIモデルによって呼び出され、オーナーのリストを取得します。

完全なソースコードはこちら：

```java
package org.springframework.samples.petclinic.chats;

import org.springframework.ai.azure.openai.AzureOpenAiChatModel;
import org.springframework.ai.azure.openai.AzureOpenAiChatOptions;
import org.springframework.ai.chat.messages.Message;
import org.springframework.ai.chat.messages.UserMessage;
import org.springframework.ai.chat.model.ChatResponse;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.chat.prompt.SystemPromptTemplate;
import org.springframework.ai.model.function.FunctionCallback;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.Arrays;
import java.util.List;
import java.util.function.Function;

@RestController
public class ChatController {

    private final AzureOpenAiChatModel chatClient;

    
    @Autowired
    public ChatController(AzureOpenAiChatModel chatClient) {
        this.chatClient = chatClient;
    }

    @PostMapping("/chatclient")
    public String chat(@RequestBody String userPrompt) {
        Message systemMessage = new SystemPromptTemplate("""
                                                                  あなたはSpring Petclinicという獣医ペットクリニックの管理を支援するために設計されたフレンドリーなAIアシスタントです。
                                                                  あなたの仕事は、ユーザーのリクエストに応じて質問に答えたり、ユーザーの代わりにアクションを実行したりすることです。主に獣医、オーナー、オーナーのペット、オーナーの訪問に関するものです。
                                                                  プロフェッショナルな態度で回答する必要があります。答えがわからない場合は、丁寧にユーザーに答えがわからないことを伝え、次にユーザーにフォローアップの質問をして、彼らが尋ねている質問を明確にしようとします。
                                                                  答えがわかる場合は、答えを提供しますが、追加のフォローアップの質問は提供しません。
                                                                  獣医に関しては、ユーザーが返された結果に不確かである場合、返されなかった追加のデータがある可能性があることを説明します。
                                                                  すべての獣医の総数についてユーザーが尋ねている場合のみ、たくさんいると答え、追加の基準を求めます。
                                                                  オーナー、ペット、または訪問に関しては、正しいデータを提供します。
                                                                 """).createMessage();

        UserMessage userMessage = new UserMessage(userPrompt);

        Prompt prompt = new Prompt(List.of(systemMessage, userMessage), AzureOpenAiChatOptions.builder()
                                                                                              .withFunction("OwnerService")
                                                                                              .build());
        ChatResponse response = chatClient.call(prompt);

        return response.getResult().getOutput().getContent();
    }

    @Configuration
    static class Config {

        @Bean
        public FunctionCallback ownersFunctionInfo() {

            return FunctionCallback.builder()
                                   .description("List of all the owners")
                                   .function("OwnerService", new OwnerService())
                                   .inputType(Void.class)
                                   .build();
        }

    }
}

class OwnerService implements Function<Void, List<OwnerDetails>> {

    @Override
    public List<OwnerDetails> apply(Void unused) {
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<List<OwnerDetails>> responseEntity = restTemplate.exchange(
                "http://frontend-service/api/customer/owners",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<OwnerDetails>>() {
                }
        );

        List<OwnerDetails> owners = responseEntity.getBody();
        return owners;
    }
}
```

オーナーに関するさまざまな質問を試してみてください。

![PetClinic Chat](images/chat-2.png)
![PetClinic Chat](images/chat-3.png)
---


## :notebook_with_decorative_cover: まとめ

このモジュールでは、Azure OpenAIをSpring Bootアプリケーションに統合してチャットボットを作成しました。また、Spring AIを使用して基本的なRAGパターンを実装し、PetClinicアプリケーション内のオーナーのリストを取得しました。Azure OpenAIとSpring AIを活用することで、ユーザーとインテリジェントに対話し、複雑なタスクを実行できるAIを組み込んだJavaアプリケーションを作成する方法を示しました。

