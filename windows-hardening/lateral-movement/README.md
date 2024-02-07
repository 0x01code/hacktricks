# Lateral Movement

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricks で企業を宣伝**したいですか？または、**PEASS の最新バージョンにアクセスしたり、HackTricks を PDF でダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) を発見し、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) のコレクションを見つけてください
* [**公式 PEASS & HackTricks スウェグ**](https://peass.creator-spring.com) を手に入れましょう
* **[💬](https://emojipedia.org/speech-balloon/) Discord グループ**に参加するか、[telegram グループ](https://t.me/peass) に参加するか、**Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** をフォローしてください**
* **ハッキングトリックを共有するために、[hacktricks リポジトリ](https://github.com/carlospolop/hacktricks) と [hacktricks-cloud リポジトリ](https://github.com/carlospolop/hacktricks-cloud)** に PR を提出してください

</details>

外部システムでコマンドを実行するためのさまざまな方法があります。ここでは、主要な Windows レータル移動技術の動作原理について説明します:

* [**PsExec**](../ntlm/psexec-and-winexec.md)
* [**SmbExec**](../ntlm/smbexec.md)
* [**WmicExec**](../ntlm/wmicexec.md)
* [**AtExec / SchtasksExec**](../ntlm/atexec.md)
* [**WinRM**](../ntlm/winrm.md)
* [**DCOM Exec**](dcom-exec.md)
* ****[**Pass the cookie**](https://cloud.hacktricks.xyz/pentesting-cloud/azure-security/az-lateral-movements/az-pass-the-cookie) **** (cloud)
* ****[**Pass the PRT**](https://cloud.hacktricks.xyz/pentesting-cloud/azure-security/az-lateral-movements/pass-the-prt) **** (cloud)
* [**Pass the AzureAD Certificate**](https://cloud.hacktricks.xyz/pentesting-cloud/azure-security/az-lateral-movements/az-pass-the-certificate) (cloud)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricks で企業を宣伝**したいですか？または、**PEASS の最新バージョンにアクセスしたり、HackTricks を PDF でダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) を発見し、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) のコレクションを見つけてください
* [**公式 PEASS & HackTricks スウェグ**](https://peass.creator-spring.com) を手に入れましょう
* **[💬](https://emojipedia.org/speech-balloon/) Discord グループ**に参加するか、[telegram グループ](https://t.me/peass) に参加するか、**Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** をフォローしてください**
* **ハッキングトリックを共有するために、[hacktricks リポジトリ](https://github.com/carlospolop/hacktricks) と [hacktricks-cloud リポジトリ](https://github.com/carlospolop/hacktricks-cloud)** に PR を提出してください

</details>
