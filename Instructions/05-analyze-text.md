---
lab:
  title: テキストの分析
  module: Module 3 - Getting Started with Natural Language Processing
---

# テキストの分析

**Azure Language** は、言語検出、感情分析、キー フレーズ抽出、エンティティ認識など、テキストの分析をサポートします。

たとえば、旅行代理店が会社の Web サイトに送信されたホテルのレビューを処理したいとします。 Azure AI Language を使用すると、各レビューが書かれている言語、レビューの感情 (ポジティブ、ニュートラル、ネガティブ)、レビューで議論されている主なトピックを示す可能性のあるキー フレーズ、名前付きエンティティ (レビューで言及された場所、ランドマーク、人など) を特定できます。

## Azure AI Language リソースをプロビジョニングする

サブスクリプションに **Azure AI Language サービス** リソースがまだない場合は、サポートされているリージョンでプロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1. 上部にある検索フィールドで「**Azure AI サービス**」を検索します。 次に結果で、[**言語サービス**] の下の [**作成**] を選択します。
1. **[リソースの作成を続行する]** を選択します。
1. 次の設定を使用してリソースをプロビジョニングします。
    - **[サブスクリプション]**: *お使いの Azure サブスクリプション*。
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。その場合は、提供されているものを使用してください)*。
    - **リージョン**: *使用できるリージョンを選択する*
    - **[名前]**: *一意の名前を入力します*。
    - **価格レベル**: **Free (F0)** または **Standard (S)* *レベル (Free が利用できない場合) のいずれかを選択します。
    - **責任ある AI 通知**: 同意。
1. **[Review + create](レビュー + 作成)** を選択します。
1. デプロイが完了するまで待ち、デプロイの詳細を表示します。

## Cloud Shell でこのコースのリポジトリを複製する

Cloud Shell を操作するには、新しいブラウザー タブを開きます。 このリポジトリを最近 Cloud Shell に複製していない場合は、次の手順に従って最新のバージョンがあることを確認します。 それ以外の場合は、Cloud Shell を開き、複製に移動します。

