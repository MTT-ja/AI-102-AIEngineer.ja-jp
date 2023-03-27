---
lab:
  title: 画像内のテキストの読み取り
  module: Module 11 - Reading Text in Images and Documents
---

# <a name="read-text-in-images"></a>画像内のテキストの読み取り

光学式文字認識 (OCR) は、画像およびドキュメント内のテキストの読み取りを処理する Computer Vision のサブセットです。 **Computer Vision** サービスにより、テキストを読み取るための 2 つの API が提供されます。これについては、この演習で説明します。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="provision-a-cognitive-services-resource"></a>Cognitive Services リソースをプロビジョニングする

サブスクリプションに **Cognitive Services** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com/?l=ja.ja-jp`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:*ご自身の Azure サブスクリプション*
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)* 
    - **[リージョン]**: *使用できるリージョンを選択します*
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Computer Vision SDKを使用する準備をする

この演習では、Computer VisionSDK を使用してテキストを読み取る部分的に実装されたクライアント アプリケーションを完成させます。

> **注**:**C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**20-ocr** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **read-text** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Computer Vision SDK パッケージをインストールします。

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

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。
4. **read-text** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: read-text.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

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

5. クライアント アプリケーションのコードファイルで **Main** 関数に、構成設定を読み込むためのコードが提供されていることにご注意ください。 次に、コメント「**Authenticate Computer Vision client**」を見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision クライアント オブジェクトを作成および認証します

**C#**

```C#
// Authenticate Computer Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Computer Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```


## <a name="use-the-read-api"></a>Read API を使用して画像からテキストを読み取る

**Read** API は新しいテキスト認識モデルが使用されており、多くのテキストを含む大きな画像に対してより優れたパフォーマンスを発揮します。 また、 *.pdf* ファイルからのテキスト抽出をサポートし、印刷されたテキストと手書きのテキストの両方を複数の言語で認識できます。

**Read** API は、テキスト認識を開始する要求が送信される非同期操作モデルを使用します。その後、リクエストから返された操作 ID を使用して、進行状況を確認し、結果を取得できます。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。 このコードは **GetTextRead** 関数を呼び出し、パスを PDF ドキュメント ファイルに渡します。
2. **read-text/images** フォルダーで、**Rome.pdf**を右クリックし、 **[ファイルエクスプローラーで表示]** を選択します。 次に、ファイル エクスプローラーで、PDF ファイルを開いて表示します。
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

6. ダイアログが表示されたら、**1** を入力し、ドキュメントから抽出されたテキストである出力を確認します。
7. 必要であれば、**GetTextRead**に追加したコードに戻り、最後にネストした`for`ループのコメントを見つけ、最後の行のコメントを解除してファイルを保存し、上記のステップ5と6を再実行して各行の境界ボックスを確認します。次に進む前に、必ずその行を再コメントし、ファイルを保存してください。

## Read API を使って文書からテキストを読み込む

1. アプリケーションのコードファイルの **Main** 関数の中で、ユーザーがメニューオプション **2** を選択した場合に実行されるコードを調べます。このコードでは、**GetTextRead**関数を呼び出し、PDF文書ファイルへのパスを渡します。
2. **read-text/images**フォルダで、**Rome.pdf**を右クリックし、 **Reveal in File Explorer** を選択します。次に、ファイルエクスプローラーで、PDFファイルを開いて表示する。
3. **read-text**フォルダの統合端末に戻り、以下のコマンドを入力してプログラムを実行します：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. プロンプトが表示されたら、**2**と入力し、文書から抽出されたテキストである出力を観察してください。

## <a name="read-handwritten-text"></a>手書きテキストの読み取り

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

## <a name="more-information"></a>詳細情報

**Computer Vision** サービスを使用してテキストを読み取る方法の詳細については、[Computer Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text)を参照してください。
