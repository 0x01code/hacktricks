# 外部調査方法論

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>でAWSハッキングをゼロからヒーローまで学ぶ</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい**または **HackTricks をPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)をフォローする。
* **ハッキングトリックを共有するために、PRを** [**HackTricks**](https://github.com/carlospolop/hacktricks) **および** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **のGitHubリポジトリに提出してください。**

</details>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**ハッキングキャリア**に興味がある方や、**解読不能なものをハック**したい方 - **採用中です！**（_流暢なポーランド語の読み書きが必要です_）。

{% embed url="https://www.stmcyber.com/careers" %}

## 資産の発見

> ある企業に属するすべてのものが対象に含まれていると言われた場合、この企業が実際に所有しているものを把握したいと思うでしょう。

このフェーズの目標は、まず**主要企業が所有する企業**をすべて取得し、その後これらの企業の**資産**をすべて取得することです。これを行うために、以下の手順を実行します:

1. 主要企業の買収を見つけることで、対象に含まれる企業を把握します。
2. 各企業のASN（存在する場合）を見つけることで、各企業が所有するIP範囲を把握します。
3. リバースWhois検索を使用して、最初のエントリ（組織名、ドメインなど）に関連する他のエントリを検索します（これは再帰的に行うことができます）。
4. 他の手法（shodanの`org`および`ssl`フィルタなど）を使用して、他の資産を検索します（`ssl`トリックは再帰的に行うことができます）。

### **買収**

まず最初に、**主要企業が所有する他の企業**を知る必要があります。\
[https://www.crunchbase.com/](https://www.crunchbase.com)にアクセスし、**主要企業**を検索し、**「acquisitions」**をクリックすると、主要企業が取得した他の企業が表示されます。\
他のオプションとしては、主要企業の**Wikipedia**ページを訪れ、**買収**を検索する方法があります。

> この時点で、対象に含まれるすべての企業を把握しているはずです。それでは、彼らの資産を見つける方法を考えてみましょう。

### **ASNs**

自律システム番号（**ASN**）は、**インターネット割り当て番号機関（IANA）**によって**自律システム（AS）**に割り当てられた**一意の番号**です。\
**AS**は、外部ネットワークへのアクセスに対する明確に定義されたポリシーを持つ**IPアドレスのブロック**で構成され、単一の組織によって管理されますが、複数のオペレータで構成されている場合があります。

**企業がどのASNを割り当てられているか**を見つけることは興味深いです。これにより、**IP範囲**を見つけることができます。**スコープ**内のすべての**ホスト**に対して**脆弱性テスト**を実行し、これらのIP内の**ドメイン**を検索することが重要です。\
[**https://bgp.he.net/**](https://bgp.he.net)で企業の**名前**、**IP**、または**ドメイン**で検索できます。\
**企業の地域に応じて、これらのリンクはより多くのデータを収集するのに役立つ可能性があります:** [**AFRINIC**](https://www.afrinic.net) **（アフリカ）**、[**Arin**](https://www.arin.net/about/welcome/region/) **（北アメリカ）**、[**APNIC**](https://www.apnic.net) **（アジア）**、[**LACNIC**](https://www.lacnic.net) **（ラテンアメリカ）**、[**RIPE NCC**](https://www.ripe.net) **（ヨーロッパ）**。とにかく、おそらくすべての**有用な情報**（IP範囲とWhois）は最初のリンクに既に表示されています。
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
また、[**BBOT**](https://github.com/blacklanternsecurity/bbot)**の**サブドメインの列挙は、スキャンの最後にASNを自動的に集約して要約します。
```bash
bbot -t tesla.com -f subdomain-enum
...
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.244.131.0/24      | 5            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS16509  | 54.148.0.0/15       | 4            | AMAZON-02      | Amazon.com, Inc.           | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.45.124.0/24       | 3            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.32.0.0/12         | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.0.0.0/9           | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+

```
組織のIP範囲を見つけることもできます [http://asnlookup.com/](http://asnlookup.com)（無料APIを提供）。\
ドメインのIPとASNを見つけることができます [http://ipv4info.com/](http://ipv4info.com)。

### **脆弱性を探す**

この時点で、**スコープ内のすべての資産**を把握しているので、許可されている場合は、すべてのホストに対して **脆弱性スキャナー**（Nessus、OpenVAS）を実行することができます。\
また、[**ポートスキャン**](../pentesting-network/#discovering-hosts-from-the-outside)を実行したり、shodan のようなサービスを使用して **開いているポートを見つけ、見つけたものに応じて** この書籍で実行されているさまざまなサービスのペンテスト方法を確認する必要があります。\
**また、デフォルトのユーザー名**と**パスワード**のリストを用意して、[https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray) を使用してサービスを**ブルートフォース**することもできます。

## ドメイン

> スコープ内のすべての企業とその資産を把握しているので、スコープ内のドメインを見つける時が来ました。

_以下の提案された技術を使用すると、サブドメインも見つけることができ、その情報は過小評価されるべきではありません。_

まず、各企業の **メインドメイン** を探す必要があります。例えば、_Tesla Inc._ の場合は _tesla.com_ になります。

### **逆引きDNS**

ドメインのIP範囲を見つけたので、これらのIPに対して **逆引きDNS検索** を試みて、スコープ内のさらなるドメインを見つけることができます。被害者のDNSサーバーまたは一般的に知られているDNSサーバー（1.1.1.1、8.8.8.8）を使用してみてください。
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
### **リバースWhois（ループ）**

**whois**の中には、**組織名**、**住所**、**メールアドレス**、電話番号など、多くの興味深い**情報**が含まれています。しかし、さらに興味深いのは、これらのフィールドのいずれかで**リバースWhois検索を実行すると、会社に関連するさらなる資産を見つける**ことができることです（たとえば、同じメールアドレスが登場する他のwhoisレジストリ）。\
以下のようなオンラインツールを使用できます：

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **無料**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **無料**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **無料**
* [https://www.whoxy.com/](https://www.whoxy.com) - **無料** web、APIは無料ではありません。
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - 有料
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - 有料（**100回無料**検索）
* [https://www.domainiq.com/](https://www.domainiq.com) - 有料

[**DomLink** ](https://github.com/vysecurity/DomLink)を使用してこのタスクを自動化できます（whoxy APIキーが必要です）。\
[amass](https://github.com/OWASP/Amass)を使用して自動的なリバースWhoisの発見を行うこともできます： `amass intel -d tesla.com -whois`

**新しいドメインを見つけるたびに、このテクニックを使用してさらに多くのドメイン名を発見できることに注意してください。**

### **トラッカー**

2つの異なるページで**同じトラッカーの同じID**を見つけると、**両方のページ**が**同じチームによって管理されている**と推測できます。\
たとえば、複数のページで同じ**Google Analytics ID**や同じ**Adsense ID**を見つけた場合。

これらのトラッカーやその他の情報を検索できるいくつかのページやツールがあります：

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **Favicon**

私たちのターゲットに関連するドメインやサブドメインを、同じファビコンアイコンハッシュを探して見つけることができることを知っていましたか？これが、[@m4ll0k2](https://twitter.com/m4ll0k2)によって作成された[favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py)ツールが行うことです。以下は、その使用方法です：
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
![favihash - 同じファビコンアイコンハッシュを持つドメインを発見する](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

単純に言えば、favihashを使用すると、ターゲットと同じファビコンアイコンハッシュを持つドメインを発見できます。

さらに、[**このブログ投稿**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139)で説明されているように、ファビコンハッシュを使用して技術を検索することもできます。つまり、脆弱なWebテクノロジーのバージョンのファビコンのハッシュを知っている場合、shodanで検索して**より多くの脆弱な場所を見つける**ことができます。
```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```
これは、ウェブの**ファビコンハッシュを計算する方法**です：
```python
import mmh3
import requests
import codecs

def fav_hash(url):
response = requests.get(url)
favicon = codecs.encode(response.content,"base64")
fhash = mmh3.hash(favicon)
print(f"{url} : {fhash}")
return fhash
```
### **著作権 / ユニークな文字列**

同じ組織内の異なるウェブ間で共有される可能性のある文字列をウェブページ内で検索します。著作権文字列は良い例です。その文字列を**Google**、他の**ブラウザ**、または**Shodan**で検索します: `shodan search http.html:"Copyright string"`

### **CRT 時間**

次のようなクーロンジョブを持つことが一般的です。
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
### **外部リコン手法**

サーバー上のすべてのドメイン証明書を更新します。これは、このために使用されるCAが生成された時間を有効期間内に設定していなくても、**証明書透明性ログで同じ会社に属するドメインを見つけることが可能**です。\
詳細については、[**こちらの解説**](https://swarm.ptsecurity.com/discovering-domains-via-a-time-correlation-attack/)をご覧ください。

### **パッシブな乗っ取り**

クラウドプロバイダーに属するIPにサブドメインを割り当て、そのIPアドレスをいつの間にか**失い、DNSレコードを削除し忘れる**ことが一般的です。そのため、クラウド（たとえばDigital Ocean）でVMを**起動**するだけで、実際にはいくつかのサブドメインを**乗っ取ることができます**。

[**この投稿**](https://kmsec.uk/blog/passive-takeover/)では、そのストーリーを説明し、**DigitalOceanでVMを起動**し、新しいマシンの**IPv4**を取得し、Virustotalでそれを指すサブドメインレコードを検索するスクリプトを提案しています。

### **その他の方法**

**新しいドメインを見つけるたびに、この手法を使用できることに注意してください**。

**Shodan**

すでにIPスペースを所有している組織の名前を知っている場合、そのデータでshodanで検索できます: `org:"Tesla, Inc."` TLS証明書で新しい予期しないドメインを見つけるために、見つかったホストをチェックしてください。

主要なWebページの**TLS証明書**にアクセスし、**組織名**を取得し、その名前を持つ**すべてのWebページのTLS証明書**内でshodanで検索するか、`ssl:"Tesla Motors"`のフィルタを使用するか、[**sslsearch**](https://github.com/HarshVaragiya/sslsearch)のようなツールを使用できます。

**Assetfinder**

[**Assetfinder**](https://github.com/tomnomnom/assetfinder)は、主要ドメインに関連する**ドメイン**とそれらの**サブドメイン**を探すツールで、非常に素晴らしいです。

### **脆弱性の検索**

[ドメイン乗っ取り](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover)をチェックしてください。会社が**ドメインを使用している**が**所有権を失っている**可能性があります。安価であれば登録し、会社に通知してください。

アセットの発見で見つけたIPとは異なる**ドメインを持つドメイン**を見つけた場合、**基本的な脆弱性スキャン**（NessusまたはOpenVASを使用）と**nmap/masscan/shodan**での一部の[**ポートスキャン**](../pentesting-network/#discovering-hosts-from-the-outside)を実行する必要があります。実行されているサービスに応じて、**この書籍でそれらを"攻撃"するためのトリック**を見つけることができます。\
_ドメインがクライアントによって制御されていないIP内にホストされている場合があるため、スコープ外にあることに注意してください。_

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**バグバウンティのヒント**: **Intigriti**に**サインアップ**してください。これは、ハッカーによって作成されたプレミアム**バグバウンティプラットフォーム**です！今すぐ[**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks)で参加し、最大**$100,000**のバウンティを獲得し始めましょう！

{% embed url="https://go.intigriti.com/hacktricks" %}

## サブドメイン

> スコープ内のすべての企業、各企業のすべての資産、および企業に関連するすべてのドメインを知っています。

見つかった各ドメインのすべての可能なサブドメインを見つける時が来ました。

### **DNS**

**DNS**レコードから**サブドメイン**を取得しましょう。**ゾーン転送**も試してみるべきです（脆弱性がある場合は報告する必要があります）。
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

大量のサブドメインを取得する最速の方法は、外部ソースで検索することです。最もよく使われる**ツール**は以下のものです（より良い結果を得るためには、APIキーを設定してください）：

* [**BBOT**](https://github.com/blacklanternsecurity/bbot)
```bash
# subdomains
bbot -t tesla.com -f subdomain-enum

# subdomains (passive only)
bbot -t tesla.com -f subdomain-enum -rf passive

# subdomains + port scan + web screenshots
bbot -t tesla.com -f subdomain-enum -m naabu gowitness -n my_scan -o .
```
* [**Amass**](https://github.com/OWASP/Amass)
```bash
amass enum [-active] [-ip] -d tesla.com
amass enum -d tesla.com | grep tesla.com # To just list subdomains
```
* [**subfinder**](https://github.com/projectdiscovery/subfinder)
```bash
# Subfinder, use -silent to only have subdomains in the output
./subfinder-linux-amd64 -d tesla.com [-silent]
```
* [**findomain**](https://github.com/Edu4rdSHL/findomain/)
```bash
# findomain, use -silent to only have subdomains in the output
./findomain-linux -t tesla.com [--quiet]
```
* [**OneForAll**](https://github.com/shmilylty/OneForAll/tree/master/docs/en-us)
```bash
python3 oneforall.py --target tesla.com [--dns False] [--req False] [--brute False] run
```
* [**assetfinder**](https://github.com/tomnomnom/assetfinder)
```bash
assetfinder --subs-only <domain>
```
* [**Sudomy**](https://github.com/Screetsec/Sudomy)
```bash
# It requires that you create a sudomy.api file with API keys
sudomy -d tesla.com
```
* [**vita**](https://github.com/junnlikestea/vita)
```
vita -d tesla.com
```
* [**theHarvester**](https://github.com/laramies/theHarvester)
```bash
theHarvester -d tesla.com -b "anubis, baidu, bing, binaryedge, bingapi, bufferoverun, censys, certspotter, crtsh, dnsdumpster, duckduckgo, fullhunt, github-code, google, hackertarget, hunter, intelx, linkedin, linkedin_links, n45ht, omnisint, otx, pentesttools, projectdiscovery, qwant, rapiddns, rocketreach, securityTrails, spyse, sublist3r, threatcrowd, threatminer, trello, twitter, urlscan, virustotal, yahoo, zoomeye"
```
以下は、サブドメインを直接的に特定することに特化していなくても、サブドメインを見つけるのに役立つ**他の興味深いツール/API**があります：

- [**Crobat**](https://github.com/cgboal/sonarsearch)**:** サブドメインを取得するためにAPI [https://sonar.omnisint.io](https://sonar.omnisint.io) を使用します
```bash
# Get list of subdomains in output from the API
## This is the API the crobat tool will use
curl https://sonar.omnisint.io/subdomains/tesla.com | jq -r ".[]"
```
* [**JLDC無料API**](https://jldc.me/anubis/subdomains/google.com)
```bash
curl https://jldc.me/anubis/subdomains/tesla.com | jq -r ".[]"
```
* [**RapidDNS**](https://rapiddns.io) 無料API
```bash
# Get Domains from rapiddns free API
rapiddns(){
curl -s "https://rapiddns.io/subdomain/$1?full=1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
rapiddns tesla.com
```
* [**https://crt.sh/**](https://crt.sh)
```bash
# Get Domains from crt free API
crt(){
curl -s "https://crt.sh/?q=%25.$1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
crt tesla.com
```
* [**gau**](https://github.com/lc/gau)**:** 特定のドメインからAlienVaultのOpen Threat Exchange、Wayback Machine、Common Crawlに知られているURLを取得します。
```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```
* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **&** [**subscraper**](https://github.com/Cillian-Collins/subscraper): 彼らはWebをスクレイピングし、JSファイルからサブドメインを抽出します。
```bash
# Get only subdomains from SubDomainizer
python3 SubDomainizer.py -u https://tesla.com | grep tesla.com

# Get only subdomains from subscraper, this already perform recursion over the found results
python subscraper.py -u tesla.com | grep tesla.com | cut -d " " -f
```
* [**Shodan**](https://www.shodan.io/)
```bash
# Get info about the domain
shodan domain <domain>
# Get other pages with links to subdomains
shodan search "http.html:help.domain.com"
```
* [**Censysサブドメイン検出ツール**](https://github.com/christophetd/censys-subdomain-finder)
```bash
export CENSYS_API_ID=...
export CENSYS_API_SECRET=...
python3 censys-subdomain-finder.py tesla.com
```
* [**DomainTrail.py**](https://github.com/gatete/DomainTrail)
```bash
python3 DomainTrail.py -d example.com
```
* [**securitytrails.com**](https://securitytrails.com/) はサブドメインやIPの履歴を検索するための無料APIを提供しています
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

このプロジェクトは、**バグバウンティプログラムに関連するすべてのサブドメインを無料で提供**しています。このデータには、[chaospy](https://github.com/dr-0x0x/chaospy)を使用したり、このプロジェクトで使用されているスコープにアクセスすることもできます [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

これらのツールの**比較**をこちらで見つけることができます: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNSブルートフォース**

可能なサブドメイン名を使用してDNSサーバーをブルートフォースし、新しい**サブドメイン**を見つけてみましょう。

このアクションには、次のような**一般的なサブドメインのワードリスト**が必要です:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

さらに、信頼できるDNSリゾルバのIPも必要です。信頼できるDNSリゾルバのリストを生成するには、[https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) からリゾルバをダウンロードし、それらをフィルタリングするために [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) を使用するか、[https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt) を使用することができます

DNSブルートフォースに最も推奨されるツールは次のとおりです:

* [**massdns**](https://github.com/blechschmidt/massdns): これは効果的なDNSブルートフォースを実行した最初のツールです。非常に高速ですが、誤検知しやすいです。
```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```
* [**gobuster**](https://github.com/OJ/gobuster): これは1つのリゾルバーを使用していると思います
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) は、`massdns` をラップしたもので、有効なサブドメインをアクティブなブルートフォースを使用して列挙し、ワイルドカード処理と簡単な入出力サポートを備えています。Go言語で書かれています。
```
shuffledns -d example.com -list example-subdomains.txt -r resolvers.txt
```
* [**puredns**](https://github.com/d3mondev/puredns): それは`massdns`も使用します。
```
puredns bruteforce all.txt domain.com
```
* [**aiodnsbrute**](https://github.com/blark/aiodnsbrute)は、非同期でドメイン名を総当たり攻撃するためにasyncioを使用します。
```
aiodnsbrute -r resolvers -w wordlist.txt -vv -t 1024 domain.com
```
### 第2のDNSブルートフォースラウンド

オープンソースやブルートフォースを使用してサブドメインを見つけた後、見つかったサブドメインの変更を生成してさらに見つけることができます。この目的にはいくつかのツールが役立ちます：

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** ドメインとサブドメインを与えられた場合、順列を生成します。
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): ドメインとサブドメインを与えられた場合、順列を生成します。
* goaltdnsの順列の**ワードリスト**は[**こちら**](https://github.com/subfinder/goaltdns/blob/master/words.txt)で入手できます。
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** ドメインとサブドメインを与えられた場合、順列を生成します。順列ファイルが指定されていない場合、gotatorは独自のファイルを使用します。
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): サブドメインの順列を生成するだけでなく、それらを解決しようともします（ただし、以前にコメントされたツールを使用する方が良いです）。
* altdnsの順列は[**こちら**](https://github.com/infosec-au/altdns/blob/master/words.txt)で入手できます。
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): サブドメインの順列、変異、変更を実行する別のツール。このツールは結果を総当たり攻撃します（DNSワイルドカードはサポートされていません）。
* dmutの順列ワードリストは[**こちら**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt)で入手できます。
```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```
* [**alterx**](https://github.com/projectdiscovery/alterx)**:** ドメインに基づいて、指定されたパターンに基づいて新しい潜在的なサブドメイン名を生成し、さらに多くのサブドメインを発見しようとします。

#### スマートな順列生成

* [**regulator**](https://github.com/cramppet/regulator): 詳細については、この[**post**](https://cramppet.github.io/regulator/index.html)を読んでくださいが、基本的には**発見されたサブドメイン**から**主要な部分**を取得し、それらを混ぜてさらに多くのサブドメインを見つけようとします。
```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_は、極めてシンプルで効果的なDNS応答ガイドアルゴリズムと組み合わされたサブドメインブルートフォースファズツールです。_subzuf_は、カスタマイズされたワードリストや過去のDNS/TLSレコードなどの提供された入力データを利用して、より対応するドメイン名を正確に合成し、DNSスキャン中に収集された情報に基づいてさらにループで拡張します。
```
echo www | subzuf facebook.com
```
### **サブドメイン発見ワークフロー**

このブログ記事をチェックしてください。**Trickestワークフロー**を使用してドメインから**サブドメインの発見を自動化**する方法について書いています。これにより、コンピュータで手動で多くのツールを起動する必要がなくなります。

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / バーチャルホスト**

サブドメインに属する**1つ以上のウェブページを含むIPアドレス**を見つけた場合、そのIP内の他のサブドメインを見つけることができます。これは、IP内のドメインを検索するか、**そのIP内のVHostドメイン名をブルートフォースする**ことで行うことができます。

#### OSINT

[**HostHunter**](https://github.com/SpiderLabs/HostHunter) **やその他のAPI**を使用して、IP内の**VHostを見つける**ことができます。

**ブルートフォース**

Webサーバーに隠されている可能性があるサブドメインがあると疑っている場合、ブルートフォースしてみることができます。
```bash
ffuf -c -w /path/to/wordlist -u http://victim.com -H "Host: FUZZ.victim.com"

gobuster vhost -u https://mysite.com -t 50 -w subdomains.txt

wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --hc 400,404,403 -H "Host: FUZZ.example.com" -u http://example.com -t 100

#From https://github.com/allyshka/vhostbrute
vhostbrute.py --url="example.com" --remoteip="10.1.1.15" --base="www.example.com" --vhosts="vhosts_full.list"

#https://github.com/codingo/VHostScan
VHostScan -t example.com
```
{% hint style="info" %}
このテクニックを使用すると、内部/非表示のエンドポイントにアクセスできるかもしれません。
{% endhint %}

### **CORS Brute Force**

時々、有効なドメイン/サブドメインが _**Origin**_ ヘッダーに設定されている場合にのみ、_**Access-Control-Allow-Origin**_ ヘッダーを返すページが見つかります。 これらのシナリオでは、この動作を悪用して、新しい **サブドメイン** を**発見**することができます。
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### Buckets Brute Force

**サブドメイン**を探している間に、それがどの種類の**バケット**を**指しているか**を確認し、その場合は[**権限をチェック**](../../network-services-pentesting/pentesting-web/buckets/)**してください**。\
また、この時点でスコープ内のすべてのドメインを知っているため、[**可能なバケット名をブルートフォースし、権限をチェック**](../../network-services-pentesting/pentesting-web/buckets/)してください。

### モニタリング

**ドメイン**の**新しいサブドメイン**が作成されたかどうかを**モニタリング**することができます。[**Certificate Transparency**ログを監視することで、[**sublert**](https://github.com/yassineaboukir/sublert/blob/master/sublert.py)が行います。

### 脆弱性の探索

可能な[**サブドメインテイクオーバー**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover)をチェックしてください。\
サブドメインが**S3バケット**を指している場合は、[**権限をチェック**](../../network-services-pentesting/pentesting-web/buckets/)してください。

アセットの発見で見つけたIPと異なるIPを持つ**サブドメイン**を見つけた場合、**基本的な脆弱性スキャン**（NessusまたはOpenVASを使用）と**ポートスキャン**（**nmap/masscan/shodan**を使用）を実行する必要があります。実行されているサービスに応じて、**この書籍でそれらを"攻撃"するためのトリック**を見つけることができます。\
_サブドメインがクライアントによって制御されていないIP内にホストされている場合があるため、スコープ外にある可能性があることに注意してください。_

## IPs

初期段階で**いくつかのIP範囲、ドメイン、およびサブドメイン**を見つけたかもしれません。\
これらの範囲から**すべてのIP**と**ドメイン/サブドメイン（DNSクエリ）**を再収集する時が来ました。

以下の**無料API**を使用すると、ドメインとサブドメインが以前に使用したIPを見つけることもできます。これらのIPはクライアントが所有している可能性があり（[**CloudFlareのバイパス**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md)を見つけることができるかもしれません）

* [**https://securitytrails.com/**](https://securitytrails.com/)

また、ツール[**hakip2host**](https://github.com/hakluke/hakip2host)を使用して特定のIPアドレスを指すドメインをチェックすることもできます。

### 脆弱性の探索

CDNに属さないすべてのIPに**ポートスキャン**を実行してください（そこに興味深いものはほとんど見つからない可能性が高いため）。発見された実行中のサービスで**脆弱性を見つける**ことができるかもしれません。

**ホストをスキャンする方法についての**[**ガイド**](../pentesting-network/) **を見つけてください。**

## Webサーバーの探索

> すべての企業とその資産を見つけ、スコープ内のIP範囲、ドメイン、およびサブドメインを知っています。Webサーバーを検索する時が来ました。

前のステップでおそらくすでに発見したIPとドメインの**調査**を実行しているかもしれませんので、**すでに可能なすべてのWebサーバー**を見つけているかもしれません。しかし、まだ見つけていない場合は、スコープ内のWebサーバーを検索するための**高速なトリック**を見ていきます。

これは**Webアプリの発見**に向けられているため、スコープ内で**脆弱性**と**ポートスキャン**も実行する必要があります（**スコープ内で許可されている場合**）。

[**masscanを使用して**関連する**Web**サーバーに**オープンポート**を発見する高速な方法はこちらにあります](../pentesting-network/#http-port-discovery)。\
Webサーバーを探すためのもう1つの便利なツールは[**httprobe**](https://github.com/tomnomnom/httprobe)**、**[**fprobe**](https://github.com/theblackturtle/fprobe)、および[**httpx**](https://github.com/projectdiscovery/httpx)です。ドメインのリストを渡すと、ポート80（http）および443（https）に接続しようとします。さらに、他のポートを試すように指示することもできます。
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
### **スクリーンショット**

スコープ内のすべてのWebサーバー（企業のIPアドレス、すべてのドメイン、サブドメインの中のIPアドレス）を発見したので、**どこから始めればよいかわからない**かもしれません。ですので、単純にすべてのWebサーバーのスクリーンショットを撮影してみましょう。**メインページ**を見るだけで、**脆弱性を抱えやすい**と考えられる**奇妙な**エンドポイントを見つけることができます。

提案されたアイデアを実行するために、[**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness)、[**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot)、[**Aquatone**](https://github.com/michenriksen/aquatone)、[**Shutter**](https://shutter-project.org/downloads/third-party-packages/)、または[**webscreenshot**](https://github.com/maaaaz/webscreenshot)**を使用**できます。

さらに、[**eyeballer**](https://github.com/BishopFox/eyeballer)を使用して、**スクリーンショット**全体を実行し、**脆弱性を含む可能性が高い**ものとそうでないものを教えてもらうことができます。

## パブリッククラウド資産

企業に属する潜在的なクラウド資産を見つけるためには、その企業を識別するためのキーワードのリストから始める必要があります。たとえば、暗号通貨企業の場合、"crypto"、"wallet"、"dao"、"<domain_name>"、<"subdomain_names">などの単語を使用するかもしれません。

また、**バケツで使用される一般的な単語のワードリスト**が必要です：

- [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
- [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
- [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

その単語を使用して**順列**を生成し、[**Second Round DNS Brute-Force**](./#second-dns-bruteforce-round)をチェックしてください。

生成されたワードリストを使用して、[**cloud\_enum**](https://github.com/initstring/cloud\_enum)**、[**CloudScraper**](https://github.com/jordanpotti/CloudScraper)**、[**cloudlist**](https://github.com/projectdiscovery/cloudlist)、または[**S3Scanner**](https://github.com/sa7mon/S3Scanner)**などのツールを使用**できます。

クラウド資産を探す際には、**AWSのバケツだけでなく**、他のものも探す必要があります。

### **脆弱性を探す**

**オープンなバケツや公開されたクラウド関数**などが見つかった場合は、**アクセス**して、提供されているものを確認し、悪用できるかどうかを試してください。

## メール

スコープ内の**ドメイン**と**サブドメイン**を使用すると、企業のメールアドレスを検索するのに必要なすべてが揃います。これらは私にとって最も効果的だった企業のメールアドレスを見つけるための**API**と**ツール**です：

- [**theHarvester**](https://github.com/laramies/theHarvester) - APIsを使用
- [**https://hunter.io/**](https://hunter.io/)のAPI（無料版）
- [**https://app.snov.io/**](https://app.snov.io/)のAPI（無料版）
- [**https://minelead.io/**](https://minelead.io/)のAPI（無料版）

### **脆弱性を探す**

後でメールアドレスは、WebログインやSSHなどの認証サービスを**ブルートフォース攻撃**するのに役立ちます。また、**フィッシング**にも必要です。さらに、これらのAPIは、メールアドレスの背後にいる**個人に関する情報**をさらに提供してくれるため、フィッシングキャンペーンに役立ちます。

## 資格情報の漏洩

**ドメイン**、**サブドメイン**、および**メールアドレス**を使用して、過去に漏洩した資格情報を探すことができます：

- [https://leak-lookup.com](https://leak-lookup.com/account/login)
- [https://www.dehashed.com/](https://www.dehashed.com/)

### **脆弱性を探す**

**有効な漏洩した**資格情報を見つけた場合、これは非常に簡単な勝利です。

## シークレットの漏洩

資格情報の漏洩は、**機密情報が漏洩して販売された**企業のハッキングに関連しています。ただし、企業はそのデータベースにない情報が含まれている**他の漏洩**にも影響を受ける可能性があります。

### Githubの漏洩

資格情報やAPIが、**企業**またはそのgithub企業の**開発者**の**パブリックリポジトリ**に漏洩している可能性があります。\
[**Leakos**](https://github.com/carlospolop/Leakos)ツールを使用して、**組織**とその**開発者**のすべての**パブリックリポジトリ**を**ダウンロード**し、自動的に[gitleaks](https://github.com/zricethezav/gitleaks)を実行できます。

**Leakos**は、**URL**を渡された**テキスト**全体に**gitleaks**を実行するためにも使用できます。なぜなら、**ウェブページにも秘密が含まれている**場合があるからです。

#### Github Dorks

攻撃対象の組織で検索する可能性のある**github dorks**を見つけるために、この**ページ**もチェックしてください：

{% content-ref url="github-leaked-secrets.md" %}
[github-leaked-secrets.md](github-leaked-secrets.md)
{% endcontent-ref %}

### ペーストの漏洩

攻撃者や作業者が**企業のコンテンツをペーストサイトに公開**することがあります。これには**機密情報**が含まれる場合も含まれない場合もありますが、検索する価値があります。\
80以上のペーストサイトで検索するために[**Pastos**](https://github.com/carlospolop/Pastos)ツールを使用できます。

### Google Dorks

古くからあるが有用なGoogle Dorksは、**そこにあってはならない情報を見つける**のに常に役立ちます。唯一の問題は、[**google-hacking-database**](https://www.exploit-db.com/google-hacking-database)には数千ものクエリが含まれており、手動で実行することはできないことです。したがって、お気に入りの10個を選択するか、[**Gorks**](https://github.com/carlospolop/Gorks)などの**ツールを使用してすべてを実行**することができます。

_通常のGoogleブラウザを使用してデータベース全体を実行することを期待しているツールは、Googleがすぐにブロックするため、決して終了しません。_

### **脆弱性を探す**

**有効な漏洩した**資格情報やAPIトークンを見つけた場合、これは非常に簡単な勝利です。

## パブリックコードの脆弱性

企業が**オープンソースコード**を持っていることがわかった場合、そのコードを**分析**して**脆弱性**を検索できます。

**言語によって異なるツール**を使用できます：

{% content-ref url="../../network-services-pentesting/pentesting-web/code-review-tools.md" %}
[code-review-tools.md](../../network-services-pentesting/pentesting-web/code-review-tools.md)
{% endcontent-ref %}

また、次のような無料のサービスを使用して**パブリックリポジトリをスキャン**することもできます：

- [**Snyk**](https://app.snyk.io/)
## [**Webペンテスト方法論**](../../network-services-pentesting/pentesting-web/)

**バグハンター**によって見つかる**脆弱性の大部分**は**Webアプリケーション**内に存在するため、この時点で**Webアプリケーションのテスト方法論**について話したいと思います。詳細は[**こちらで見つけることができます**](../../network-services-pentesting/pentesting-web/)。

また、[**Web自動スキャナーのオープンソースツール**](../../network-services-pentesting/pentesting-web/#automatic-scanners)セクションに特別な言及をしたいと思います。非常に機密性の高い脆弱性を見つけることは期待できないかもしれませんが、**初期のWeb情報を取得するためのワークフローに役立ちます**。

## 要約

> おめでとうございます！この時点で**すべての基本的な列挙**をすでに実行しています。はい、基本的な列挙ですが、（後でさらにトリックを見るでしょう）他にもたくさんの列挙ができます。

したがって、すでに次のことを行っています：

1. スコープ内の**すべての企業**を見つけました
2. 企業に属する**すべての資産**を見つけました（スコープ内であればいくつかの脆弱性スキャンを実行しました）
3. 企業に属する**すべてのドメイン**を見つけました
4. ドメインの**すべてのサブドメイン**を見つけました（サブドメインの乗っ取りはありましたか？）
5. スコープ内の**CDNからでないIP**を含むすべての**IP**を見つけました
6. **すべてのWebサーバー**を見つけ、それらの**スクリーンショット**を撮りました（深く調査する価値のある奇妙なものはありましたか？）
7. 企業に属する**潜在的なパブリッククラウド資産**をすべて見つけました
8. **簡単に大きな勝利をもたらす可能性のある** **メール**、**資格情報の漏洩**、および**秘密の漏洩**を見つけました
9. 見つけたすべてのWebを**ペンテスト**しました

## **完全なRecon自動ツール**

与えられたスコープに対して提案されたアクションの一部を実行するいくつかのツールがあります。

* [**https://github.com/yogeshojha/rengine**](https://github.com/yogeshojha/rengine)
* [**https://github.com/j3ssie/Osmedeus**](https://github.com/j3ssie/Osmedeus)
* [**https://github.com/six2dez/reconftw**](https://github.com/six2dez/reconftw)
* [**https://github.com/hackerspider1/EchoPwn**](https://github.com/hackerspider1/EchoPwn) - 少し古く、更新されていません

## **参考文献**

* [**@Jhaddix**](https://twitter.com/Jhaddix)のすべての無料コース、[**The Bug Hunter's Methodology v4.0 - Recon Edition**](https://www.youtube.com/watch?v=p4JgIu1mceI)など

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**ハッキングキャリア**に興味がある方や**解読不能なものをハック**したい方 - **採用中です！**（_流暢なポーランド語の書き言葉と会話が必要です_）。

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>**htARTE（HackTricks AWS Red Team Expert）**で**ゼロからヒーローまでのAWSハッキング**を学びましょう！</summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[NFTs](https://opensea.io/collection/the-peass-family)コレクションを見つける
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)をフォローする
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks)のGitHubリポジトリにPRを提出して、あなたのハッキングトリックを共有する

</details>
