WindowsでMozcをビルドする方法

## 概要

以下のコマンドが何をするのか分からない場合は、説明を確認し、実行前に操作を確認してください。

```
python -m pip install six requests

git clone https://github.com/google/mozc.git --depth 1
cd mozcSKK\src

python build_tools/update_deps.py
python build_tools/build_qt.py --release --confirm_license
python build_mozc.py gyp
python build_mozc.py build -c Release package

# Mozcをインストール
out_win\Release\Mozc64.msi
```

ヒント: Bazelを使用してMozcをビルドすることもできます（実験的）。詳細は以下を参照してください。

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

BazelでMozcをビルドするための追加要件については、以下を参照してください。

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
  * [.NET tools](FyukMdaa/mozcSKK/dotnet-tools.json)
  * [git submodules](FyukMdaa/mozcSKK/.gitmodules)

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

---

## Bazelでビルド（実験的）

追加要件:

* [Bazelisk](https://github.com/bazelbuild/bazelisk)
  * Bazeliskは、特定のバージョンのBazelを使用するための[Bazel](https://bazel.build)のラッパーです。
  * [src/.bazeliskrc](../src/.bazeliskrc)で使用するBazelのバージョンを制御します。
* [MSYS2](https://github.com/msys2/msys2)

`build_tools/update_deps.py`と`build_tools/build_qt.py`を実行した後、`build_mozc.py`の代わりに以下のコマンドを実行します。

```
bazelisk --bazelrc=windows.bazelrc build --config oss_windows --config release_build package
```

リリースビルドのバイナリは`bazel-bin\win32\installer\Mozc64.msi`にあります。

### Bazelセットアップのヒント

* Mozcのためだけに新しいJDKをインストールする必要はありません。
* [Scoop](https://scoop.sh)を使用してBazelをインストールした場合、MSYS2もScoop経由でインストールすることをお勧めします。

https://bazel.build/install/windows?hl=ja#install-compilers