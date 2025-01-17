---
lab:
  title: 画像内のテキストの読み取り
  module: Module 11 - Reading Text in Images and Documents
---

# 画像内のテキストの読み取り

光学式文字認識 (OCR) は、画像およびドキュメント内のテキストの読み取りを処理する Computer Vision のサブセットです。 **Azure AI Vision** サービスにより、テキストを読み取るための 2 つの API が提供されます。これについては、この演習で説明します。

## このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## Azure AI サービス リソースをプロビジョニングする

サブスクリプションにまだリソースがない場合は、**Azure AI サービス** リソースをプロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで「*Azure AI サービス*」を検索し、**Azure AI サービス**を選択し、次の設定で Azure AI サービスのマルチサービス アカウント リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## Azure AI Vision SDK を使用するための準備

この演習では、Azure AI Vision SDK を使用してテキストを読み取る部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**20-ocr** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **read-text** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Azure AI Vision SDK パッケージをインストールします。

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 7.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.9.0
```

3. **read-text** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、構成値を更新して、Azure AI サービス リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。
4. **read-text** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: read-text.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision SDK を使用するために必要な名前空間をインポートします。

**C#**

```C#
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```

5. クライアント アプリケーションのコードファイルで **Main** 関数に、構成設定を読み込むためのコードが提供されていることにご注意ください。 次に、「**Authenticate Azure AI Vision client**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision クライアント オブジェクトを作成および認証します。

**C#**

```C#
// Authenticate Azure AI Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Azure AI Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```

## Read API を使用して画像からテキストを読み取る

**Read** API では、新しいテキスト認識モデルを使用しており、一般には大量のテキストが含まれている大きな画像に対してパフォーマンスが向上しますが、テキストの量にかかわらず機能します。 また、 *.pdf* ファイルからのテキスト抽出をサポートし、印刷されたテキストと手書きのテキストの両方を複数の言語で認識できます。

**Read** API は、テキスト認識を開始する要求が送信される非同期操作モデルを使用します。その後、リクエストから返された操作 ID を使用して、進行状況を確認し、結果を取得できます。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。 このコードによって **GetTextRead** 関数が呼び出されて、画像ファイルにパスが渡されます。
2. **read-text/images** フォルダーで、**Lincoln.jpg** をクリックし、コードによって処理されるファイルを表示します。
3. Visual Studio Code のコードファイルに戻り、**GetTextRead** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C#
// Use Read API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // Get the async operation ID so we can check for the results
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // Wait for the asynchronous operation to complete
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // If the operation was successfully, process the text line by line
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);
                
                // Uncomment the following line if you'd like to see the bounding box 
                //Console.WriteLine(line.BoundingBox);
            }
        }
    }
}  
```

**Python**

```Python
# Use Read API to read text in image
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # Get the async operation ID so we can check for the results
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # Wait for the asynchronous operation to complete
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # If the operation was successfully, process the text line by line
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
                # Uncomment the following line if you'd like to see the bounding box 
                #print(line.bounding_box)
```

4. **GetTextRead** 関数に追加したコードを調べます。 読み取り操作の要求を送信し、操作が完了するまでステータスを繰り返しチェックします。 成功した場合、コードは各ページを繰り返し処理し、次に各行を繰り返し処理して結果を処理します。
5. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```


6. プロンプトが表示されたら、**1** を入力して、画像から抽出されたテキストである出力を確認します。
7. 必要に応じて、**GetTextRead** に追加したコードに戻り、最後にある入れ子になった `for` ループ内でコメントを見つけ、最後の行のコメントを解除して、ファイルを保存し、上記の手順 5 と 6 を実行して、各行の境界ボックスを表示します。 次に進む前に、必ずその行にコメントを付け直し、ファイルを保存してください。

## Read API を使用してドキュメントからテキストを読み取る

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **2** を選択した場合に実行されるコードを調べます。 このコードは **GetTextRead** 関数を呼び出し、パスを PDF ドキュメント ファイルに渡します。
2. **read-text/images** フォルダーで、**Rome.pdf**を右クリックし、 **[ファイルエクスプローラーで表示]** を選択します。 次に、ファイル エクスプローラーで、PDF ファイルを開いて表示します。
3. **read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. ダイアログが表示されたら、**2** を入力し、ドキュメントから抽出されたテキストである出力を確認します。

## 手書きテキストの読み取り

**Read** API は、印刷されたテキスト以外にも、英語の手書きのテキストを抽出できます。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **3** を選択した場合に実行されるコードを調べます。 このコードは **GetTextRead** 関数を呼び出し、パスを画像ファイルに渡します。
2. **read-text/images** フォルダーで、**Note.jpg** を開いて、コードによって処理される画像を表示します。
3. **read-text** フォルダーの統合ターミナルで、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. ダイアログが表示されたら、**3** を入力し、ドキュメントから抽出されたテキストである出力を確認します。

## 詳細情報

**Azure AI Vision** サービスを使用してテキストを読み取る方法の詳細については、「[Azure AI Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text)」を参照してください。
