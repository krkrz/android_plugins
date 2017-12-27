# IStreamからBinaryStreamへ

従来はプラグインでバイナリファイルの読み込みを行う場合、TVPCreateIStream でファイルを開き、IStream で読み込みを行っていたが、IStream インターフェイスは、Windows の COM に依存しているため、マルチプラットフォーム (Android) 環境では使えない。  
Windows 環境ではそのまま使えるが、マルチプラットフォームで使えるようにするには iTJSBinaryStream を使用する。  
具体的には TVPCreateBinaryStreamInterfaceForRead/TVPCreateBinaryStreamInterfaceForWrite を使用してファイルを開き、iTJSBinaryStream でファイルの読み書きを行う。  
iTJSBinaryStream インターフェイスは、本体の tTJSBinaryStream クラスとほぼ同じ。  
ただ、終了時は Destruct() メソッドを呼び出して終了し、delete で削除しないようにする。  
Destruct メソッドによって本体側に delete させる。

テキストファイルの読み込みは従来通り TVPCreateTextStreamForRead/TVPCreateTextStreamForReadByEncoding が使用できる。  
テキストファイルの読み書きを行っていたプラグインで、上述のAPIを使用していたものはそのままマルチプラットフォームで使用できる。  