1. [Azure portal](https://portal.azure.com?azure-portal=true) で、ページ上部の検索ボックスの右側にある **[>_]** (*Cloud Shell*) ボタンを選びます。 ポータルの下部に Cloud Shell ペインが開きます。

    ![上部の検索ボックスの右側にあるアイコンをクリックして Cloud Shell を開始している状態のスクリーンショット。](images/cloudshell-launch-portal.png#lightbox)

2. Cloud Shell を初めて開くと、使用するシェルの種類 (*Bash* または *PowerShell*) を選択するように求められる場合があります。 **[Bash]** を選択します。 このオプションが表示されない場合は、この手順をスキップします。  

3. Cloud Shell のストレージを作成するように求めるメッセージが表示された場合は、お使いのサブスクリプションが指定されていることを確認して、**[ストレージの作成]** を選択します。 その後、ストレージが作成されるのを 1 分程度待ちます。

4. Cloud Shell ペインの左上に表示されるシェルの種類が *Bash* に切り替えられたことを確認します。 *PowerShell* の場合は、ドロップダウン メニューを使用して *Bash* に切り替えます。

5. ターミナルが起動したら、次のコマンドを入力してサンプル アプリケーションをダウンロードし、`azure-ai-eng` という名前のフォルダーに保存します。

    ```bash
   rm -r azure-ai-eng -f
   git clone https://github.com/MicrosoftLearning/AI-102-AIEngineer azure-ai-eng
    ```
  
6. ファイルは、「**azure-ai-eng**」という名前のフォルダーにダウンロードされます。 次のコマンドを使用して、この演習のラボ ファイルに移動します。

    ```bash
   cd azure-ai-eng/05-analyze-text
    ```

C# と Python の両方のアプリケーションと、その機能をテストするためのサポートファイルが提供されています。 どちらのアプリにも同じ機能があります。 優先する言語のフォルダーに移動します。

組み込みのコード エディターを開き、`text-analysis` フォルダー内のテキスト ファイルを確認します。 次のコマンドを使用して、コード エディターでラボ ファイルを開きます。

```bash
code .
```

## テキスト分析に Azure AI Language SDK を使用するための準備

この演習では、Azure AI Language テキスト分析 SDK を使用してホテルのレビューを分析する、部分的に実装されたクライアント アプリケーションを完成させます。

> [!NOTE]
> > **C#** または **Python** 用の SDK のいずれかに使用するよう選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Cloud Shell で、ご自分が **05-analyze-text** フォルダー内にいて、言語の設定に応じて **C-Sharp** または **Python** フォルダーに移動していることを確認します。
1. 言語設定に適合するコマンドを実行して、Text Analytics SDK パッケージをインストールします。

    **C#**

    ```bash
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**

    ```bash
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

1. コード ウィンドウの **text-analysis** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、そこに含まれる構成値を更新して、Azure AI Language サービス リソース用の**エンドポイント**と認証 **キー**を反映します。 変更を保存します。

1. **text-analysis** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: text-analysis.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Text Analytics SDK を使用するために必要な名前空間インポートします。

    **C#**

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. **Main** 関数で、構成ファイルから Azure AI Language サービスのエンドポイントとキーを読み込むためのコードが既に提供されていることに注意してください。 次に、**エンドポイントとキーを使用してクライアントを作成する**というコメントを見つけ、次のコードを追加して、Text Analysis API のクライアントを作成します。

    **C#**

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(cogSvcKey);
    Uri endpoint = new Uri(cogSvcEndpoint);
    TextAnalyticsClient CogClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(cog_key)
    cog_client = TextAnalyticsClient(endpoint=cog_endpoint, credential=credential)
    ```

1. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python text-analysis.py
    ```

1. コードがエラーなしで実行され、**reviews** フォルダー内の各レビュー テキストファイルの内容が表示されるので、出力を確認します。 アプリケーションは Text Analytics API のクライアントを正常に作成しますが、それを利用しません。 次の手順で修正します。

## 言語を検出する

API のクライアントを作成したので、それを使用して、各レビューが書かれている言語を検出します。

1. プログラムの **Main** 関数で、コメント **Get language** を見つけます。 次に、このコメントの下に、各レビュー ドキュメントで言語を検出するために必要なコードを追加します。

    **C#**

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = CogClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**

    ```python
    # Get language
    detectedLanguage = cog_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

    > **注**: *この例では、各レビューが個別に分析され、ファイルごとにサービスが個別に呼び出されます。別の方法として、ドキュメントのコレクションを作成し、1 回の呼び出しでサービスに渡す方法があります。どちらの方法でも、サービスからの応答はドキュメントのコレクションで構成されます。これは、上記の Python コードでは、応答 ([0]) 内の最初の (そして唯一の) ドキュメントのインデックスが指定されている理由です。*

1. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python text-analysis.py
    ```

1. 出力を観察します。今回は、各レビューの言語が識別されていることに注意してください。

## 感情の評価

"*感情分析*" は、テキストを "*ポジティブ*" または "*ネガティブ*" ("*ニュートラル*" または "*混合*" の場合もある) として分類するために一般的に使用される手法です。 ソーシャル メディアの投稿、製品レビュー、およびテキストの感情が有用な洞察を提供する可能性があるその他のアイテムを分析するために一般的に使用されます。

1. プログラムの **Main** 関数で、コメント **Get sentiment** を見つけます。 次に、このコメントの下に、各レビュードキュメントの感情を検出するために必要なコードを追加します。

    **C#**

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = CogClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**

    ```python
    # Get sentiment
    sentimentAnalysis = cog_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python text-analysis.py
    ```

1. 出力を観察し、レビューの感情が検出されたことに注意します。

## キー フレーズの特定

テキストの本文でキーフレーズを特定すると、説明する主なトピックを特定するのに役立ちます。

1. プログラムの **Main** 関数で、コメント **Get key phrases** を見つけます。 次に、このコメントの下に、各レビュードキュメントのキー フレーズを検出するために必要なコードを追加します。

    **C#**

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = CogClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**

    ```python
    # Get key phrases
    phrases = cog_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python text-analysis.py
    ```

1. 各ドキュメントにキー フレーズが含まれていることに注意して、出力を確認します。それはレビューが何であるかについてのいくつかの洞察を与えます。

## エンティティの抽出

多くの場合、ドキュメントまたはその他のテキスト本体は、人、場所、期間、またはその他のエンティティについて言及しています。 Text Analytics API は、テキスト内のエンティティの複数のカテゴリ (およびサブカテゴリ) を検出できます。

1. プログラムの **Main**関数で、コメント **Get entities** を見つけます。 次に、このコメントの下に、各レビューで言及されているエンティティを識別するために必要なコードを追加します。

    **C#**

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = CogClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**

    ```python
    # Get entities
    entities = cog_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python text-analysis.py
    ```

1. 出力を確認します。テキストで検出されたエンティティに注意してください。

## リンクされたエンティティを抽出する

分類されたエンティティに加えて、Text Analytics API は、Wikipedia などのデータソースへの既知のリンクがあるエンティティを検出できます。

1. プログラムの **Main** 関数で、コメント **Get linked entities** を見つけます。 次に、このコメントの下に、各レビューで言及されているリンクされたエンティティを識別するために必要なコードを追加します。

    **C#**

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = CogClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**

    ```python
    # Get linked entities
    entities = cog_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python text-analysis.py
    ```

1. 出力を確認します。識別されたリンクされたエンティティに注意してください。

## 詳細情報

**Azure AI Language** の詳細については、「[ドキュメント](https://learn.microsoft.com/en-us/azure/ai-services/language-service/overview)」を参照してください。
