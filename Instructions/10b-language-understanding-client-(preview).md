---
lab:
  title: Conversational Language Understanding クライアント アプリケーションを作成する
  module: Module 5 - Creating Language Understanding Solutions
---

# 言語サービス クライアント アプリケーションを作成する

Azure AI Service for Language の会話言語理解機能を使用すると、クライアント アプリがユーザーからの自然言語入力を解釈し、ユーザーの*意図* (達成したいこと) を予測し、その意図を適用すべき*エンティティ*を識別できる会話言語モデルを定義できます。 会話言語理解モデルを使用するクライアント アプリケーションは、REST インターフェイスを介して直接作成するか、言語固有のソフトウェア開発キット (SDK) を使用して作成できます。

## このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。

2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。

3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。

4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## 言語サービス リソースを作成する

言語サービス リソースを Azure サブスクリプションで既に作成した場合は、この演習で使用できます。 それ以外の場合は、次の手順に従って作成してください。

1. Azure portal (`https://portal.azure.com/?l=ja.ja-jp`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。

2. ポータル上部の検索窓に、*language* と入力し、次の設定を使用して**言語サービス** リソースを作成します。

    - **既定の機能**:すべて
    - **カスタム機能**: なし
    - **[サブスクリプション]**: *ご自身の Azure サブスクリプション*
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)* 
    - **リージョン**: *westus2*
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: *S*
    - **責任ある AI 通知**: *確認するチェック ボックスをオンにする*

3. リソースが作成されるまで待ちます。 リソースを作成したリソース グループに移動すると、リソースを表示できます。

## 会話言語理解モデルをインポート、トレーニング、発行する

前のラボまたは演習の **Clock** プロジェクトを既にお持ちの場合は、この演習で使用できます。 それ以外の場合は、次の手順に従って作成してください。

1. ブラウザーの新しいタブで、`https://language.cognitive.azure.com` の Language Studio - プレビュー ポータルを開きます。

2. Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。 言語サービス ポータルに初めてサインインする場合は、アカウントの詳細にアクセスするためのアクセス許可をアプリに付与する必要がある場合があります。 次に、Azure サブスクリプションと作成したオーサリング リソースを選択して、*ようこそ* の手順を完了します。

3. **[Conversational Language Understanding]** ページを開きます。

4. **&#65291;[新しいプロジェクトの作成]** の横の **[インポート]** を選択します。 **[ファイルの選択]** をクリックし、この演習のラボ ファイルを含むプロジェクト フォルダーの **10b-clu-client-(preview)** サブフォルダーを参照します。 **Clock.json** を選択し、 **[開く]** をクリックして、 **[完了]** をクリックします。

5. 効果的な言語サービス アプリを作成するためのヒントが記載されたパネルが表示されたら、それを閉じます。

6. Language Studio ポータルの左側にある **[トレーニング ジョブ]** を選択して、アプリをトレーニングします。 [**トレーニング ジョブの開始**] をクリックし、モデルに「**Clock**」という名前を付け、既定のトレーニング モード (標準) とデータ分割を維持します。 **[Train](トレーニング)** を選択します。 トレーニングが完了するまでに数分かかる場合があります。

    > **注**:モデル名 **Clock** はクロック クライアント コード (ラボの後半で使用) でハードコーディングされているため、説明に従って名前を大文字にしてスペルを指定します。 

7. Language Studio ポータルの左側にある **[モデルのデプロイ]** を選択し、 **[デプロイの追加]** を使用して、**production** という名前の Clock モデルのデプロイを作成します。

    > **注**:デプロイ名 **production** はクロック クライアント コード (ラボの後半で使用) でハードコーディングされているため、説明に従って名前を大文字にしてスペルを指定します。 

