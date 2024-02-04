---
lab:
  title: 音声の認識と合成
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# 音声の認識と合成

**Azure AI 音声**は、次のような音声関連機能を提供するサービスです。

- 音声認識 (音声の話し言葉をテキストに変換する) を実装できるようにする *speech-to-text* API。
- 音声合成 (テキストを可聴音声に変換する) を実装できるようにする*text-to-speech* API。

この演習では、これらの API の両方を使用して、スピーキング クロック アプリケーションを実装します。

> [!NOTE]
> この演習では、スピーカー / ヘッドフォン対応のコンピューターを使用する必要があります。 最良のエクスペリエンスのため、マイクも必要です。 一部のホストされる仮想環境では、ローカル マイクから音声をキャプチャできる場合があります。しかし、これが機能しない場合 (または、マイクがない場合)、音声入力用に付属の音声ファイルを使用できます。 マイクまたは音声ファイルを使用するかどうかに応じて、ことなるオプションを選択する必要があるろきは、慎重に手順に従ってください。

## このコースのリポジトリを複製する

このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
1. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
1. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
1. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

> [!NOTE]
> ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、**[後で]** を選択します。

## Azure AI 音声のリソースをプロビジョニングする

サブスクリプションにまだない場合は、**Azure AI 音声サービス** リソースをプロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1. ポータルの上部にある検索フィールドに「**Azure AI**」と入力します。 次に、表示される候補のドロップダウンで **Azure AI サービス**を選択します。
1. 結果ページの **Speech Service** で **[作成]** を選択します。
1. 次の設定を使ってリソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
1. **[確認および作成]** を選択し、 **[作成]** を選択します。
1. デプロイが完了するまで待ち、デプロイの詳細を表示します。
1. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページからサービスがプロビジョニングされるキーと場所の 1 つが必要になります。

## Azure AI 音声サービスを使用するための準備

この演習では、Azure AI 音声 SDK を使用して音声を認識および合成する、部分的に実装されたクライアント アプリケーションを完成させます。

> [!NOTE]
> **C#** または **Python** 用の SDK のいずれかに使用するよう選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ペインで、**07-speech** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
1. **speaking-clock** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Speech SDK パッケージをインストールします。

    **C#**

    ```bash
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```bash
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. **speaking-clock** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、そこに含まれる構成値を更新して、Azure AI 音声リソースの認証**キー**と、それが展開されている**場所**を含めます。 変更を保存します。
1. **speaking-clock** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#** : Program.cs
    - **Python**: speaking-clock.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI 音声 SDK を使用するために必要な名前空間をインポートします。

    **C#**

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. **Main** 関数では、構成ファイルからキーとリージョンをロードするコードがすでに提供されていることに注意してください。 Azure AI 音声リソースの **SpeechConfig** を作成するには、これらの変数を使用する必要があります。 コメント **「Configure speech service」** の下に次のコードを追加します。

    **C#**

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. 変更を保存して、**speaking-clock** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python speaking-clock.py
    ```

1. C# を使用している場合は、非同期メソッドで **await** 演算子を使用することに関する警告を無視できます。これは後で修正します。 コードは、アプリケーションが使用する音声サービスリソースの領域を表示する必要があります。

## 音声を認識する

Azure AI 音声リソースに音声サービス用の **SpeechConfig** があるので、**Speech-to-text** API を使用して音声を認識し、テキストに転写することができます。

### マイクが機能する場合

1. プログラムの **Main** 関数で、コードが **TranscribeCommand** 関数を使用して音声入力を受け入れることに注意してください。
1. **TranscribeCommand** 関数のコメント **「Configure speech recognition」** の下に、次のコードを追加して、入力用のデフォルトのシステムマイクを使用して音声を認識および転写するために使用できる **SpeechRecognizer** クライアントを作成します。

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. 以下の「**コードを追加して書き起こしたマンドを処理する**」のセクションにスキップします。

### または、ファイルからの音声入力を使用します

1. ターミナル ウィンドウで、次のコマンドを入力して、音声ファイルの再生に使用できるライブラリをインストールします。

    **C#**

    ```bash
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```bash
    pip install playsound==1.3.0
    ```

1. プログラムのコード ファイルで、既存の名前空間のインポートの下に、次のコードを追加して、インストールしたライブラリをインポートします。

    **C#**

    ```csharp
    using System.Media;
    ```

    **Python**

    ```python
    from playsound import playsound
    ```

