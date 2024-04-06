# macOS Proces Abuse

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>でゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい**または **HackTricks をPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* **💬** [**Discordグループ**](https://discord.gg/hRep4RUj7f)**または**[**telegramグループ**](https://t.me/peass)**に参加**するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)を**フォロー**する。
* **ハッキングトリックを共有するために、**[**HackTricks**](https://github.com/carlospolop/hacktricks)**と**[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)**のGitHubリポジトリにPRを提出してください。**

</details>

## MacOS プロセスの悪用

MacOSは、他のすべてのオペレーティングシステムと同様に、**プロセスが相互作用し、通信し、データを共有する**ためのさまざまな方法とメカニズムを提供しています。これらの技術はシステムの効率的な動作に不可欠ですが、脅威行為者が**悪意のある活動を行う**ためにも悪用される可能性があります。

### ライブラリインジェクション

ライブラリインジェクションは、攻撃者が**プロセスに悪意のあるライブラリを読み込ませる**技術です。インジェクトされると、そのライブラリはターゲットプロセスのコンテキストで実行され、攻撃者にプロセスと同じ権限とアクセス権を提供します。

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### 関数フック

関数フックは、ソフトウェアコード内の**関数呼び出し**やメッセージを**傍受**することを含みます。関数をフックすることで、攻撃者はプロセスの動作を**変更**したり、機密データを観察したり、実行フローを制御したりすることができます。

{% content-ref url="macos-function-hooking.md" %}
[macos-function-hooking.md](macos-function-hooking.md)
{% endcontent-ref %}

### プロセス間通信

プロセス間通信（IPC）は、異なるプロセスが**データを共有し交換する**さまざまな方法を指します。IPCは多くの正当なアプリケーションにとって基本的ですが、プロセスの分離を逆転させ、機密情報を漏洩させたり、不正なアクションを実行したりするために悪用されることがあります。

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Electronアプリケーションのインジェクション

特定の環境変数で実行されるElectronアプリケーションは、プロセスインジェクションの脆弱性がある可能性があります：

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Chromiumインジェクション

`--load-extension`および`--use-fake-ui-for-media-stream`フラグを使用して、**ブラウザ内の中間者攻撃**を実行し、キーストローク、トラフィック、クッキーを盗み、ページにスクリプトをインジェクトすることができます：

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### Dirty NIB

NIBファイルは、アプリケーション内の**ユーザーインターフェース（UI）要素**とその相互作用を定義します。ただし、これらは**任意のコマンドを実行**でき、**Gatekeeper**は、**NIBファイルが変更された場合**にすでに実行されているアプリケーションを実行を防ぎません。そのため、これらは任意のプログラムが任意のコマンドを実行するのに使用できます：

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Javaアプリケーションのインジェクション

特定のJava機能（**`_JAVA_OPTS`環境変数など）を悪用して、Javaアプリケーションが任意のコード/コマンドを実行**することができます。

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### .Netアプリケーションのインジェクション

macOSの保護機能（ランタイムハードニングなど）によって保護されていない\*\*.Netデバッグ機能を悪用\*\*することで、.Netアプリケーションにコードをインジェクトすることができます。

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Perlインジェクション

Perlスクリプトが任意のコードを実行するようにするさまざまなオプションをチェックしてください：

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Rubyインジェクション

Ruby環境変数を悪用して、任意のスクリプトが任意のコードを実行することができます：

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Pythonインジェクション

環境変数\*\*`PYTHONINSPECT`**が設定されている場合、PythonプロセスはPython CLIに移行します。対話セッションが終了すると、**`PYTHONSTARTUP`**を使用して対話セッションの開始時に実行するPythonスクリプトを指定することもできます。**\
**ただし、**`PYTHONSTARTUP`**スクリプトは、**`PYTHONINSPECT`\*\*が対話セッションを作成するときには実行されません。

\*\*`PYTHONPATH`**や**`PYTHONHOME`\*\*などの他の環境変数も、Pythonコマンドが任意のコードを実行するのに役立つ場合があります。

\*\*`pyinstaller`\*\*でコンパイルされた実行可能ファイルは、埋め込まれたPythonを使用していても、これらの環境変数を使用しません。

全体的に、環境変数を悪用してPythonが任意のコードを実行する方法を見つけることができませんでした。\ ただし、ほとんどの人は\*\*Hombrew\*\*を使用してPythonをインストールするため、デフォルトの管理者ユーザーの\*\*書き込み可能な場所\*\*にPythonがインストールされます。次のようにそれを乗っ取ることができます: \`\`\`bash mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old cat > /opt/homebrew/bin/python3 <

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) は、次のような**プロセスインジェクション**アクションを**検出およびブロック**するオープンソースアプリケーションです：

* **環境変数**の使用：次の環境変数の存在を監視します：**`DYLD_INSERT_LIBRARIES`**、**`CFNETWORK_LIBRARY_PATH`**、**`RAWCAMERA_BUNDLE_PATH`**、**`ELECTRON_RUN_AS_NODE`**
* **`task_for_pid`** の呼び出し：1つのプロセスが別のプロセスの**タスクポート**を取得しようとする場合を見つけ、プロセスにコードをインジェクトすることを可能にします。
* **Electronアプリのパラメータ**：誰かが\*\*`--inspect`**、**`--inspect-brk`**、**`--remote-debugging-port`\*\* のコマンドライン引数を使用してElectronアプリをデバッグモードで起動し、それにコードをインジェクトすることができます。
* **シンボリックリンク**または**ハードリンク**の使用：一般的に最も一般的な悪用方法は、**ユーザ権限でリンクを作成**し、それを**より高い権限**の場所に向けることです。ハードリンクとシンボリックリンクの検出は非常に簡単です。リンクを作成するプロセスがターゲットファイルよりも**異なる権限レベル**を持っている場合、**アラート**を作成します。残念ながら、シンボリックリンクの場合、ブロックは不可能です。リンクの宛先に関する情報が作成前にはわからないためです。これはAppleのEndpointSecuriyフレームワークの制限です。

### 他のプロセスによる呼び出し

[**このブログ投稿**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) では、**`task_name_for_pid`** 関数を使用して、他のプロセスがプロセスにコードをインジェクトすることに関する情報を取得し、その他のプロセスに関する情報を取得する方法について説明しています。

その関数を呼び出すには、プロセスを実行しているユーザーと**同じuid**である必要があります。または**root**である必要があります（情報を返すだけであり、コードをインジェクトする方法ではありません）。

## 参考文献

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)