8. クライアント アプリケーションでは、デプロイされたモデルを使用するために、**エンドポイント URL** と**主キー**が必要です。 デプロイが完了したら、これらのパラメーターを取得するために、[https://portal.azure.com](https://portal.azure.com/?azure-portal=true) で Azure portal を開き、Azure サブスクリプションに関連付けられた Microsoft アカウントを使用してサインインします。 検索バーで、「**Language**」と検索して選択し、*Azure AI Services|Language サービス*を選択します。

9. 言語サービス リソースが一覧表示されるので、そのリソースを選択します。

10. 左側のメニューの *[リソース管理]* セクションで、 **[キーとエンドポイント]** を選択します。

11. **KEY 1** と**エンドポイント**のコピーを作成します。

12. クライアント アプリケーションでは、デプロイされたモデルに接続して認証するために、予測 URL エンドポイントと言語サービス キーからの情報が必要です。

## Language service SDK を使用する準備をする

この演習では、Clock モデル (公開済みの Conversational Language Understanding モデル) を使用してユーザー入力から意図を予測し、適切に応答する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **.NET** または **Python** 用の SDK のいずれかを使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**10b-clu-client-(preview)** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。

2. **clock-client** フォルダーを右クリックして、 **[Open in Integrated Terminal](統合ターミナルで開く)** を選択します。 次に、言語設定に適したコマンドを実行して、Conversational Language Service SDK パッケージをインストールします

    **C#**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.0.0
    dotnet add package Azure.Core
    ```

    **Python**

    ```
    pip install azure-ai-language-conversations --pre
    python -m pip install python-dotenv
    python -m pip install python-dateutil
    ```

3. **clock-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。

    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、ご自分の言語リソースの**エンドポイント URL** と**主キー**を反映します。 必要な値は、Azure portal または Language Studio で次のように見つけることができます。

    - Azure  portal:言語リソースを開きます。 **[リソース管理]** の下の **[キーとエンドポイント]** を選択します。 **KEY 1** と**エンドポイント**の値を構成設定ファイルにコピーします。
    - Language Studio:**Clock** プロジェクトを開きます。 言語サービス エンドポイントは、 **[予測 URL の取得]** の **[モデルのデプロイ]** ページで確認でき、**主キー**は **[プロジェクトの設定]** ページで確認できます。 予測 URL の言語サービス エンドポイント部分は、 **.cognitiveservices.azure.com/** で終わります。 (例: `https://ai102-langserv.cognitiveservices.azure.com/`)。

4. **clock-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: clock-client.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、"**Import namespaces**" というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Language service SDK を使用するために必要な名前空間をインポートします。

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

## 会話言語モデルから予測を取得する

これで、SDK を使用して会話言語モデルから予測を取得するコードを実装する準備が整いました。

1. **Main** 関数では、構成ファイルから予測エンドポイント、およびキーを読み込むためのコードが既に提供されていることに注意してください。 次に、コメント "**Create a client for the Language service model**" を見つけ、次のコードを追加して、言語サービス アプリの予測クライアントを作成します。

    **C#**

    ```C#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python**

    ```Python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

2. ユーザーが「quit」と入力するまで、**Main** 関数のコードはユーザー入力を求めるプロンプトを表示することにご注意ください。 このループ内で、コメント "**Call the Language service model to get intent and entities**" を見つけて、次のコードを追加します。

    **C#**

    ```C#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python**

    ```Python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    言語サービス モデルを呼び出すと、予測または結果が返されます。これには、入力発話で検出されたエンティティだけでなく、最上位の (最も可能性の高い) 意図も含まれます。 クライアント アプリケーションは、その予測を使用して適切なアクションを決定および実行する必要があります。

3. コメント **Apply the appropriate action** を見つけ、次のコードを追加します。このコードは、アプリケーションでサポートされている意図 (**GetTime**、**GetDate**、および **GetDay**) をチェックします。また、適切な応答を生成するために既存の関数を呼び出す前に、関連するエンティティが検出されたかどうかを判断します。

    **C#**

    ```C#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**

    ```Python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

4. 変更を保存し、**clock-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python clock-client.py
    ```

5. プロンプトが表示されたら、発話を入力してアプリケーションをテストします。 たとえば、次の操作を試してください。

    *Hello*
    
    *What time is it?*

    *What's the time in London?* \(ロンドンの時刻は何時ですか?\)

    *What's the date?* \(何日ですか?\)

    *What date is Sunday?* \(日曜日は何日ですか?\)

    *What day is it?* \(何曜日ですか?\)

    *What day is 01/01/2025?* \(2025 年 1 月 1 日は何曜日ですか?\)

> **注**: アプリケーションのロジックは意図的に単純であり、いくつかの制限があります。 たとえば、時間を取得する場合、制限された都市のセットのみがサポートされ、夏時間は無視されます。 目標は、アプリケーションが次のことを行う必要がある言語サービスを使用するための一般的なパターンの例を確認することです。
>
>   1. 予測エンドポイントに接続します。
>   2. 予測を得るために発話を送信します。
>   3. 予測された意図とエンティティに適切に応答するロジックを実装します。

6. テストが終了したら、「*quit*」と入力します。

## 詳細情報

言語サービス クライアントの作成の詳細については、[開発者向けドキュメント](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)を参照してください。
