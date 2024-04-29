# macOS库注入

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

- 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
- 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
- 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[NFTs](https://opensea.io/collection/the-peass-family)收藏品
- **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
- 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

{% hint style="danger" %}
**dyld的代码是开源的**，可以在[https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/)找到，也可以使用类似[https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)的URL下载tar文件。
{% endhint %}

## **Dyld进程**

查看Dyld如何在二进制文件中加载库：

{% content-ref url="macos-dyld-process.md" %}
[macos-dyld-process.md](macos-dyld-process.md)
{% endcontent-ref %}

## **DYLD\_INSERT\_LIBRARIES**

这类似于[**Linux上的LD\_PRELOAD**](../../../../linux-hardening/privilege-escalation/#ld\_preload)。它允许指示一个进程将从路径加载特定库（如果启用了环境变量）。

这种技术也可以**用作ASEP技术**，因为每个安装的应用程序都有一个名为"Info.plist"的属性列表，允许使用名为`LSEnvironmental`的键**分配环境变量**。

{% hint style="info" %}
自2012年以来，**苹果大大减少了** **`DYLD_INSERT_LIBRARIES`** 的权限。

转到代码并**检查`src/dyld.cpp`**。在函数**`pruneEnvironmentVariables`**中，您可以看到**删除了`DYLD_*`**变量。

在函数**`processRestricted`**中设置了限制的原因。检查该代码，您会看到限制的原因是：

- 二进制文件是`setuid/setgid`
- 在macho二进制文件中存在`__RESTRICT/__restrict`部分。
- 软件具有没有[`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables)授权的强化运行时
- 使用以下命令检查二进制文件的**授权**：`codesign -dv --entitlements :- </path/to/bin>`

在更新的版本中，您可以在函数**`configureProcessRestrictions`**的第二部分找到此逻辑。但是，在较新版本中执行的是函数的**开始检查**（您可以删除与iOS或模拟相关的if，因为这些在macOS中不会使用）。
{% endhint %}

### 库验证

即使二进制文件允许使用**`DYLD_INSERT_LIBRARIES`**环境变量，如果二进制文件检查要加载的库的签名，它将不会加载自定义内容。

为了加载自定义库，二进制文件需要具有以下授权之一：

- [`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
- [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

或者二进制文件**不应该**具有**强化运行时标志**或**库验证标志**。

您可以使用`codesign --display --verbose <bin>`检查二进制文件是否具有**强化运行时**，检查**`CodeDirectory`**中的标志运行时，例如：**`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

如果**使用与二进制文件相同的证书签名**，也可以加载库。

找到一个关于如何（滥用）利用此功能并检查限制的示例：

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dylib劫持

{% hint style="danger" %}
请记住**以前的库验证限制也适用**于执行Dylib劫持攻击。
{% endhint %}

与Windows一样，在MacOS中，您也可以**劫持dylibs**以使**应用程序**执行**任意** **代码**（实际上，从普通用户这样做可能不可能，因为您可能需要TCC权限才能写入`.app`包并劫持库）。\
然而，**MacOS**应用程序**加载**库的方式**比Windows更受限制**。这意味着**恶意软件**开发人员仍然可以使用此技术进行**隐蔽**，但是**滥用此技术以提升权限的可能性要低得多**。

首先，**更常见**的是发现**MacOS二进制文件指示库的完整路径**。其次，**MacOS从不在** **$PATH** **文件夹中搜索**库。

与此功能相关的**主要**代码部分位于`ImageLoader.cpp`中的**`ImageLoader::recursiveLoadLibraries`**中。

Macho二进制文件可以使用**4个不同的头部命令**来加载库：

- **`LC_LOAD_DYLIB`**命令是加载dylib的常见命令。
- **`LC_LOAD_WEAK_DYLIB`**命令与前一个命令类似，但如果未找到dylib，则继续执行而不会出现任何错误。
- **`LC_REEXPORT_DYLIB`**命令代理（或重新导出）来自不同库的符号。
- **`LC_LOAD_UPWARD_DYLIB`**命令在两个库彼此依赖时使用（这称为_向上依赖_）。

然而，有**2种dylib劫持**：

- **缺失的弱链接库**：这意味着应用程序将尝试加载一个使用**LC\_LOAD\_WEAK\_DYLIB**配置的不存在的库。然后，**如果攻击者将dylib放在预期的位置，它将被加载**。
- 链接是“弱”的意思是即使未找到库，应用程序也将继续运行。
- 与此相关的**代码**位于`ImageLoaderMachO.cpp`的`ImageLoaderMachO::doGetDependentLibraries`函数中，其中`lib->required`仅在`LC_LOAD_WEAK_DYLIB`为true时为`false`。
- 在二进制文件中查找**弱链接库**（稍后您将看到如何创建劫持库的示例）：
  ```bash
  otool -l </path/to/bin> | grep LC_LOAD_WEAK_DYLIB -A 5 cmd LC_LOAD_WEAK_DYLIB
  cmdsize 56
  name /var/tmp/lib/libUtl.1.dylib (offset 24)
  time stamp 2 Wed Jun 21 12:23:31 1969
  current version 1.0.0
  compatibility version 1.0.0
  ```
- **配置为@rpath**：Mach-O二进制文件可以具有**`LC_RPATH`**和**`LC_LOAD_DYLIB`**命令。根据这些命令的**值**，库将从**不同目录**加载。
* **`LC_LOAD_DYLIB`** 包含要加载的特定库的路径。这些路径可以包含 **`@rpath`**，它将被 **`LC_RPATH`** 中的值 **替换**。如果 **`LC_RPATH`** 中有多个路径，则每个路径都将用于搜索要加载的库。例如：
* 如果 **`LC_LOAD_DYLIB`** 包含 `@rpath/library.dylib`，而 **`LC_RPATH`** 包含 `/application/app.app/Contents/Framework/v1/` 和 `/application/app.app/Contents/Framework/v2/`。那么这两个文件夹都将用于加载 `library.dylib`。如果库在 `[...]/v1/` 中不存在，攻击者可以将其放在那里以劫持在 `[...]/v2/` 中的库加载，因为会按照 **`LC_LOAD_DYLIB`** 中路径的顺序进行加载。
* 使用以下命令在二进制文件中 **查找 rpath 路径和库**：`otool -l </path/to/binary> | grep -E "LC_RPATH|LC_LOAD_DYLIB" -A 5`

{% hint style="info" %}
**`@executable_path`**：是包含 **主可执行文件** 的 **目录路径**。

**`@loader_path`**：是包含包含加载命令的 **Mach-O 二进制文件** 的 **目录路径**。

* 当在可执行文件中使用时，**`@loader_path`** 实际上与 **`@executable_path`** **相同**。
* 当在 **dylib** 中使用时，**`@loader_path`** 给出了 **dylib** 的 **路径**。
{% endhint %}

利用这种功能进行 **权限提升** 的方式是在罕见情况下，由 **root** 执行的 **应用程序** 正在 **查找** 一些 **库**，而攻击者具有写权限的某个文件夹中存在该库。

{% hint style="success" %}
一个很好的 **扫描工具**，用于查找应用程序中的 **缺失库** 是 [**Dylib Hijack Scanner**](https://objective-see.com/products/dhs.html) 或 [**CLI 版本**](https://github.com/pandazheng/DylibHijack)。
关于这种技术的一个带有技术细节的不错的 **报告** 可以在 [**这里**](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x) 找到。
{% endhint %}

**示例**

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dlopen 劫持

{% hint style="danger" %}
请记住，执行 Dlopen 劫持攻击时也适用 **先前的库验证限制**。
{% endhint %}

来自 **`man dlopen`**：

* 当路径 **不包含斜杠字符**（即只是一个叶子名称）时，**dlopen() 将进行搜索**。如果在启动时设置了 **`$DYLD_LIBRARY_PATH`**，dyld 将首先在该目录中查找。接下来，如果调用的 mach-o 文件或主可执行文件指定了 **`LC_RPATH`**，那么 dyld 将在这些目录中查找。接下来，如果进程是 **不受限制的**，dyld 将在 **当前工作目录** 中搜索。最后，对于旧的二进制文件，dyld 将尝试一些回退。如果在启动时设置了 **`$DYLD_FALLBACK_LIBRARY_PATH`**，dyld 将在 **这些目录中搜索**，否则，dyld 将在 **`/usr/local/lib/`** 中查找（如果进程是不受限制的），然后在 **`/usr/lib/`** 中查找（此信息取自 **`man dlopen`**）。
1. `$DYLD_LIBRARY_PATH`
2. `LC_RPATH`
3. `CWD`（如果不受限制）
4. `$DYLD_FALLBACK_LIBRARY_PATH`
5. `/usr/local/lib/`（如果不受限制）
6. `/usr/lib/`

{% hint style="danger" %}
如果名称中没有斜杠，有两种方法可以进行劫持：

* 如果任何 **`LC_RPATH`** 是 **可写的**（但会检查签名，因此您还需要二进制文件是不受限制的）
* 如果二进制文件是 **不受限制的**，那么可以从 CWD 中加载内容（或滥用其中提到的环境变量之一）
{% endhint %}

* 当路径 **看起来像一个框架** 路径（例如 `/stuff/foo.framework/foo`），如果在启动时设置了 **`$DYLD_FRAMEWORK_PATH`**，dyld 将首先在该目录中查找 **框架部分路径**（例如 `foo.framework/foo`）。接下来，dyld 将尝试使用 **提供的路径**（对于相对路径，使用当前工作目录）。最后，对于旧的二进制文件，dyld 将尝试一些回退。如果在启动时设置了 **`$DYLD_FALLBACK_FRAMEWORK_PATH`**，dyld 将搜索这些目录。否则，它将搜索 **`/Library/Frameworks`**（在 macOS 上，如果进程是不受限制的），然后在 **`/System/Library/Frameworks`** 中搜索。
1. `$DYLD_FRAMEWORK_PATH`
2. 提供的路径（对于相对路径，如果不受限制，使用当前工作目录）
3. `$DYLD_FALLBACK_FRAMEWORK_PATH`
4. `/Library/Frameworks`（如果不受限制）
5. `/System/Library/Frameworks`

{% hint style="danger" %}
如果是框架路径，劫持的方式是：

* 如果进程是 **不受限制的**，可以滥用从 CWD 开始的 **相对路径** 和提到的环境变量（即使在文档中没有提到进程是否受限制，DYLD\_\* 环境变量会被移除）
{% endhint %}

* 当路径 **包含斜杠但不是框架路径**（即完整路径或指向 dylib 的部分路径），dlopen() 首先在（如果设置了） **`$DYLD_LIBRARY_PATH`** 中查找（使用路径的叶子部分）。接下来，dyld 将尝试使用 **提供的路径**（对于相对路径，仅对于不受限制的进程使用当前工作目录）。最后，对于旧的二进制文件，dyld 将尝试一些回退。如果在启动时设置了 **`$DYLD_FALLBACK_LIBRARY_PATH`**，dyld 将在这些目录中搜索，否则，dyld 将在 **`/usr/local/lib/`** 中查找（如果进程是不受限制的），然后在 **`/usr/lib/`** 中查找。
1. `$DYLD_LIBRARY_PATH`
2. 提供的路径（对于相对路径，如果不受限制，使用当前工作目录）
3. `$DYLD_FALLBACK_LIBRARY_PATH`
4. `/usr/local/lib/`（如果不受限制）
5. `/usr/lib/`

{% hint style="danger" %}
如果名称中有斜杠而不是框架，劫持的方式是：

* 如果二进制文件是 **不受限制的**，那么可以从 CWD 或 `/usr/local/lib` 中加载内容（或滥用其中提到的环境变量之一）
{% endhint %}

{% hint style="info" %}
注意：没有配置文件来 **控制 dlopen 搜索**。

注意：如果主可执行文件是 **set\[ug]id 二进制文件或使用授权签名**，则会 **忽略所有环境变量**，只能使用完整路径（[查看 DYLD\_INSERT\_LIBRARIES 限制](macos-dyld-hijacking-and-dyld\_insert\_libraries.md#check-dyld\_insert\_librery-restrictions) 获取更详细信息）

注意：Apple 平台使用 "universal" 文件来组合 32 位和 64 位库。这意味着没有 **单独的 32 位和 64 位搜索路径**。

注意：在 Apple 平台上，大多数 OS dylibs 都 **合并到 dyld 缓存** 中，不存在于磁盘上。因此，调用 **`stat()`** 来预先检查 OS dylib 是否存在 **不起作用**。但是，**`dlopen_preflight()`** 使用与 **`dlopen()`** 相同的步骤来查找兼容的 mach-o 文件。
{% endhint %}

**检查路径**

让我们使用以下代码检查所有选项：
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
如果您编译并执行它，您可以看到**每个库未成功搜索的位置**。此外，您可以**过滤FS日志**：
```bash
sudo fs_usage | grep "dlopentest"
```
## 相对路径劫持

如果一个**特权二进制应用程序**（比如一个SUID或一些拥有强大权限的二进制文件）正在**加载一个相对路径**库（例如使用`@executable_path`或`@loader_path`），并且**禁用了库验证**，那么可能会将二进制文件移动到攻击者可以**修改相对路径加载的库**的位置，并滥用它来向进程注入代码。

## 修剪 `DYLD_*` 和 `LD_LIBRARY_PATH` 环境变量

在文件 `dyld-dyld-832.7.1/src/dyld2.cpp` 中，可以找到函数**`pruneEnvironmentVariables`**，它将删除任何以`DYLD_`开头和`LD_LIBRARY_PATH=`的环境变量。

它还会将**`DYLD_FALLBACK_FRAMEWORK_PATH`**和**`DYLD_FALLBACK_LIBRARY_PATH`**这两个环境变量对于**suid**和**sgid**二进制文件设置为**null**。

如果针对OSX，可以从同一文件的**`_main`**函数中调用此函数：
```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```
并且这些布尔标志在代码中的同一文件中设置：
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
这基本上意味着，如果二进制文件是**suid**或**sgid**，或者在标头中有一个**RESTRICT**段，或者使用**CS\_RESTRICT**标志进行签名，那么**`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`**为真，环境变量将被修剪。

请注意，如果CS\_REQUIRE\_LV为真，则变量不会被修剪，但库验证将检查它们是否使用与原始二进制文件相同的证书。

## 检查限制

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
### 区块 `__RESTRICT` 与段 `__restrict`
```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```
### 强化运行时

在钥匙串中创建一个新证书，并使用它来签署二进制文件：

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
请注意，即使有用标志**`0x0(none)`**签名的二进制文件，当执行时也可以动态地获得**`CS_RESTRICT`**标志，因此这种技术在其中不起作用。

您可以使用以下命令检查进程是否具有此标志（获取[**csops here**](https://github.com/axelexic/CSOps)）:
```bash
csops -status <pid>
```
## 参考资料

* [https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/](https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/)
* [**\*OS Internals, Volume I: User Mode. By Jonathan Levin**](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 发现[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
