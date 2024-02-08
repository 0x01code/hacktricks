# macOSライブラリインジェクション

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>でAWSハッキングをゼロからヒーローまで学ぶ</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

- **HackTricksで企業を宣伝したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を入手する
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションを見つける
- 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦で私をフォローする：[**@carlospolopm**](https://twitter.com/carlospolopm)。
- **ハッキングトリックを共有する**には、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。

</details>

{% hint style="danger" %}
**dyldのコードはオープンソース**であり、[https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/)で見つけることができ、**`https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz`**のような**URL**を使用して**tar**をダウンロードできます。
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

これは、[**LinuxのLD\_PRELOAD**](../../../../linux-hardening/privilege-escalation#ld\_preload)のようなものです。プロセスに特定のライブラリをパスからロードするよう指示することができます（環境変数が有効になっている場合）

このテクニックは、インストールされたすべてのアプリケーションに「Info.plist」というplistがあり、`LSEnvironmental`というキーを使用して**環境変数を割り当てる**ことができるため、**ASEPテクニックとしても使用できます**。

{% hint style="info" %}
2012年以降、**Appleは`DYLD_INSERT_LIBRARIES`の権限を大幅に削減**しています。

コードに移動して**`src/dyld.cpp`**を確認してください。関数**`pruneEnvironmentVariables`**では、**`DYLD_*`**変数が削除されていることがわかります。

関数**`processRestricted`**では、制限の理由が設定されています。そのコードを確認すると、理由は次のとおりです。

- バイナリが`setuid/setgid`である
- machoバイナリに`__RESTRICT/__restrict`セクションが存在する
- ソフトウェアに[`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables)権限がないハード化されたランタイムの権限
- バイナリの権限を確認するには、`codesign -dv --entitlements :- </path/to/bin>`を使用します

より新しいバージョンでは、このロジックを**`configureProcessRestrictions`**関数の後半に見つけることができます。ただし、新しいバージョンで実行されるのは、**関数の最初のチェック**です（iOSまたはシミュレーションに関連するif文はmacOSでは使用されないため、それらを削除できます）。
{% endhint %}

### ライブラリ検証

バイナリが**`DYLD_INSERT_LIBRARIES`**環境変数を使用することを許可していても、バイナリがライブラリの署名をチェックする場合、カスタムライブラリはロードされません。

カスタムライブラリをロードするには、バイナリに次のいずれかの権限が必要です。

- **`com.apple.security.cs.disable-library-validation`**
- **`com.apple.private.security.clear-library-validation`**

または、バイナリには**ハード化されたランタイムフラグ**または**ライブラリ検証フラグ**がない必要があります。

バイナリが**ハード化されたランタイム**を持っているかどうかは、`codesign --display --verbose <bin>`を使用して確認し、**`CodeDirectory`**内のランタイムフラグを確認します。例：**`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

また、バイナリと**同じ証明書で署名されている場合**もライブラリをロードできます。

これを悪用する例と制限事項を確認するには、次の場所を参照してください：

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dylibハイジャッキング

{% hint style="danger" %}
**以前のライブラリ検証の制限**も、Dylibハイジャッキング攻撃を実行するために適用されます。
{% endhint %}

Windowsと同様に、MacOSでも**dylibsをハイジャック**して、**アプリケーションが** **任意の** **コードを実行** **する** **ことができます**（実際には通常のユーザーでは`.app`バンドル内に書き込むためのTCC権限が必要かもしれません）。
ただし、**MacOS**アプリケーションがライブラリをロードする方法は、Windowsよりも**制限が多い**ことを意味します。これは、**マルウェア**開発者がこのテクニックを**ステルス**に使用できる可能性があるが、特権を昇格させるためにこれを悪用する可能性はずっと低いということを意味します。

まず、**MacOSバイナリがライブラリをロードする際にフルパスを指定する**ことが**一般的**です。そして、**MacOSは決して** **$PATH**のフォルダを検索しません。

この機能に関連する**コード**の**主要な**部分は、`ImageLoader.cpp`の**`ImageLoader::recursiveLoadLibraries`**にあります。

machoバイナリがライブラリをロードするために使用できる**4つの異なるヘッダーコマンド**があります：

- **`LC_LOAD_DYLIB`**コマンドは、dylibをロードする一般的なコマンドです。
- **`LC_LOAD_WEAK_DYLIB`**コマンドは、前のコマンドと同様に機能しますが、dylibが見つからない場合でもエラーなしで実行が続行されます。
- **`LC_REEXPORT_DYLIB`**コマンドは、異なるライブラリからシンボルをプロキシ（または再エクスポート）します。
- **`LC_LOAD_UPWARD_DYLIB`**コマンドは、お互いに依存する2つのライブラリがある場合に使用されます（これは_上向き依存性_と呼ばれます）。

ただし、**2種類のdylibハイジャッキング**があります：

- **欠落している弱リンクされたライブラリ**：これは、アプリケーションが存在しないライブラリを**LC\_LOAD\_WEAK\_DYLIB**で構成してロードしようとすることを意味します。その後、**攻撃者が期待される場所にdylibを配置すると、ロードされます**。
- リンクが「弱い」という事実は、ライブラリが見つからなくてもアプリケーションが実行を続行することを意味します。
- これに関連する**コード**は、`ImageLoaderMachO.cpp`の`ImageLoaderMachO::doGetDependentLibraries`関数にあり、`lib->required`が`LC_LOAD_WEAK_DYLIB`がtrueの場合にのみ`false`になります。
- バイナリで**弱リンクされたライブラリ**を見つけるには（後でハイジャックライブラリを作成する例があります）：
  ```bash
  otool -l </path/to/bin> | grep LC_LOAD_WEAK_DYLIB -A 5 cmd LC_LOAD_WEAK_DYLIB
  cmdsize 56
  name /var/tmp/lib/libUtl.1.dylib (offset 24)
  time stamp 2 Wed Jun 21 12:23:31 1969
  current version 1.0.0
  compatibility version 1.0.0
  ```
- **@rpath**で構成されたライブラリ：Mach-Oバイナリには**`LC_RPATH`**と**`LC_LOAD_DYLIB`**コマンドが含まれる場合があります。これらのコマンドの値に基づいて、**異なるディレクトリからライブラリがロードされます**。
- **`LC_RPATH`**には、バイナリでライブラリをロードするために使用されるいくつかのフォルダのパスが含まれています。
- **`LC_LOAD_DYLIB`**には、ロードする特定のライブラリへのパスが含まれています。これらのパスには**`@rpath`**が含まれる場合があり、**`LC_RPATH`**内の値で置換されます。**`LC_RPATH`**に複数のパスがある場合、すべてが使用されてライブラリを検索します。例：
  - **`LC_LOAD_DYLIB`**に`@rpath/library.dylib`が含まれ、**`LC_RPATH`**に`/application/app.app/Contents/Framework/v1/`と`/application/app.app/Contents/Framework/v2/`が含まれている場合。両方のフォルダが`library.dylib`をロードするために使用されます。ライブラリが`[...]/v1/`に存在しない場合、攻撃者はそこに配置して`[...]/v2/`のライブラリのロードをハイジャックできます。**`LC_LOAD_DYLIB`**内のパスの順序に従われます。
- バイナリで**rpathパスとライブラリ**を検索するには、`otool -l </path/to/binary> | grep -E "LC_RPATH|LC_LOAD_DYLIB" -A 5`を使用します。

{% hint style="info" %}
**`@executable_path`**：**メイン実行ファイルを含むディレクトリ**への**パス**です。

**`@loader_path`**：**Mach-Oバイナリ**を含む**ディレクトリ**への**パス**です。

- **実行可能ファイル**で使用される場合、**`@loader_path`**は**実質的に** **`@executable_path`**と**同じ**です。
- **dylib**で使用される場合、**`@loader_path`**は**dylib**への**パス**を提供します。
{% endhint %}

この機能を悪用して特権を昇格させる方法は、**root**で実行されている**アプリケーション**が**攻撃者が書き込み権限を持つフォルダ**で**ライブラリを探している**場合です。

{% hint style="success" %}
アプリケーション内の**欠落しているライブラリ**を見つけるための素敵な**スキャナー**は、[**Dylib Hijack Scanner**](https://objective-see.com/products/dhs.html)または[**CLIバージョン**](https://github.com/pandazheng/DylibHijack)です。
このテクニックに関する技術的な詳細を含む素敵な**レポート**は[**こちら**](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x)で見つけることができます。
{% endhint %}

**例**

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dlopenハイジャッキング

{% hint style="danger" %}
**以前のライブラリ検証の制限も**、Dlopenハイジャッキング攻撃を実行するために適用されます。
{% endhint %}

**`man dlopen`**から：

- パスに**スラッシュ文字が含まれていない場合**（つまり、単なるリーフ名である場合）、**dlopen()は検索を行います**。**`$DYLD_LIBRARY_PATH`**が起動時に設定されている場合、dyldはまずそのディレクトリで検索します。次に、呼び出し元のmach-oファイルまたはメイン実行可能ファイルが**`LC_RPATH`**を指定している場合、dyldは**そのディレクトリで検索します**。次に、プロセスが**制限されていない**場合、dyldは**現在の作業ディレクトリ**で検索します。最後に、古いバイナリの場合、dyldはいくつかのフォールバックを試みます。**`$DYLD_FALLBACK_LIBRARY_PATH`**が起動時に設定されている場合、dyldは**そのディレクト
```c
// gcc dlopentest.c -o dlopentest -Wl,-rpath,/tmp/test
#include <dlfcn.h>
#include <stdio.h>

int main(void)
{
void* handle;

fprintf("--- No slash ---\n");
handle = dlopen("just_name_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative framework ---\n");
handle = dlopen("a/framework/rel_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs framework ---\n");
handle = dlopen("/a/abs/framework/abs_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative Path ---\n");
handle = dlopen("a/folder/rel_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs Path ---\n");
handle = dlopen("/a/abs/folder/abs_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

return 0;
}
```
コンパイルして実行すると、**各ライブラリが見つからなかった場所**がわかります。また、**FSログをフィルタリング**することもできます。
```bash
sudo fs_usage | grep "dlopentest"
```
## 相対パスハイジャッキング

**特権バイナリ/アプリ**（たとえばSUIDや強力な権限を持つバイナリ）が**相対パス**ライブラリ（たとえば`@executable_path`や`@loader_path`を使用）をロードしており、かつ**ライブラリ検証が無効**になっている場合、バイナリを攻撃者が**相対パスでロードされたライブラリを変更**できる位置に移動し、そのプロセスにコードをインジェクトすることが可能になるかもしれません。

## `DYLD_*`および`LD_LIBRARY_PATH`環境変数の剪定

ファイル`dyld-dyld-832.7.1/src/dyld2.cpp`には、**`pruneEnvironmentVariables`** 関数があり、**`DYLD_`**で始まる環境変数と **`LD_LIBRARY_PATH=`** を削除します。

また、**suid**および**sgid**バイナリに対して、この関数は明示的に**`DYLD_FALLBACK_FRAMEWORK_PATH`**と**`DYLD_FALLBACK_LIBRARY_PATH`**を**null**に設定します。

この関数は、OSXをターゲットにしている場合、同じファイルの**`_main`** 関数から次のように呼び出されます：
```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```
そして、これらのブールフラグはコード内の同じファイルで設定されています：
```cpp
#if TARGET_OS_OSX
// support chrooting from old kernel
bool isRestricted = false;
bool libraryValidation = false;
// any processes with setuid or setgid bit set or with __RESTRICT segment is restricted
if ( issetugid() || hasRestrictedSegment(mainExecutableMH) ) {
isRestricted = true;
}
bool usingSIP = (csr_check(CSR_ALLOW_TASK_FOR_PID) != 0);
uint32_t flags;
if ( csops(0, CS_OPS_STATUS, &flags, sizeof(flags)) != -1 ) {
// On OS X CS_RESTRICT means the program was signed with entitlements
if ( ((flags & CS_RESTRICT) == CS_RESTRICT) && usingSIP ) {
isRestricted = true;
}
// Library Validation loosens searching but requires everything to be code signed
if ( flags & CS_REQUIRE_LV ) {
isRestricted = false;
libraryValidation = true;
}
}
gLinkContext.allowAtPaths                = !isRestricted;
gLinkContext.allowEnvVarsPrint           = !isRestricted;
gLinkContext.allowEnvVarsPath            = !isRestricted;
gLinkContext.allowEnvVarsSharedCache     = !libraryValidation || !usingSIP;
gLinkContext.allowClassicFallbackPaths   = !isRestricted;
gLinkContext.allowInsertFailures         = false;
gLinkContext.allowInterposing         	 = true;
```
これは、バイナリが**suid**または**sgid**であるか、ヘッダーに**RESTRICT**セグメントがあるか、**CS\_RESTRICT**フラグで署名されている場合、**`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`**がtrueであり、環境変数が削除されることを意味します。

CS\_REQUIRE\_LVがtrueの場合、変数は削除されませんが、ライブラリ検証は元のバイナリと同じ証明書を使用していることを確認します。

## 制限のチェック

### SUID & SGID
```bash
# Make it owned by root and suid
sudo chown root hello
sudo chmod +s hello
# Insert the library
DYLD_INSERT_LIBRARIES=inject.dylib ./hello

# Remove suid
sudo chmod -s hello
```
### セクション `__RESTRICT` とセグメント `__restrict`
```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```
### ハード化されたランタイム

Keychain で新しい証明書を作成し、それを使用してバイナリに署名します:

{% code overflow="wrap" %}
```bash
# Apply runtime proetction
codesign -s <cert-name> --option=runtime ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello #Library won't be injected

# Apply library validation
codesign -f -s <cert-name> --option=library ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed #Will throw an error because signature of binary and library aren't signed by same cert (signs must be from a valid Apple-signed developer certificate)

# Sign it
## If the signature is from an unverified developer the injection will still work
## If it's from a verified developer, it won't
codesign -f -s <cert-name> inject.dylib
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed

# Apply CS_RESTRICT protection
codesign -f -s <cert-name> --option=restrict hello-signed
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed # Won't work
```
{% endcode %}

{% hint style="danger" %}
`0x0(none)`フラグで署名されたバイナリがあっても、実行時に**`CS_RESTRICT`**フラグが動的に付与される可能性があるため、このテクニックはそれらには機能しません。

このフラグを持つprocをチェックするには、(ここで[**csops**](https://github.com/axelexic/CSOps)を取得してください):&#x20;
```bash
csops -status <pid>
```
そして、フラグ0x800が有効になっているかどうかを確認します。
{% endhint %}

## 参考文献
* [https://theevilbit.github.io/posts/dyld_insert_libraries_dylib_injection_in_macos_osx_deep_dive/](https://theevilbit.github.io/posts/dyld_insert_libraries_dylib_injection_in_macos_osx_deep_dive/)

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>を使って、ゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れる
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**telegramグループ**](https://t.me/peass)に**参加**したり、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)で**フォロー**する。
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出して、あなたのハッキングテクニックを共有してください。

</details>
