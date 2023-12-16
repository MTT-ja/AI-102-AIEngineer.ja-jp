---
lab:
  title: テキストの翻訳
  module: Module 3 - Getting Started with Natural Language Processing
---

# テキストの翻訳

**Azure AI 翻訳**は、言語間でテキストを翻訳できるようにするサービスです。

たとえば、旅行代理店が、分析に使用される言語として英語を標準化して、会社の Web サイトに送信されたホテルのレビューを調べたいとします。 Azure AI 翻訳サービスを使用すると、各レビューが書かれている言語を判別し、レビューがまだ英語でない場合は、使用されているソース言語から英語に翻訳できます。

## Azure Cloud Shell にリポジトリのクローンを作成します

1. [Azure portal](https://portal.azure.com?azure-portal=true) で、ページ上部の検索ボックスの右側にある **[>_]** (*Cloud Shell*) ボタンを選びます。 ポータルの下部に Cloud Shell ペインが開きます。

    ![上部の検索ボックスの右側にあるアイコンをクリックして Cloud Shell を開始している状態のスクリーンショット。](images/cloudshell-launch-portal.png#lightbox)

1. Cloud Shell を初めて開くと、使用するシェルの種類 (*Bash* または *PowerShell*) を選択するように求められる場合があります。 **[PowerShell]** を選択します。 このオプションが表示されない場合は、この手順をスキップします。

1. Cloud Shell のストレージを作成するように求めるメッセージが表示された場合は、お使いのサブスクリプションが指定されていることを確認して、**[ストレージの作成]** を選択します。 その後、ストレージが作成されるのを 1 分程度待ちます。

1. ターミナルが起動したら、次のコマンドを実行して、リポジトリのコピーを Cloud Shell にダウンロードします。

    ```bash
    rm -r azure-ai-eng -f
    git clone https://github.com/MicrosoftLearning/AI-102-AIEngineer azure-ai-eng
    ```

1. ファイルは **azure-ai-eng** というフォルダーにダウンロードされています。 Cloud Shell コード エディターを使用して、次を実行し、適切なフォルダーを開きます。

    ```bash
        code azure-ai-eng/06-translate-text
    ```

## Azure AI Translator リソースをプロビジョニングする

サブスクリプションに **Azure AI 翻訳**リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1. 上部の検索フィールドで、「**Azure AI サービス**」を検索し、**Enter キー**を押して、結果の翻訳の下にある [**作成**] を選択します。
1. 次の設定を使ってリソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: *米国東部*
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: *Standard S1*
1. [**作成と確認**] を選択し、次に [**作成**] を選択します。
1. デプロイが完了するまで待ち、デプロイの詳細を表示します。
1. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページからサービスがプロビジョニングされるキーと場所の 1 つが必要になります。

## Azure AI 翻訳を使用するための準備をする

この演習では、Azure AI Translator REST API を使用してホテルのレビューを翻訳する部分的に実装されたクライアント アプリケーションを完成させます。

> [!NOTE]
> **C#** または **Python** のいずれかから API を使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Cloud Shell エディターで、言語の設定に応じて、 **06-translate-text** フォルダーを展開し、次に **C-Sharp** または **Python** フォルダーを展開します。
1. **text-translation** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、そこに含まれる構成値を更新して、Azure AI 翻訳リソースの認証 **キー**と、それがデプロイされている**場所** (エンドポイントでは<u>ない</u>) を含めます。Azure AI 翻訳リソース用に、**キーとエンドポイント**ページからこれらの両方をコピーする必要があります。 変更を保存するには、**Ctrl + S** キーを押します。
1. **text-translation** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: text-translation.py

    コード ファイルを開き、含まれているコードを調べます。

1. **Main** 関数では、構成ファイルから Azure AI 翻訳のキーとリージョンを読み込むためのコードが既に提供されていることに注意してください。 サービスのエンドポイントもコードで指定されています。

1. ターミナルで次のコマンドを入力してターミナルを適切なフォルダーにポイントし、テストを実行します。

    **C#**

    ```bash
    cd AI-Engineer/06-translate-text/C-Sharp/text-translation
    ```

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    cd AI-Engineer/06-translate-text/Python/text-translation
    ```

    **Python**

    ```bash
    pip install python-dotenv
    ```

    **Python**

    ```bash
    python text-translation.py
    ```

1. コードがエラーなしで実行され、**reviews** フォルダー内の各レビュー テキストファイルの内容が表示されるので、出力を確認します。 現在、アプリケーションは Azure AI 翻訳を利用していません。 次の手順で修正します。

## 言語を検出する

Azure AI 翻訳は、翻訳対象のテキストのソース言語を自動的に検出できますが、テキストが記述されている言語を明示的に検出することもできます。

1. コード ファイルで、**GetLanguage** 関数を見つけます。これにより、現在、すべてのテキスト値に対して "en" が返されます。
1. **GetLanguage** 関数で、「**Use the Azure AI Translator detect function**」というコメントの下に、Azure AI 翻訳 の REST API を使用して、指定されたテキストの言語を検出するために、次のコードを追加します。

    **C#**

    ```csharp
    // Use the Azure AI Translator detect function
    object[] body = new object[] { new { Text = text } };
    var requestBody = JsonConvert.SerializeObject(body);
    using (var client = new HttpClient())
    {
        using (var request = new HttpRequestMessage())
        {
            // Build the request
            string path = "/detect?api-version=3.0";
            request.Method = HttpMethod.Post;
            request.RequestUri = new Uri(translatorEndpoint + path);
            request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
            request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
            request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);
    
            // Send the request and get response
            HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
            // Read response as a string
            string responseContent = await response.Content.ReadAsStringAsync();
    
            // Parse JSON array and get language
            JArray jsonResponse = JArray.Parse(responseContent);
            language = (string)jsonResponse[0]["language"]; 
        }
    }
    ```

    **Python**

    ```python
    # Use the Azure AI Translator detect function
    path = '/detect'
    url = translator_endpoint + path
    
    # Build the request
    params = {
        'api-version': '3.0'
    }
    
    headers = {
    'Ocp-Apim-Subscription-Key': cog_key,
    'Ocp-Apim-Subscription-Region': cog_region,
    'Content-type': 'application/json'
    }
    
    body = [{
        'text': text
    }]
    
    # Send the request and get response
    request = requests.post(url, params=params, headers=headers, json=body)
    response = request.json()
    
    # Parse JSON array and get language
    language = response[0]["language"]
    ```

1. **Ctrl + S キー**を押して変更を保存し、ターミナルで次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python text-translation.py
    ```

1. 出力を観察します。今回は、各レビューの言語が識別されていることに注意してください。

## テキストを翻訳する

アプリケーションがレビューの作成言語を判別できるようになったので、Azure AI 翻訳を使用して、英語以外のレビューを英語に翻訳できます。

1. コード ファイルで、**Translate** 関数を見つけます。これにより、現在、すべてのテキスト値に対して空の文字列が返されます。
1. **Translate** 関数の「**Use the Translator translate function**」というコメントの下に、次のコードを追加して、Azure AI 翻訳の REST API を使用し、指定されたテキストをソース言語から英語に翻訳します。翻訳を返す関数の最後にあるコードを置き換えないようにご注意ください。

**C#**

```csharp
// Use the Azure AI Translator translate function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/translate?api-version=3.0&from=" + sourceLanguage + "&to=en" ;
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get translation
        JArray jsonResponse = JArray.Parse(responseContent);
        translation = (string)jsonResponse[0]["translations"][0]["text"];  
    }
}
```

**Python**

```python
# Use the Azure AI Translator translate function
path = '/translate'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0',
    'from': source_language,
    'to': ['en']
}

headers = {
    'Ocp-Apim-Subscription-Key': cog_key,
    'Ocp-Apim-Subscription-Region': cog_region,
    'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get translation
translation = response[0]["translations"][0]["text"]
```

1. 変更を保存し、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python text-translation.py
    ```

1. 英語以外のレビューは英語に翻訳されていることに注意して、出力を観察します。

## 詳細情報

**Azure AI 翻訳**の使用の詳細については、「[Azure AI 翻訳のドキュメント](/azure/ai-services/translator/)」を参照してください。
