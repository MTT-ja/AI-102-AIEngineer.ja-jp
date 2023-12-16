---
lab:
  title: ラボ環境のセットアップ
  module: Setup
---

# ラボ環境のセットアップ

これらの演習は、ホストされたラボ環境で完了するように設計されています。 ご自分のパソコンで完成させたい場合は、以下のソフトウェアをインストールしてください。 独自の環境を使用すると、予期しないダイアログや動作が発生する場合があります。 また、独自の環境は大きく異なる可能性があるため、コースチームは発生する問題をサポートできない場合があります。

> **注:** 以下の手順は、Windows 10 コンピューター用です。 Linux または MacOS を使用することもできます。 選択した OS に応じたラボ手順を適応する必要があります。

### 基本オペレーティング システム (Windows 10)

#### Windows 10

Windows 10 をインストールし、すべての更新プログラムを適用します。

#### Edge

[Edge (Chromium)](https://microsoft.com/edge) をインストールします

### .NET Core SDK

1. https://dotnet.microsoft.com/download からダウンロードしてインストールします (ランタイムだけでなく、.NET Core SDK をダウンロードします)。 このコースのラボを自分のコンピューターで実行している場合は、.NET 7.0 がインストールされている必要があります。

### C++ 再頒布可能パッケージ

1. https://aka.ms/vs/16/release/vc_redist.x64.exe から Visual C++ 再頒布可能パッケージ (x64) をダウンロードしてインストールします

### Node.JS

1. https://nodejs.org/en/download/ から最新の LTS バージョンをダウンロードします 
2. 既定のオプションを使用してインストールします

### Python (および必要なパッケージ)

1. https://docs.conda.io/en/latest/miniconda.html からバージョン3.8 をダウンロードします 
2. セットアップを実行してインストールします - **重要**: Miniconda を PATH 変数に追加し、Miniconda を既定の Python 環境として登録するオプションを選択します。
3. インストール後、Anaconda プロンプトを開き、次のコマンドを入力してパッケージをインストールします。 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### Azure CLI

1. https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest からダウンロードします 
2. 既定のオプションを使用してインストールします

### Git

1. 既定のオプションを使用して https://git-scm.com/download.html からダウンロードしてインストールします


### Visual Studio Code (および拡張機能)

1. https://code.visualstudio.com/Download からダウンロードします 
2. 既定のオプションを使用してインストールします 
3. インストール後、Visual Studio Code を起動し、 **[拡張機能]** タブ (CTRL + SHIFT + X) で、Microsoft から次の拡張機能を検索してインストールします。
    - Python
    - C#
    - Azure Functions
    - PowerShell


### Bot Framework エミュレーター

https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md の手順に従って、お使いのオペレーティングシステム用の最新の安定バージョンの Bot Framework Emulator をダウンロードしてインストールします。

### Bot Framework Composer

https://docs.microsoft.com/en-us/composer/install-composer からインストールします。
