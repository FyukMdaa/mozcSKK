WindowsでMozcをビルドする方法
## コマンド一覧
詳しくはセットアップ以下を確認してください。  
また、以下作業はDeveloper PowerShell for VS (バージョン名)で実行してください
```
# 必要なツールのインストール
python -m pip install six requests
# 必要なツールのインストールなので毎回する必要はなし

git clone https://github.com/FyukMdaa/mozcSKK.git --depth 1
cd mozcSKK\src

python build_tools/update_deps.py
python build_tools/build_qt.py --release --confirm_license
python build_mozc.py gyp
python build_mozc.py build -c Release package

# mozcSKKのインストール
out_win\Release\Mozc64.msi
```

## セットアップ

### システム要件

64ビット版のWindows 10以降。

### ソフトウェア要件

WindowsでMozcをビルドするには、以下のソフトウェアが必要です。

  * [Visual Studio 2022 Community Edition](https://visualstudio.microsoft.com/downloads/#visual-studio-community-2022)
    * [Build Tools for Visual Studio 2022](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022)も動作します
  * Python 3.9以降と以下のpipモジュール。
    * `six`
    * `requests`
  * `.NET 6`以降（`dotnet`コマンド用）。


### pipモジュールのインストール

```
python3 -m pip install six requests
```

### GitHubからリポジトリをダウンロード

```
git clone https://github.com/FyukMdaa/mozcSKK.git --depth 1
cd mozcSKK\src
```

以降はディレクトリを変更せずにすべての操作を行うことができます。

### 追加のビルド依存関係をチェックアウト

```
python build_tools/update_deps.py
```

このステップで、追加のビルド依存関係がダウンロードされます。

  * [LLVM 19.1.7](https://github.com/llvm/llvm-project/releases/tag/llvmorg-19.1.7)
  * [Ninja 1.11.0](https://github.com/ninja-build/ninja/releases/download/v1.11.0/ninja-win.zip)
  * [Qt 6.8.0](https://download.qt.io/archive/qt/6.8/6.8.0/submodules/qtbase-everywhere-src-6.8.0.tar.xz)
  * [.NET tools](dotnet-tools.json)
  * [git submodules](.gitmodules)

これらのライブラリを手動でダウンロードする場合は、このステップをスキップできます。

## ビルド

### Qtのビルド

```
python build_tools/build_qt.py --release --confirm_license
```

Qtライセンスを手動で確認したい場合は、`--confirm_license`オプションを省略してください。

すでにQtのプリビルトバイナリをインストールしている場合は、このプロセスをスキップできます。

### Mozcのビルド

上記のステップでQtをビルドした場合、以下のコマンドでMozcインストーラーをビルドするのに十分です。

```
python build_mozc.py gyp
python build_mozc.py build -c Release package
```

Qt依存関係なしでMozcをビルドしたい場合は、以下のように`--noqt`オプションを指定します。`--noqt`を指定すると、`mozc_tool.exe`は何もしないモックバージョンとしてビルドされることに注意してください。

```
python build_mozc.py gyp --noqt
python build_mozc.py build -c Release package
```

独自のQtバイナリを使用してMozcをビルドしたい場合は、以下のように`--qtdir`オプションを指定します。

```
python build_mozc.py gyp --qtdir=<your Qt directory>
python build_mozc.py build -c Release package
```

デバッグ情報が必要な場合は、以下のようにデバッグバージョンのMozcをビルドできます。

```
python build_mozc.py build_tools/build_qt.py --release --debug --confirm_license
python build_mozc.py build -c Debug package
```

### インストーラー

リリースビルドのバイナリは`out_win\Release\Mozc64.msi`にあります。

### ツリーのクリーンアップ

ツリーをクリーンアップするには、以下を実行します。これにより、実行ファイルやオブジェクトファイル、生成されたソースファイル、プロジェクトファイルなどの中間ファイルが削除されます。

```
python build_mozc.py clean
```

## Mozcのインストール

64ビット環境では、以下のコマンドを実行するか、対応するファイルをダブルクリックします。

```
out_win\Release\Mozc64.msi
```

## Mozcのアンインストール

Win+Rを押してダイアログを開き、`ms-settings:appsfeatures-app`と入力するか、ターミナルで以下のコマンドを実行します。

```
start ms-settings:appsfeatures-app
```

その後、リストから`Mozc`をアンインストールします。

---

## ユニットテストの実行

以下のようにユニットテストを実行できます。

```
python build_mozc.py gyp --noqt
python build_mozc.py runtests -c Release
```

現在、Qtに依存するユニットテストはないため、GYPフェーズで`--noqt`の代わりに`--qtdir=`オプションを指定することもできます。