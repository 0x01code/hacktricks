# macOS プロセス乱用

<details>

<summary><strong>AWSハッキングをゼロからヒーローまで学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksにあなたの会社を広告したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションをチェックする
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に**参加する**か、[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)を**フォローする**。
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングのコツを共有する。

</details>

## MacOS プロセス乱用

MacOSは、他のオペレーティングシステムと同様に、**プロセスが相互作用し、通信し、データを共有する**ためのさまざまな方法とメカニズムを提供しています。これらの技術はシステムの効率的な機能に不可欠ですが、脅威アクターによって**悪意のある活動を実行する**ために悪用されることもあります。

### ライブラリインジェクション

ライブラリインジェクションは、攻撃者が**プロセスに悪意のあるライブラリをロードさせる**技術です。一度注入されると、ライブラリはターゲットプロセスのコンテキストで実行され、攻撃者にプロセスと同じ権限とアクセスを提供します。

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### 関数フッキング

関数フッキングは、ソフトウェアコード内の**関数呼び出しやメッセージを傍受する**ことを含みます。関数をフックすることで、攻撃者はプロセスの**振る舞いを変更したり**、機密データを観察したり、実行フローを制御することさえできます。

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### プロセス間通信

プロセス間通信（IPC）は、別々のプロセスが**データを共有し交換する**さまざまな方法を指します。IPCは多くの正当なアプリケーションにとって基本的ですが、プロセスの分離を妨げたり、機密情報を漏洩させたり、許可されていないアクションを実行したりするために悪用されることもあります。

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Electronアプリケーションのインジェクション

特定の環境変数で実行されたElectronアプリケーションは、プロセスインジェクションに対して脆弱になる可能性があります：

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Dirty NIB

NIBファイルはアプリケーション内の**ユーザーインターフェース（UI）要素とその相互作用を定義します**。しかし、**任意のコマンドを実行する**ことができ、**Gatekeeperは**既に実行されたアプリケーションが**NIBファイルが変更された場合に実行されるのを止めません**。したがって、任意のプログラムに任意のコマンドを実行させるために使用される可能性があります：

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Javaアプリケーションのインジェクション

**`_JAVA_OPTS`** 環境変数のような特定のJavaの機能を悪用して、Javaアプリケーションに**任意のコード/コマンドを実行させる**ことが可能です。

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### .Netアプリケーションのインジェクション

**.Netのデバッグ機能を悪用する**ことにより、.Netアプリケーションにコードを注入することが可能です（macOSの保護機能、例えばランタイムハードニングによって保護されていません）。

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Perlインジェクション

Perlスクリプトが任意のコードを実行するさまざまなオプションをチェックしてください：

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Rubyインジェクション

Ruby環境変数を悪用して任意のスクリプトに任意のコードを実行させることも可能です：

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Pythonインジェクション

環境変数**`PYTHONINSPECT`**が設定されている場合、Pythonプロセスは終了後にPython CLIに入ります。また、**`PYTHONSTARTUP`**を使用して、対話型セッションの開始時に実行するPythonスクリプトを指定することも可能です。\
ただし、**`PYTHONINSPECT`**が対話型セッションを作成するときには、**`PYTHONSTARTUP`**スクリプトは実行されないことに注意してください。

**`PYTHONPATH`**や**`PYTHONHOME`**などの他の環境変数も、Pythonコマンドに任意のコードを実行させるのに役立つ可能性があります。

組み込みPythonを使用して実行されている場合でも、**`pyinstaller`**でコンパイルされた実行可能ファイルはこれらの環境変数を使用しません。

{% hint style="danger" %}
全体として、環境変数を悪用してPythonに任意のコードを実行させる方法は見つかりませんでした。\
しかし、ほとんどの人が**Hombrew**を使用してpyhtonをインストールしており、これはデフォルトの管理ユーザーにとって**書き込み可能な場所**にpyhtonをインストールします。次のような方法でハイジャックすることができます：
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
```
**root**もこのコードを実行するときにpythonを実行します。
{% endhint %}

## 検出

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) はオープンソースのアプリケーションで、**プロセスインジェクションの検出とブロック**ができます：

* **環境変数**を使用して：次の環境変数の存在を監視します：**`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`**, **`ELECTRON_RUN_AS_NODE`**
* **`task_for_pid`**の呼び出しを使用して：一つのプロセスが別のプロセスの**タスクポートを取得**しようとするのを見つけ、プロセスにコードをインジェクトすることができます。
* **Electronアプリのパラメータ**：**`--inspect`**, **`--inspect-brk`**, **`--remote-debugging-port`**のコマンドライン引数を使用してElectronアプリをデバッグモードで起動し、コードをインジェクトすることができます。
* **シンボリックリンク**または**ハードリンク**を使用して：通常、最も一般的な悪用は、**ユーザー権限でリンクを配置し**、それを**より高い権限の場所にポイントする**ことです。ハードリンクとシンボリックリンクの両方に対する検出は非常に簡単です。リンクを作成するプロセスがターゲットファイルと**異なる権限レベル**を持っている場合、**アラート**を作成します。残念ながらシンボリックリンクの場合、リンクの作成前に目的地に関する情報がないため、ブロックは不可能です。これはAppleのEndpointSecuriyフレームワークの制限です。

### 他のプロセスによる呼び出し

[**このブログ投稿**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html)では、**`task_name_for_pid`**関数を使用して、他の**プロセスがプロセスにコードをインジェクトする**情報を取得し、その他のプロセスに関する情報を取得する方法が見つかります。

その関数を呼び出すには、実行中のプロセスと同じuidであるか**root**である必要があります（プロセスに関する情報を返しますが、コードをインジェクトする方法ではありません）。

## 参考文献

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)でAWSハッキングをゼロからヒーローまで学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>こちら</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksに広告を掲載したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手してください。
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見してください。私たちの独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションです。
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**テレグラムグループ**](https://t.me/peass)に**参加する**か、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)で**フォロー**してください。
* **HackTricks**の[**githubリポジトリ**](https://github.com/carlospolop/hacktricks)や[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)にPRを提出して、あなたのハッキングのコツを共有してください。

</details>
```
