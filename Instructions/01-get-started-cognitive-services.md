---
lab:
  title: Azure AI サービスの使用を開始する
  module: Module 2 - Developing AI Apps with Azure AI Services
---

# Azure AI サービスの使用を開始する

この演習では、Azure サブスクリプションで **Azure AI サービス** リソースを作成し、そのリソースをクライアント アプリケーションから使用して、Azure AI サービスの使用を開始します。 この演習の目標は、特定のサービスに関する専門知識を習得することではなく、Azure AI サービスを開発者としてプロビジョニングおよび操作するための一般的なパターンに精通することです。

## このコースのリポジトリを複製する

このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
2. コマンドパレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## Azure AI サービス リソースをプロビジョニングする

Azure AI サービスは、アプリケーションに組み込むことができる人工知能機能をカプセル化したクラウドベースのサービスです。 特定の API (**Language** または **Vision** など) に個別の **Azure AI サービス** リソースをプロビジョニングすることも、単一のエンドポイントとキーを介して複数の Azure AI サービス API へのアクセスを提供する単一の Azure AI サービス リソースをプロビジョニングすることもできます。 今回は、単一の **Azure AI サービス** リソースを使用します。

1. Azure portal (`https://portal.azure.com/?l=ja.ja-jp`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで「*Azure AI*」を検索し、**Azure AI services multi-service account**を選択し、次の設定で Azure AI サービスのマルチサービス アカウント リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"

    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースに移動し、その **[キーとエンドポイント]** ページを表示します。 このページには、リソースに接続して、開発したアプリケーションからリソースを使用するために必要な情報が含まれています。 具体的な内容は次のとおりです。
    - クライアント アプリケーションが要求を送信できる HTTP *エンドポイント*。
    - 認証に使用できる 2 つの "*キー*" (クライアント アプリケーションはどちらのキーも認証に使用できます)。
    - リソースがホストされている "*場所*"。 これは、一部の (すべてではない) API へのリクエストに必要です。

## REST インターフェイスの使用

Azure AI サービス API は REST ベースであるため、HTTP 経由で JSON 要求を送信することで API を利用できます。 この例では、**Language** REST API を使用して言語検出を実行するコンソール アプリケーションについて説明します。ただし、基本的な原則は、Azure AI サービス リソースでサポートされているすべての API で同じです。

> **注**: この演習では、**C#** または **Python** のいずれかから REST API を使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **[エクスプローラー]** ウィンドウで、**01-getting-started** フォルダーを参照し、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **rest-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、構成値を更新して、Azure AI サービス リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。
3. **rest-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: rest-client.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - HTTP 通信を可能にするために、さまざまな名前空間がインポートされます
    - **Main** 関数のコードは、Azure AI サービス リソースのエンドポイントとキーを取得します。これらは、REST 要求を Text Analytics サービスに送信するために使用されます。
    - プログラムはユーザー入力を受け入れ、**GetLanguage** 関数を使用して、Azure AI サービス エンドポイントの Text Analytics 言語検出 REST API を呼び出して、入力されたテキストの言語を検出します。
    - API に送信される要求は、入力データを含む JSON オブジェクトで構成されます。この場合、**ドキュメント** オブジェクトのコレクションであり、それぞれに **ID** と**テキスト**があります。
    - サービスのキーは、クライアント アプリケーションを認証するためのリクエスト ヘッダーに含まれています。
    - サービスからの応答は JSON オブジェクトであり、クライアント アプリケーションはこれを解析できます。
4. **rest-client** フォルダーを右クリックして、統合ターミナルを開きます(Open in Integrated Terminal)。 次に、次の言語固有のコマンドを入力して、プログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python rest-client.py
    ```

5. プロンプトが表示されたら、テキストを入力し、サービスによって検出された言語を確認します。これは、JSON 応答で返されます。 たとえば、「Hello」、「Bonjour」、「Gracias」と入力してみてください。
6. アプリケーションのテストが終了したら、「quit」と入力してプログラムを終了します。

## SDK を使用する

Azure AI サービスの REST API を直接使用するコードを記述することもできますが、Microsoft C#、Python、Node.js など、多くの一般的なプログラミング言語用のソフトウェア開発キット (SDK) を利用できます。 SDK を使用すると、Azure AI サービスを使用するアプリケーションの開発を大幅に簡素化できます。

1. Visual Studio Code の **[エクスプローラー]** ウィンドウの **01-getting-started** フォルダーで、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **sdk-client** フォルダーを右クリックして、統合ターミナルを開きます(Open in Integrated Terminal)。 次に、言語設定に適合するコマンドを実行して、Text Analytics SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

3. **sdk-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、構成値を更新して、Azure AI サービス リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。
    
4. **sdk-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: sdk-client.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - インストールした SDK の名前空間がインポートされます
    - **Main** 関数のコードは、Azure AI サービス リソースのエンドポイントとキーを取得します。これらは SDK で使用され、Text Analytics サービスのクライアントを作成します。
    - **GetLanguage** 関数は、SDK を使用してサービスのクライアントを作成し、クライアントを使用して入力されたテキストの言語を検出します。
5. **sdk-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python sdk-client.py
    ```

6. プロンプトが表示されたら、テキストを入力し、サービスによって検出された言語を確認します。 たとえば、「Goodbye」、「Au revoir」、「Hasta la vista」と入力してみてください。
7. アプリケーションのテストが終了したら、「quit」と入力してプログラムを終了します。

> **注**: Unicode 文字セットを必要とする一部の言語は、この単純なコンソール アプリケーションでは認識されない場合があります。

## 詳細情報

Azure AI サービスの詳細については、[Azure AI サービスのドキュメント](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services)を参照してください。
