# Android用吉里吉里Zプラグインの作り方
android_plugins リポジトリは、Android用吉里吉里Zプラグインをビルドするためのベースプロジェクトを提供しています。  
このプロジェクトはコンパイルしてsoファイルを生成するためのもので、デバッグは行えません。  
プラグインのデバッグを行いたい場合はメインの吉里吉里Zプロジェクトにプラグインのプロジェクトをコピーして、ルートのCMakeList.txtに対象プラグインのディレクトリ位置を追記します(メインプロジェクトではtp_stub.h/.cppがAndroidプロジェクトルートにないのでコピーが必要です)。  
詳細については後述します。

## 制限事項
Android 版のプラグインでは、プラグイン側のV2Link/V2Unlinkのみをインポートし他の関数は使用できません。  
プラグイン名はlibで始まる必要があります。
また、名称はすべて小文字にする必要があります。
つまり、libxxx.soと言った名前になります。
libや.soの部分はWindows版と異なりますが、TJS2でXXX.dllのように指定しても、読み込み時に自動的にlibxxx.soとして読み込まれます(プレフィックスにlibがなければ追加され、.dllは.soに変換、大文字は小文字へ変換されます)。
これによって共通のTJS2スクリプトで同一のプラグインへアクセスできます。  
他に当たり前ですが、WindowsのAPI等は利用できません。

## Android対応に必要な作業
android_plugins 内の plugins 以下にビルドしたいプラグインのディレクトリを追加します。  
plugins/CMakeList.txt に追加したディレクトリを追記します。  
具体的には add_subdirectory( ディレクトリ名 ) を末尾に記述します。  
プラグインのソースコードは GitHub の別リポジトリに置いていたりするでしょうから、submoduleで追加すると良いと思います。  
kagparserプラグインをサンプルとして追加しているので、これを参考に以降説明します。

## Android対応に必要なファイル
以下の2つのファイルが新規に必要です。

### version-script
公開する関数等を記述するファイルです。
Windowsの.defファイルに当たります。
KAGParserプラグインリポジトリのversion_script.mapファイルが該当します。
内容は全く同じで良いので、単純にファイルをコピーします。

### CMakeList.txt
cmakeでのビルドに必要なファイルです。
ビルドするライブラリ名、インクルードディレクトリ、ソースコードファイルを記述します。
KAGParserプラグインリポジトリのCMakeList.txtをコピーして書き換えるのが手軽です。  
書き換えポイント  
include_directoriesの()内にインクルードディレクトリが列挙されているので、必要であれば、最後の)の前で改行しインクルードディレクトリを追記します。  
add_library()に対象のライブラリ名とソースコードを列挙します。kagparserとなっているところを対象のプラグイン名に、SHAREDの後の空行以降にソースコードファイル名を列挙します。../tp_stub.cppは共通と思われるので、その手前まで書き換えます。  
target_link_libraries()に対象のライブラリ名を記述します。ここにはadd_libraryで書いたものと同一の名前を記述します。  
基本的にはこの三点で問題ないと思われますが、デバッグオプションを追加したり、他のライブラリを参照する場合は、cmakeの使い方を検索したり吉里吉里本体のCMakeList.txtを参照してください。

## プラグインのビルド
android_plugins ディレクトリを Android Studio で開きます。  
Build Variant で release を選択します。  
他の Android アプリ同様にビルドします。  
android_plugins/app/build/intermediates/cmake/release/obj 以下にCPUごとのsoファイルが生成されます。

## プラグインのデバッグ
上述の方法は単純にプラグインをビルドして吉里吉里で利用するための方法です。  
Windows版で問題なく動作していて、環境に依存しないプラグインであればビルドしてそのまま利用できると思いますが、ソースコードデバッグしたい場合は、本体と共にビルドする必要があります。  
krkrz/android以下にplugins等の名前のフォルダを作り、その中にtp_stub.h/.cppをコピーします。  
plugins内にプラグインのソースコードをディレクトリごとコピーします。
ルートのCMakeList.txt内の下の方に「# external library」と書かれたところにadd_subdirectoryが列挙してあるので、その最下行にコピーしたフォルダ名add_subdirectory(android/plugins/xxx)等と追加します。  
プラグインではデバッグ情報を削っているので、削らないように修正します。  
具体的にはCMakeList.txtのset( CMAKE_SHARED_LINKER_FLAGS～と書かれた行の最後の方に「-Wl,--strip-debug -Wl,--discard-all」と書かれているところを削除します。  
吉里吉里Z本体をビルドすれば同時にプラグインのソースコードもビルドされるので、必要に応じてブレークポイントなどを貼ってデバッグします。

## その他ソースコード上の注意
Android対応に当たってソースコードで修正が必要になるであろう項目
* UTF-8化する。AndroidではソースコードがUTF-8である必要があるので、UTF-8に変換しWindows版でもビルドできるようにコンパイルオプションに/source-charset:utf-8 を追加する(Visual Studio2015)。
* V2Link/V2Unlinkのプロトタイプの変更。Windows版ではHRESULT _stdcall V2Link(iTVPFunctionExporter*)であったが、HRESULTはWindows固有であるため、tjs_errorへと変更されている。ifdefなどでtypedef tjs_error HRESULT;等する必要がある。\_\_stdcallや\_\_declspec(dllexport)も#defineで別名定義する。
* S_OK/E_FAILではなくTJS_S_OK/TJS_E_FAILを使う。S_OK等はWindows固有なので、TJSの定義の方を使うようにする。
* wstringやwchar_tを避けて、tjs_stringやtjs_charに変更する。wchar_tはサイズが環境依存のため、マルチプラットフォーム化に当たって同じサイズになるようにWindows以外ではu16stringとchar16_tが使われるようにしている。Windows版はWin32 APIへの引数などの関係から従来通りの定義になっている。環境固有とならないようにwstringやwchar_tは使わないようにする。
* IStreamでファイル読み込みしている箇所を書き換える。[IStreamからBinaryStreamへ](https://krkrz.github.io/android_plugins/istream.html)

# Android版でのプラグインの利用
プラグインのバイナリ(soファイル)が入ったディレクトリ(armeabi/armeabi-v7a/arm64-v8a など)ごと本体のプロジェクトのsoを入れるディレクトリ(krkrz/android/app/src/main/jniLibs)にコピーします。  
Build APK して apk を生成すれば、apk 内にプラグインが入ります。  
後はいつものようにTJS2スクリプトからPlugin.link(filename)すると利用できます。