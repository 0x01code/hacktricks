# リバースエンジニアリングツールと基本的な手法

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **ハッキングのトリックを共有**するには、[**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks)と[**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud)にPRを提出してください。

</details>

## ImGuiベースのリバースエンジニアリングツール

ソフトウェア：

* ReverseKit: [https://github.com/zer0condition/ReverseKit](https://github.com/zer0condition/ReverseKit)

## Wasmデコンパイラ/ Watコンパイラ

オンライン：

* [https://webassembly.github.io/wabt/demo/wasm2wat/index.html](https://webassembly.github.io/wabt/demo/wasm2wat/index.html)を使用して、wasm（バイナリ）からwat（クリアテキスト）に**デコンパイル**します。
* [https://webassembly.github.io/wabt/demo/wat2wasm/](https://webassembly.github.io/wabt/demo/wat2wasm/)を使用して、watからwasmに**コンパイル**します。
* [https://wwwg.github.io/web-wasmdec/](https://wwwg.github.io/web-wasmdec/)を使用してデコンパイルすることもできます。

ソフトウェア：

* [https://www.pnfsoftware.com/jeb/demo](https://www.pnfsoftware.com/jeb/demo)
* [https://github.com/wwwg/wasmdec](https://github.com/wwwg/wasmdec)

## .Netデコンパイラ

### [dotPeek](https://www.jetbrains.com/decompiler/)

dotPeekは、**ライブラリ**（.dll）、**Windowsメタデータファイル**（.winmd）、および**実行可能ファイル**（.exe）など、複数の形式を**デコンパイル**および調査するデコンパイラです。デコンパイルされたアセンブリは、Visual Studioプロジェクト（.csproj）として保存することができます。

ここでの利点は、失われたソースコードを復元するために、古いアセンブリからの復元が必要な場合、このアクションが時間を節約できることです。さらに、dotPeekはデコンパイルされたコード全体で便利なナビゲーションを提供するため、**Xamarinアルゴリズム分析**に最適なツールの1つです。

### [.Net Reflector](https://www.red-gate.com/products/reflector/)

包括的なアドインモデルと、ツールを拡張して独自のニーズに合わせるAPIを備えた.NETリフレクタは、時間を節約し、開発を簡素化します。このツールが提供する逆向きエンジニアリングサービスの多様性を見てみましょう。

* データがライブラリやコンポーネントを介してどのように流れるかを示します。
* .NET言語とフレームワークの実装と使用方法を示します。
* 使用されているAPIとテクノロジーからより多くの情報を取得するために、ドキュメント化されていない機能や公開されていない機能を見つけます。
* 依存関係と異なるアセンブリを見つけます。
* コード、サードパーティのコンポーネント、およびライブラリのエラーの正確な位置を特定します。

### [ILSpy](https://github.com/icsharpcode/ILSpy) & [dnSpy](https://github.com/dnSpy/dnSpy/releases)

[Visual Studio Code用のILSpyプラグイン](https://github.com/icsharpcode/ilspy-vscode)：どのOSでも使用できます（VSCodeから直接インストールできます。gitをダウンロードする必要はありません。**Extensions**をクリックして**ILSpy**を検索します）。
**デコンパイル**、**変更**、**再コンパイル**が必要な場合は、[**https://github.com/0xd4d/dnSpy/releases**](https://github.com/0xd4d/dnSpy/releases)を使用できます（関数内の何かを変更するには、**右クリック->メソッドの変更**）。
[https://www.jetbrains.com/es-es/decompiler/](https://www.jetbrains.com/es-es/decompiler/)も試してみることができます。

### DNSpyログ

**DNSpyが情報をファイルに記録する**ようにするには、次の.Netの行を使用できます：
```bash
using System.IO;
path = "C:\\inetpub\\temp\\MyTest2.txt";
File.AppendAllText(path, "Password: " + password + "\n");
```
### DNSpy デバッグ

DNSpyを使用してコードをデバッグするには、次の手順を実行する必要があります：

まず、**デバッグに関連する** **アセンブリ属性**を変更します：

![](<../../.gitbook/assets/image (278).png>)
```aspnet
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints)]
```
reversing-tools-basic-methods/README.md

# Reversing Tools: Basic Methods

## Introduction

In the field of reverse engineering, having a good set of reversing tools is essential. These tools help in analyzing and understanding the inner workings of software, allowing us to uncover vulnerabilities, modify functionality, or simply gain a deeper understanding of how a program operates.

This guide will introduce you to some of the basic reversing tools commonly used by hackers and security professionals. By familiarizing yourself with these tools, you will be better equipped to analyze and manipulate software for various purposes.

## Contents

1. [IDA Pro](#ida-pro)
2. [OllyDbg](#ollydbg)
3. [Ghidra](#ghidra)
4. [Radare2](#radare2)
5. [x64dbg](#x64dbg)
6. [Hopper](#hopper)
7. [Binary Ninja](#binary-ninja)
8. [Conclusion](#conclusion)

## IDA Pro

IDA Pro is one of the most popular and powerful disassemblers/debuggers available. It supports a wide range of architectures and file formats, making it a versatile tool for reverse engineering. IDA Pro provides a user-friendly interface and a powerful analysis engine, allowing users to navigate through disassembled code, analyze functions, and identify vulnerabilities.

## OllyDbg

OllyDbg is a popular debugger used for analyzing and modifying binary code. It provides a dynamic analysis environment, allowing users to step through code, set breakpoints, and inspect registers and memory. OllyDbg also includes features for patching binaries, searching for specific instructions, and analyzing malware.

## Ghidra

Ghidra is a free and open-source software reverse engineering suite developed by the National Security Agency (NSA). It provides a wide range of features for analyzing and understanding binary code, including disassembly, decompilation, and scriptable analysis. Ghidra supports a variety of platforms and file formats, making it a powerful tool for reverse engineering.

## Radare2

Radare2 is a command-line based reverse engineering framework that supports a wide range of architectures and file formats. It provides a set of tools for disassembling, debugging, analyzing, and patching binary code. Radare2 is highly extensible and scriptable, allowing users to automate tasks and create custom analysis tools.

## x64dbg

x64dbg is a powerful open-source debugger for Windows that supports both x86 and x64 architectures. It provides a user-friendly interface and a wide range of features for analyzing and debugging binary code. x64dbg includes features such as dynamic analysis, breakpoints, memory dumping, and plugin support.

## Hopper

Hopper is a reverse engineering tool for macOS and Linux that supports a wide range of architectures and file formats. It provides a user-friendly interface and a powerful disassembly engine, allowing users to analyze and understand binary code. Hopper also includes features for debugging, patching binaries, and scripting.

## Binary Ninja

Binary Ninja is a commercial reverse engineering platform that supports a wide range of architectures and file formats. It provides a user-friendly interface and a powerful analysis engine, allowing users to analyze and manipulate binary code. Binary Ninja includes features such as advanced graph visualization, scripting, and plugin support.

## Conclusion

These are just a few of the many reversing tools available for analyzing and manipulating binary code. By familiarizing yourself with these tools and their capabilities, you will be better equipped to tackle reverse engineering challenges and uncover vulnerabilities in software. Remember to always use these tools responsibly and ethically, and to respect the laws and regulations governing their use. Happy reversing!

---

# リバースエンジニアリングツール：基本的な手法

## はじめに

リバースエンジニアリングの分野では、優れたリバースエンジニアリングツールのセットを持つことが重要です。これらのツールは、ソフトウェアの内部動作を分析し理解するのに役立ち、脆弱性を発見したり、機能を変更したり、プログラムの動作をより深く理解することができます。

このガイドでは、ハッカーやセキュリティ専門家がよく使用するいくつかの基本的なリバースエンジニアリングツールを紹介します。これらのツールに慣れることで、さまざまな目的でソフトウェアを分析し操作するための準備が整います。

## 目次

1. [IDA Pro](#ida-pro)
2. [OllyDbg](#ollydbg)
3. [Ghidra](#ghidra)
4. [Radare2](#radare2)
5. [x64dbg](#x64dbg)
6. [Hopper](#hopper)
7. [Binary Ninja](#binary-ninja)
8. [結論](#結論)

## IDA Pro

IDA Proは、最も人気のある強力なディスアセンブラ/デバッガの1つです。さまざまなアーキテクチャとファイル形式をサポートしており、リバースエンジニアリングのための多目的なツールです。IDA Proは、ユーザーフレンドリーなインターフェースと強力な解析エンジンを提供し、ユーザーは逆アセンブルされたコードをナビゲートし、関数を分析し、脆弱性を特定することができます。

## OllyDbg

OllyDbgは、バイナリコードの分析と修正に使用される人気のあるデバッガです。動的な分析環境を提供し、ユーザーはコードをステップ実行したり、ブレークポイントを設定したり、レジスタやメモリを調査したりすることができます。OllyDbgには、バイナリのパッチ適用、特定の命令の検索、マルウェアの分析などの機能も備わっています。

## Ghidra

Ghidraは、国家安全保障局（NSA）によって開発された無料のオープンソースのソフトウェアリバースエンジニアリングスイートです。ディスアセンブル、デコンパイル、スクリプト可能な解析など、バイナリコードの分析と理解のためのさまざまな機能を提供します。Ghidraは、さまざまなプラットフォームとファイル形式をサポートしており、リバースエンジニアリングのための強力なツールです。

## Radare2

Radare2は、コマンドラインベースのリバースエンジニアリングフレームワークで、さまざまなアーキテクチャとファイル形式をサポートしています。バイナリコードのディスアセンブル、デバッグ、分析、パッチ適用などのツールセットを提供します。Radare2は高度に拡張可能でスクリプト可能であり、ユーザーはタスクを自動化し、カスタムの解析ツールを作成することができます。

## x64dbg

x64dbgは、Windows用の強力なオープンソースデバッガで、x86およびx64アーキテクチャの両方をサポートしています。ユーザーフレンドリーなインターフェースとバイナリコードの分析とデバッグのためのさまざまな機能を提供します。x64dbgには、動的な分析、ブレークポイント、メモリのダンプ、プラグインサポートなどの機能が含まれています。

## Hopper

Hopperは、macOSとLinux向けのリバースエンジニアリングツールで、さまざまなアーキテクチャとファイル形式をサポートしています。ユーザーフレンドリーなインターフェースと強力なディスアセンブリエンジンを提供し、ユーザーはバイナリコードを分析し理解することができます。Hopperには、デバッグ、バイナリのパッチ適用、スクリプトなどの機能も備わっています。

## Binary Ninja

Binary Ninjaは、さまざまなアーキテクチャとファイル形式をサポートする商用リバースエンジニアリングプラットフォームです。ユーザーフレンドリーなインターフェースと強力な解析エンジンを提供し、ユーザーはバイナリコードを分析し操作することができます。Binary Ninjaには、高度なグラフ可視化、スクリプト、プラグインサポートなどの機能が含まれています。

## 結論

これらは、バイナリコードの分析と操作に使用できる多くのリバースエンジニアリングツールのうちのいくつかです。これらのツールとその機能に慣れることで、リバースエンジニアリングの課題に取り組み、ソフトウェアの脆弱性を発見するための準備が整います。これらのツールを常に責任を持って、倫理的に使用し、使用に関する法律と規制を尊重することを忘れないでください。リバースエンジニアリングを楽しんでください！
```
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.Default |
DebuggableAttribute.DebuggingModes.DisableOptimizations |
DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints |
DebuggableAttribute.DebuggingModes.EnableEditAndContinue)]
```
そして、**コンパイル**をクリックします：

![](<../../.gitbook/assets/image (314) (1) (1).png>)

次に、新しいファイルを _**ファイル >> モジュールを保存...**_ に保存します：

![](<../../.gitbook/assets/image (279).png>)

これは必要です。なぜなら、これを行わない場合、**ランタイム**でいくつかの**最適化**がコードに適用され、**デバッグ中にブレークポイントがヒットしない**か、いくつかの**変数が存在しない**可能性があるからです。

次に、.Netアプリケーションが**IIS**で実行されている場合、次のコマンドで**再起動**できます：
```
iisreset /noforce
```
次に、デバッグを開始するためには、すべての開いているファイルを閉じ、**デバッグタブ**で**プロセスにアタッチ**を選択します：

![](<../../.gitbook/assets/image (280).png>)

次に、**w3wp.exe**を選択して**IISサーバー**にアタッチし、**アタッチ**をクリックします：

![](<../../.gitbook/assets/image (281).png>)

プロセスのデバッグが開始されたので、それを停止してすべてのモジュールをロードします。まず、_デバッグ >> 中断_ をクリックし、次に _**デバッグ >> ウィンドウ >> モジュール**_ をクリックします：

![](<../../.gitbook/assets/image (286).png>)

![](<../../.gitbook/assets/image (283).png>)

**モジュール**の中の任意のモジュールをクリックし、**すべてのモジュールを開く**を選択します：

![](<../../.gitbook/assets/image (284).png>)

**アセンブリエクスプローラ**の中の任意のモジュールを右クリックし、**アセンブリをソート**をクリックします：

![](<../../.gitbook/assets/image (285).png>)

## Javaデコンパイラ

[https://github.com/skylot/jadx](https://github.com/skylot/jadx)\
[https://github.com/java-decompiler/jd-gui/releases](https://github.com/java-decompiler/jd-gui/releases)

## DLLのデバッグ

### IDAを使用する

* **rundll32をロード**します（64ビットはC:\Windows\System32\rundll32.exe、32ビットはC:\Windows\SysWOW64\rundll32.exe）。
* **Windbgデバッガ**を選択します。
* "**ライブラリのロード/アンロード時に中断**"を選択します。

![](<../../.gitbook/assets/image (135).png>)

* 実行の**パラメータ**を設定し、**DLLのパス**と呼び出したい関数を入力します：

![](<../../.gitbook/assets/image (136).png>)

その後、デバッグを開始すると、各DLLがロードされるたびに実行が停止します。rundll32がDLLをロードすると、実行が停止します。

しかし、ロードされたDLLのコードにどのようにアクセスできるのでしょうか？この方法では、方法がわかりません。

### x64dbg/x32dbgを使用する

* **rundll32をロード**します（64ビットはC:\Windows\System32\rundll32.exe、32ビットはC:\Windows\SysWOW64\rundll32.exe）。
* **コマンドラインを変更**します（ _ファイル --> コマンドラインの変更_ ）し、dllのパスと呼び出したい関数を設定します。例："C:\Windows\SysWOW64\rundll32.exe" "Z:\shared\Cybercamp\rev2\\\14.ridii\_2.dll",DLLMain
* _オプション --> 設定_ を変更し、"**DLLエントリ**"を選択します。
* それから**実行を開始**します。デバッガは各DLLメインで停止します。いずれかの時点で、自分のDLLのDLLエントリで停止します。そこから、ブレークポイントを設定したい場所を検索します。

win64dbgで実行が何らかの理由で停止されると、win64dbgウィンドウの上部にいる**どのコードにいるか**が表示されます：

![](<../../.gitbook/assets/image (137).png>)

そのため、デバッグしたいDLLで実行が停止した場所を確認できます。

## GUIアプリ/ビデオゲーム

[**Cheat Engine**](https://www.cheatengine.org/downloads.php)は、実行中のゲームのメモリ内に重要な値が保存されている場所を見つけて変更するための便利なプログラムです。詳細は以下を参照してください：

{% content-ref url="cheat-engine.md" %}
[cheat-engine.md](cheat-engine.md)
{% endcontent-ref %}

## ARM & MIPS

{% embed url="https://github.com/nongiach/arm_now" %}

## シェルコード

### blobrunnerを使用したシェルコードのデバッグ

[**Blobrunner**](https://github.com/OALabs/BlobRunner)は、シェルコードをメモリのスペースに割り当て、シェルコードが割り当てられたメモリアドレスを示し、実行を停止します。\
その後、プロセスにデバッガ（Idaまたはx64dbg）をアタッチし、指定されたメモリアドレスにブレークポイントを設定し、実行を再開します。これにより、シェルコードをデバッグできます。

リリースのGitHubページには、コンパイルされたリリースが含まれるzipファイルがあります：[https://github.com/OALabs/BlobRunner/releases/tag/v0.0.5](https://github.com/OALabs/BlobRunner/releases/tag/v0.0.5)\
以下のリンクに、Blobrunnerのわずかに変更されたバージョンがあります。コンパイルするには、Visual Studio CodeでC/C++プロジェクトを作成し、コードをコピーして貼り付け、ビルドします。

{% content-ref url="blobrunner.md" %}
[blobrunner.md](blobrunner.md)
{% endcontent-ref %}

### jmp2itを使用したシェルコードのデバッグ

[**jmp2it** ](https://github.com/adamkramer/jmp2it/releases/tag/v1.4)は、blobrunnerと非常に似ています。シェルコードをメモリのスペースに割り当て、**永遠のループ**を開始します。その後、デバッガをプロセスにアタッチし、**再生を開始し、2〜5秒待って停止**し、**永遠のループ**の中にいます。永遠のループの次の命令にジャンプすると、シェルコードを呼び出す命令になり、最終的にシェルコードを実行します。

![](<../../.gitbook/assets/image (397).png>)

[リリースページからjmp2itのコンパイル済みバージョンをダウンロードできます](https://github.com/adamkramer/jmp2it/releases/)。

### Cutterを使用したシェルコードのデバッグ

[**Cutter**](https://github.com/rizinorg/cutter/releases/tag/v1.12.0)は、radareのGUIです。Cutterを使用すると、シェルコードをエミュレートして動的に検査できます。

Cutterでは、「ファイルを開く」と「シェルコードを開く」の両方が可能です。私の場合、シェルコードをファイルとして開いた場合は正しく逆コンパイルされましたが、シェルコードとして開いた場合は逆コンパイルされませんでした：

![](<../../.gitbook/assets/image (400).png>)

特定の場所でエミュレーションを開始するには、そこにブレークポイントを設定し、おそらくCutterが自動的にそこからエミュレーションを開始します：

![](<../../.gitbook/assets/image (399).png>)

例えば、ヘックスダンプ内でスタックを表示できます：

![](<../../.gitbook/assets/image (402).png>)
### シェルコードの難読化を解除し、実行される関数を取得する

[**scdbg**](http://sandsprite.com/blogs/index.php?uid=7\&pid=152)を試してみるべきです。\
それは、シェルコードがどのような関数を使用しているか、またシェルコードがメモリ内で自己解読しているかを教えてくれます。
```bash
scdbg.exe -f shellcode # Get info
scdbg.exe -f shellcode -r #show analysis report at end of run
scdbg.exe -f shellcode -i -r #enable interactive hooks (file and network) and show analysis report at end of run
scdbg.exe -f shellcode -d #Dump decoded shellcode
scdbg.exe -f shellcode /findsc #Find offset where starts
scdbg.exe -f shellcode /foff 0x0000004D #Start the executing in that offset
```
scDbgには、グラフィカルなランチャーもあります。ここで、必要なオプションを選択してシェルコードを実行することができます。

![](<../../.gitbook/assets/image (398).png>)

**Create Dump**オプションは、シェルコードがメモリ内で動的に変更された場合に、最終的なシェルコードをダンプします（デコードされたシェルコードをダウンロードするのに便利です）。**start offset**は、特定のオフセットでシェルコードを開始するのに役立ちます。**Debug Shell**オプションは、scDbgターミナルを使用してシェルコードをデバッグするのに便利です（ただし、前述のオプションのいずれかを使用する方が、Idaやx64dbgを使用できるため、より良いと思います）。

### CyberChefを使用した逆アセンブル

シェルコードファイルをアップロードし、次のレシピを使用して逆アセンブルします：[https://gchq.github.io/CyberChef/#recipe=To\_Hex('Space',0)Disassemble\_x86('32','Full%20x86%20architecture',16,0,true,true)](https://gchq.github.io/CyberChef/#recipe=To\_Hex\('Space',0\)Disassemble\_x86\('32','Full%20x86%20architecture',16,0,true,true\))

## [Movfuscator](https://github.com/xoreaxeaxeax/movfuscator)

この難読化ツールは、すべての`mov`命令を変更します（本当にクールですね）。また、実行フローを変更するために割り込みも使用します。詳細については、以下を参照してください：

* [https://www.youtube.com/watch?v=2VF\_wPkiBJY](https://www.youtube.com/watch?v=2VF\_wPkiBJY)
* [https://github.com/xoreaxeaxeax/movfuscator/blob/master/slides/domas\_2015\_the\_movfuscator.pdf](https://github.com/xoreaxeaxeax/movfuscator/blob/master/slides/domas\_2015\_the\_movfuscator.pdf)

運が良ければ、[demovfuscator](https://github.com/kirschju/demovfuscator)がバイナリを復号化します。いくつかの依存関係があります。
```
apt-get install libcapstone-dev
apt-get install libz3-dev
```
そして、[keystoneをインストール](https://github.com/keystone-engine/keystone/blob/master/docs/COMPILE-NIX.md)します（`apt-get install cmake; mkdir build; cd build; ../make-share.sh; make install`）

もしCTFをプレイしている場合、フラグを見つけるためのこの回避策は非常に役立つでしょう：[https://dustri.org/b/defeating-the-recons-movfuscator-crackme.html](https://dustri.org/b/defeating-the-recons-movfuscator-crackme.html)

## Rust

**エントリーポイント**を見つけるには、次のように`::main`で関数を検索します：

![](<../../.gitbook/assets/image (612).png>)

この場合、バイナリの名前はauthenticatorと呼ばれているため、これが興味深いmain関数であることは明らかです。\
呼び出されている関数の**名前**を持っている場合、それらを**インターネット**で検索して、それらの**入力**と**出力**について学びます。

## **Delphi**

Delphiでコンパイルされたバイナリには、[https://github.com/crypto2011/IDR](https://github.com/crypto2011/IDR)を使用できます

Delphiバイナリをリバースエンジニアリングする場合は、IDAプラグイン[https://github.com/Coldzer0/IDA-For-Delphi](https://github.com/Coldzer0/IDA-For-Delphi)を使用することをお勧めします

単に**ATL+f7**（IDAでPythonプラグインをインポート）を押し、Pythonプラグインを選択します。

このプラグインはバイナリを実行し、デバッグの開始時に関数名を動的に解決します。デバッグを開始した後、再びStartボタン（緑色のボタンまたはf9）を押すと、実際のコードの最初にブレークポイントがヒットします。

また、グラフィックアプリケーションでボタンを押すと、デバッガはそのボタンによって実行される関数で停止します。

## Golang

Golangバイナリをリバースエンジニアリングする場合は、IDAプラグイン[https://github.com/sibears/IDAGolangHelper](https://github.com/sibears/IDAGolangHelper)を使用することをお勧めします

単に**ATL+f7**（IDAでPythonプラグインをインポート）を押し、Pythonプラグインを選択します。

これにより、関数の名前が解決されます。

## コンパイルされたPython

このページでは、ELF/EXE形式のPythonコンパイルバイナリからPythonコードを取得する方法が説明されています：

{% content-ref url="../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md" %}
[.pyc.md](../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md)
{% endcontent-ref %}

## GBA - ゲームボーイアドバンス

GBAゲームの**バイナリ**を取得した場合、さまざまなツールを使用して**エミュレート**および**デバッグ**することができます：

* [**no$gba**](https://problemkaputt.de/gba.htm)（デバッグバージョンをダウンロード）- インターフェース付きのデバッガが含まれています
* [**mgba** ](https://mgba.io)- CLIデバッガが含まれています
* [**gba-ghidra-loader**](https://github.com/pudii/gba-ghidra-loader) - Ghidraプラグイン
* [**GhidraGBA**](https://github.com/SiD3W4y/GhidraGBA) - Ghidraプラグイン

[**no$gba**](https://problemkaputt.de/gba.htm)では、_**Options --> Emulation Setup --> Controls**_\*\* \*\*でGame Boy Advanceの**ボタン**を押す方法が確認できます

![](<../../.gitbook/assets/image (578).png>)

押されたとき、各**キーには値**があり、それを識別することができます：
```
A = 1
B = 2
SELECT = 4
START = 8
RIGHT = 16
LEFT = 32
UP = 64
DOWN = 128
R = 256
L = 256
```
そうですね、この種のプログラムでは、プログラムがユーザーの入力をどのように処理するかが興味深い部分です。アドレス**0x4000130**には、一般的に見られる関数**KEYINPUT**があります。

![](<../../.gitbook/assets/image (579).png>)

前の画像では、関数が**FUN\_080015a8**から呼び出されていることがわかります（アドレス：_0x080015fa_と_0x080017ac_）。

その関数では、いくつかの初期化操作（重要ではない）の後に、
```c
void FUN_080015a8(void)

{
ushort uVar1;
undefined4 uVar2;
undefined4 uVar3;
ushort uVar4;
int iVar5;
ushort *puVar6;
undefined *local_2c;

DISPCNT = 0x1140;
FUN_08000a74();
FUN_08000ce4(1);
DISPCNT = 0x404;
FUN_08000dd0(&DAT_02009584,0x6000000,&DAT_030000dc);
FUN_08000354(&DAT_030000dc,0x3c);
uVar4 = DAT_030004d8;
```
このコードが見つかりました：
```c
do {
DAT_030004da = uVar4; //This is the last key pressed
DAT_030004d8 = KEYINPUT | 0xfc00;
puVar6 = &DAT_0200b03c;
uVar4 = DAT_030004d8;
do {
uVar2 = DAT_030004dc;
uVar1 = *puVar6;
if ((uVar1 & DAT_030004da & ~uVar4) != 0) {
```
最後のif文は、**`uVar4`**が**最後のキー**にあるかどうかをチェックし、現在のキーではないことを確認しています（現在のキーは**`uVar1`**に格納されています）。
```c
if (uVar1 == 4) {
DAT_030000d4 = 0;
uVar3 = FUN_08001c24(DAT_030004dc);
FUN_08001868(uVar2,0,uVar3);
DAT_05000000 = 0x1483;
FUN_08001844(&DAT_0200ba18);
FUN_08001844(&DAT_0200ba20,&DAT_0200ba40);
DAT_030000d8 = 0;
uVar4 = DAT_030004d8;
}
else {
if (uVar1 == 8) {
if (DAT_030000d8 == 0xf3) {
DISPCNT = 0x404;
FUN_08000dd0(&DAT_02008aac,0x6000000,&DAT_030000dc);
FUN_08000354(&DAT_030000dc,0x3c);
uVar4 = DAT_030004d8;
}
}
else {
if (DAT_030000d4 < 8) {
DAT_030000d4 = DAT_030000d4 + 1;
FUN_08000864();
if (uVar1 == 0x10) {
DAT_030000d8 = DAT_030000d8 + 0x3a;
```
前のコードでは、**uVar1**（**押されたボタンの値**が格納される場所）といくつかの値を比較しています：

* 最初に、**値4**（**SELECT**ボタン）と比較されます：このボタンはチャレンジでは画面をクリアします。
* 次に、**値8**（**START**ボタン）と比較されます：このボタンはコードがフラグを取得するために有効かどうかをチェックします。
* この場合、変数**`DAT_030000d8`**は0xf3と比較され、値が同じであればいくつかのコードが実行されます。
* それ以外の場合、いくつかのcont（`DAT_030000d4`）がチェックされます。これはcontであるため、コードに入った直後に1が追加されます。\
8未満の場合、**`DAT_030000d8`**に値を**追加**する何かが行われます（基本的には、contが8未満の間、押されたキーの値をこの変数に追加しています）。

したがって、このチャレンジでは、ボタンの値を知っている場合、**長さが8未満で、結果の加算が0xf3になる組み合わせを押す必要があります**。

**このチュートリアルの参考資料：** [**https://exp.codes/Nostalgia/**](https://exp.codes/Nostalgia/)

## ゲームボーイ

{% embed url="https://www.youtube.com/watch?v=VVbRe7wr3G4" %}

## コース

* [https://github.com/0xZ0F/Z0FCourse\_ReverseEngineering](https://github.com/0xZ0F/Z0FCourse\_ReverseEngineering)
* [https://github.com/malrev/ABD](https://github.com/malrev/ABD)（バイナリの逆コンパイル）

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか？** HackTricksで**会社を宣伝**したいですか？または、**最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見しましょう、私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクション
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に**参加**するか、**Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **ハッキングのトリックを共有するには、**[**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出**してください。

</details>
