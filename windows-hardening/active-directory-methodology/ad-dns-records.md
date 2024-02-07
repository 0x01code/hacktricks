# AD DNS レコード

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricks で企業を宣伝**したいですか？または **PEASS の最新バージョンにアクセス**したいですか？または **HackTricks を PDF でダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) を発見しましょう。独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) のコレクションです。
* [**公式 PEASS & HackTricks スワッグ**](https://peass.creator-spring.com) を手に入れましょう。
* **[💬](https://emojipedia.org/speech-balloon/) Discord グループ**に**参加**するか、[**telegram グループ**](https://t.me/peass)に参加するか、**Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** をフォロー**してください。
* **ハッキングテクニックを共有する**には、[hacktricks リポジトリ](https://github.com/carlospolop/hacktricks) と [hacktricks-cloud リポジトリ](https://github.com/carlospolop/hacktricks-cloud) に PR を提出してください。

</details>

Active Directory のデフォルトでは、**任意のユーザー**がドメインまたはフォレスト DNS ゾーン内のすべての DNS レコードを**列挙**できます。これは、AD 環境において DNS ゾーンの子オブジェクトをリストアップできるゾーン転送と同様です。

ツール [**adidnsdump**](https://github.com/dirkjanm/adidnsdump) は、内部ネットワークの調査目的でゾーン内の**すべての DNS レコード**を**列挙**および**エクスポート**することを可能にします。
```bash
git clone https://github.com/dirkjanm/adidnsdump
cd adidnsdump
pip install .

adidnsdump -u domain_name\\username ldap://10.10.10.10 -r
cat records.csv
```
詳細については[こちら](https://dirkjanm.io/getting-in-the-zone-dumping-active-directory-dns-with-adidnsdump/)を参照してください。

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで企業を宣伝**したいですか？または**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[NFTs](https://opensea.io/collection/the-peass-family)コレクションをご覧ください
* [**公式PEASS＆HackTricksスウェグ**](https://peass.creator-spring.com)を手に入れましょう
* **[💬](https://emojipedia.org/speech-balloon/) [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter**で**🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**をフォロー**してください。
* **ハッキングトリックを共有するには、[hacktricksリポジトリ](https://github.com/carlospolop/hacktricks)と[hacktricks-cloudリポジトリ](https://github.com/carlospolop/hacktricks-cloud)**にPRを提出してください。

</details>
