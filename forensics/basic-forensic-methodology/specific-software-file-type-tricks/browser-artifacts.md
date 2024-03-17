# 浏览器遗留物

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 发现[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
使用[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)可以轻松构建和**自动化工作流程**，使用世界上**最先进**的社区工具。\
立即获取访问权限：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## 浏览器遗留物 <a href="#id-3def" id="id-3def"></a>

浏览器遗留物包括由网络浏览器存储的各种类型数据，例如浏览历史记录、书签和缓存数据。这些遗留物存储在操作系统中的特定文件夹中，不同浏览器的位置和名称各不相同，但通常存储类似的数据类型。

以下是最常见的浏览器遗留物摘要：

* **浏览历史记录**：跟踪用户访问网站的记录，有助于识别访问恶意网站的情况。
* **自动完成数据**：基于频繁搜索的建议，结合浏览历史记录时提供洞察。
* **书签**：用户保存的用于快速访问的网站。
* **扩展和插件**：用户安装的浏览器扩展或插件。
* **缓存**：存储网页内容（例如图片、JavaScript文件）以提高网站加载速度，对取证分析很有价值。
* **登录信息**：存储的登录凭据。
* **网站图标**：与网站相关联的图标，显示在标签和书签中，有助于获取有关用户访问的额外信息。
* **浏览器会话**：与打开的浏览器会话相关的数据。
* **下载**：通过浏览器下载的文件记录。
* **表单数据**：输入到网络表单中的信息，保存以供将来自动填充建议使用。
* **缩略图**：网站的预览图像。
* **Custom Dictionary.txt**：用户添加到浏览器字典中的单词。

## 火狐浏览器

火狐浏览器将用户数据组织在配置文件中，根据操作系统存储在特定位置：

* **Linux**：`~/.mozilla/firefox/`
* **MacOS**：`/Users/$USER/Library/Application Support/Firefox/Profiles/`
* **Windows**：`%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\`

这些目录中的`profiles.ini`文件列出了用户配置文件。每个配置文件的数据存储在`profiles.ini`中的`Path`变量命名的文件夹中，该文件夹位于`profiles.ini`所在的同一目录中。如果配置文件的文件夹丢失，可能已被删除。

在每个配置文件文件夹中，您可以找到几个重要文件：

* **places.sqlite**：存储历史记录、书签和下载。Windows上的工具如[BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html)可以访问历史数据。
* 使用特定的SQL查询提取历史和下载信息。
* **bookmarkbackups**：包含书签的备份。
* **formhistory.sqlite**：存储网络表单数据。
* **handlers.json**：管理协议处理程序。
* **persdict.dat**：自定义字典单词。
* **addons.json**和**extensions.sqlite**：安装的插件和扩展信息。
* **cookies.sqlite**：Cookie存储，可通过Windows上的[MZCookiesView](https://www.nirsoft.net/utils/mzcv.html)进行检查。
* **cache2/entries**或**startupCache**：缓存数据，可通过工具如[MozillaCacheView](https://www.nirsoft.net/utils/mozilla\_cache\_viewer.html)访问。
* **favicons.sqlite**：存储网站图标。
* **prefs.js**：用户设置和偏好。
* **downloads.sqlite**：旧的下载数据库，现已整合到places.sqlite中。
* **thumbnails**：网站缩略图。
* **logins.json**：加密的登录信息。
* **key4.db**或**key3.db**：存储用于保护敏感信息的加密密钥。

此外，可以通过在`prefs.js`中搜索`browser.safebrowsing`条目来检查浏览器的反钓鱼设置，以确定安全浏览功能是否已启用或禁用。

要尝试解密主密码，可以使用[https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt)\
使用以下脚本和调用，您可以指定一个密码文件进行暴力破解：

{% code title="brute.sh" %}
```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```
## Google Chrome

Google Chrome将用户配置文件存储在特定位置，具体取决于操作系统：

- **Linux**：`~/.config/google-chrome/`
- **Windows**：`C:\Users\XXX\AppData\Local\Google\Chrome\User Data\`
- **MacOS**：`/Users/$USER/Library/Application Support/Google/Chrome/`

在这些目录中，大多数用户数据可以在**Default/**或**ChromeDefaultData/**文件夹中找到。以下文件包含重要数据：

- **History**：包含URL、下载和搜索关键字。在Windows上，可以使用[ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html)来查看历史记录。"Transition Type"列具有各种含义，包括用户点击链接、输入的URL、表单提交和页面重新加载。
- **Cookies**：存储Cookie。可使用[ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html)进行检查。
- **Cache**：保存缓存数据。Windows用户可以使用[ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html)进行检查。
- **Bookmarks**：用户书签。
- **Web Data**：包含表单历史。
- **Favicons**：存储网站图标。
- **Login Data**：包括用户名和密码等登录凭据。
- **Current Session**/**Current Tabs**：有关当前浏览会话和打开标签的数据。
- **Last Session**/**Last Tabs**：有关Chrome关闭前最后一个会话中活动站点的信息。
- **Extensions**：浏览器扩展和插件的目录。
- **Thumbnails**：存储网站缩略图。
- **Preferences**：包含丰富信息的文件，包括插件、扩展、弹出窗口、通知等的设置。
- **浏览器内置的反钓鱼**：要检查反钓鱼和恶意软件保护是否已启用，请运行`grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`。在输出中查找`{"enabled: true,"}`。

## **SQLite数据库数据恢复**

正如前面部分所示，Chrome和Firefox都使用**SQLite**数据库存储数据。可以使用工具[**sqlparse**](https://github.com/padfoot999/sqlparse)或[**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases)来**恢复已删除的条目**。

## **Internet Explorer 11**

Internet Explorer 11在各个位置管理其数据和元数据，有助于分离存储的信息及其相应的详细信息，以便轻松访问和管理。

### 元数据存储

Internet Explorer的元数据存储在`%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data`（其中VX为V01、V16或V24）。伴随的`V01.log`文件可能显示与`WebcacheVX.data`的修改时间不一致，表明需要使用`esentutl /r V01 /d`进行修复。这些存储在ESE数据库中的元数据可以使用工具如photorec和[ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html)进行恢复和检查。在**Containers**表中，可以区分每个数据段存储的特定表或容器，包括其他Microsoft工具（如Skype）的缓存详细信息。

### 缓存检查

[IECacheView](https://www.nirsoft.net/utils/ie\_cache\_viewer.html)工具允许进行缓存检查，需要提供缓存数据提取文件夹的位置。缓存的元数据包括文件名、目录、访问计数、URL来源以及指示缓存创建、访问、修改和到期时间的时间戳。

### Cookies管理

可以使用[IECookiesView](https://www.nirsoft.net/utils/iecookies.html)来探索Cookie，元数据包括名称、URL、访问计数和各种与时间相关的详细信息。持久性Cookie存储在`%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies`中，会话Cookie存储在内存中。

### 下载详细信息

可以通过[ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html)访问下载元数据，特定容器包含URL、文件类型和下载位置等数据。物理文件可以在`%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory`下找到。

### 浏览历史

要查看浏览历史，可以使用[BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html)，需要提供提取的历史文件位置和Internet Explorer的配置。这里的元数据包括修改和访问时间，以及访问计数。历史文件位于`%userprofile%\Appdata\Local\Microsoft\Windows\History`中。

### 输入的URL

输入的URL及其使用时间存储在注册表中的`NTUSER.DAT`下的`Software\Microsoft\InternetExplorer\TypedURLs`和`Software\Microsoft\InternetExplorer\TypedURLsTime`中，跟踪用户输入的最后50个URL及其最后输入时间。

## Microsoft Edge

Microsoft Edge将用户数据存储在`%userprofile%\Appdata\Local\Packages`中。各种数据类型的路径为：

- **Profile Path**：`C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC`
- **History, Cookies, and Downloads**：`C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat`
- **Settings, Bookmarks, and Reading List**：`C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb`
- **Cache**：`C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
- **Last Active Sessions**：`C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`

## Safari

Safari数据存储在`/Users/$User/Library/Safari`。关键文件包括：

- **History.db**：包含`history_visits`和`history_items`表，包含URL和访问时间戳。使用`sqlite3`进行查询。
- **Downloads.plist**：有关下载文件的信息。
- **Bookmarks.plist**：存储书签的URL。
- **TopSites.plist**：最常访问的站点。
- **Extensions.plist**：Safari浏览器扩展列表。使用`plutil`或`pluginkit`进行检索。
- **UserNotificationPermissions.plist**：允许推送通知的域。使用`plutil`进行解析。
- **LastSession.plist**：上次会话的标签。使用`plutil`进行解析。
- **浏览器内置的反钓鱼**：使用`defaults read com.apple.Safari WarnAboutFraudulentWebsites`进行检查。响应为1表示该功能已激活。

## Opera

Opera的数据存储在`/Users/$USER/Library/Application Support/com.operasoftware.Opera`中，并与Chrome的格式相同，用于历史记录和下载。

- **浏览器内置的反钓鱼**：通过检查偏好设置文件中`fraud_protection_enabled`是否设置为`true`来验证，使用`grep`。

这些路径和命令对于访问和理解不同网络浏览器存储的浏览数据至关重要。
* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter**上**关注**我们 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