1. **Main** 関数で、コードが **TranscribeCommand** 関数を使用して音声入力を受け入れることに注意してください。 **TranscribeCommand** 関数のコメント **「Configure speech recognition」** の下に、適切なコードを追加して、音声ファイルからの音声を認識して、書き起こすために使用できる **SpeechRecognizer** クライアントを作成します。

    **C#**

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

### コードを追加して書き起こしたコマンドを処理する

1. **TranscribeCommand** 関数のコメント **「Process speech input」** の下に、音声入力をリッスンする次のコードを追加します。コマンドを返す関数の最後にあるコードを置き換えないように注意してください。

    **C#**

    ```csharp
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**

    ```python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

1. 変更を保存して、**speaking-clock** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python speaking-clock.py
    ```

1. マイクを使用している場合、明瞭に話、"what time is it?" と言ってください。 プログラムは、音声入力を書き起こし、時刻を表示する必要があります (コードが実行されているコンピューターの現地時間に基づいており、現在の時刻とは異なる場合があります)。

    SpeechRecognizer を使用すると、約 5 秒で話すことができます。 音声入力が検出されない場合は、[一致なし] の結果が生成されます。

    SpeechRecognizer でエラーが発生した場合は、"Cancelled" の結果が生成されます。 アプリケーションのコードは、エラーメッセージを表示します。 最も可能性の高い原因は、構成ファイルのキーまたはリージョンが正しくないことです。

## 音声を合成する

speaking clock アプリケーションは話し言葉の入力を受け入れますが、実際には話しません。 音声合成用のコードを追加して修正しましょう。

1. プログラムの **Main** 関数で、コードが **TellTime** 関数を使用してユーザーに現在の時刻を通知することに注意してください。
1. **TellTime** 関数のコメント **「Configure speech synthesis」** の下に、次のコードを追加して、音声出力の生成に使用できる **SpeechSynthesizer** クライアントを作成します。

    **C#**

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

> [!NOTE]
>  *デフォルトのオーディオ構成では、出力にデフォルトのシステム オーディオ デバイスが使用されるため、**AudioConfig** を明示的に指定する必要はありません。オーディオ出力をファイルにリダイレクトする必要がある場合は、ファイルパスを指定して **AudioConfig** を使用できます。*

1. **TellTime** 関数のコメント **「Synthesize spoken output」** の下に、次のコードを追加して音声出力を生成します。応答を出力する関数の最後にあるコードを置き換えないように注意してください。

    **C#**

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 変更を保存して、**speaking-clock** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python speaking-clock.py
    ```

1. プロンプトが表示されたら、マイクに向かってはっきりと話し、"what time is it?" と言います。 プログラムが時間を教えくれるはずです。

## 別の音声を使用する

speaking clock アプリケーションは、変更可能なデフォルトの音声を使用します。 Speech サービスは、さまざまな*標準*音声だけでなく、より人間らしい*ニューラル*音声もサポートします。 *カスタム* ボイスを作成することもできます。

> **注**:ニューラル音声と標準音声のリストについては、Speech サービスのドキュメントの[言語と音声のサポート](/azure/ai-services/speech-service/language-support?tabs=stt#text-to-speech)を参照してください。

1. **TellTime** 関数のコメント **「Configure speech synthesis」** で、**SpeechSynthesizer** クライアントを作成する前に、次のようにコードを変更して代替音声を指定します。

   **C#**

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. 変更を保存して、**speaking-clock** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python speaking-clock.py
    ```

3. プロンプトが表示されたら、マイクに向かってはっきりと話し、"what time is it?" と言います。 プログラムは指定された声で話し、時間を伝えます。

## 音声合成マークアップ言語を使用する

音声合成アップ言語 (SSML) を使用すると、XML ベースの形式を使用して音声を合成する方法をカスタマイズできます。

1. **TellTime** 関数で、コメント **「Synthesize spoken output」** の下にある現在のすべてのコードを次のコードに置き換えます (コメント **Print the response** の下にコードを残します)。

   **C#**

    ```csharp
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**

    ```python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 変更を保存して、**speaking-clock** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```bash
    dotnet run
    ```

    **Python**

    ```bash
    python speaking-clock.py
    ```

1. プロンプトが表示されたら、マイクに向かってはっきりと話し、"何時ですか?" と言います。 プログラムは、SSML で指定された音声 (SpeechConfig で指定された音声を上書き) で話し、時間を通知し、一時停止した後、このラボを終了する時間であることを通知する必要があります。

## 詳細情報

**Speech-to-text** および **Text-to-speech** API の使用の詳細については、[Speech-to-text ドキュメント](/azure/ai-services/speech-service/index-speech-to-text)および [Text-to-speech ドキュメント](/azure/ai-services/speech-service/index-text-to-speech)を参照してください。
