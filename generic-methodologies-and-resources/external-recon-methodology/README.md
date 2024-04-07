# Metodologia zewnętrznego rozpoznania

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) **i** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **na GitHubie.**

</details>

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Jeśli interesuje Cię **kariera hakerska** i hakowanie niemożliwego - **rekrutujemy!** (_wymagana biegła znajomość języka polskiego_).

{% embed url="https://www.stmcyber.com/careers" %}

## Odkrywanie zasobów

> Powiedziano Ci, że wszystko należące do pewnej firmy znajduje się w zakresie, i chcesz dowiedzieć się, co ta firma faktycznie posiada.

Celem tej fazy jest uzyskanie wszystkich **firm należących do głównej firmy**, a następnie wszystkich **zasobów** tych firm. Aby to osiągnąć, będziemy:

1. Znaleźć przejęcia głównej firmy, co pozwoli nam poznać firmy wchodzące w zakres.
2. Znaleźć ASN (jeśli istnieje) każdej firmy, co pozwoli nam poznać zakresy IP posiadane przez każdą firmę.
3. Użyć odwróconego wyszukiwania whois, aby szukać innych wpisów (nazwy organizacji, domeny...) związanych z pierwszym (można to zrobić rekurencyjnie).
4. Użyć innych technik, takich jak filtry shodan `org` i `ssl`, aby szukać innych zasobów (szczególnie trik z `ssl` można wykonać rekurencyjnie).

### **Przejęcia**

Po pierwsze, musimy dowiedzieć się, które **inne firmy należą do głównej firmy**.\
Jedną z opcji jest odwiedzenie [https://www.crunchbase.com/](https://www.crunchbase.com), **wyszukanie** **głównej firmy**, a następnie **kliknięcie** w "**przejęcia**". Tam zobaczysz inne firmy przejęte przez główną.\
Inną opcją jest odwiedzenie strony **Wikipedii** głównej firmy i wyszukanie **przejęć**.

> Ok, w tym momencie powinieneś znać wszystkie firmy wchodzące w zakres. Spróbujmy teraz znaleźć ich zasoby.

### **ASNs**

Numer autonomicznego systemu (**ASN**) to **unikalny numer** przypisany **autonomicznemu systemowi** (AS) przez **Internet Assigned Numbers Authority (IANA)**.\
AS składa się z **bloków** adresów **IP**, które mają jasno określoną politykę dostępu do sieci zewnętrznych i są administrowane przez jedną organizację, ale mogą składać się z kilku operatorów.

Interesujące jest dowiedzieć się, czy **firma ma przypisany jakiś ASN**, aby poznać jej **zakresy IP**. Warto przeprowadzić **test podatności** przeciwko wszystkim **hostom** w **zakresie** i szukać **domen** w tych IP.\
Możesz **szukać** po nazwie firmy, po **IP** lub po **domenie** na stronie [**https://bgp.he.net/**](https://bgp.he.net)**.**\
**W zależności od regionu firmy te linki mogą być przydatne do zebrania więcej danych:** [**AFRINIC**](https://www.afrinic.net) **(Afryka),** [**Arin**](https://www.arin.net/about/welcome/region/)**(Ameryka Północna),** [**APNIC**](https://www.apnic.net) **(Azja),** [**LACNIC**](https://www.lacnic.net) **(Ameryka Łacińska),** [**RIPE NCC**](https://www.ripe.net) **(Europa). W każdym razie, prawdopodobnie wszystkie** przydatne informacje **(zakresy IP i Whois)** znajdują się już na pierwszym linku.
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
Również, [**BBOT**](https://github.com/blacklanternsecurity/bbot)** automatycznie agreguje i podsumowuje ASNs na końcu skanowania poddomen.
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
Możesz znaleźć zakresy IP organizacji również korzystając z [http://asnlookup.com/](http://asnlookup.com) (ma darmowe API).\
Możesz znaleźć IP i ASN domeny korzystając z [http://ipv4info.com/](http://ipv4info.com).

### **Wyszukiwanie podatności**

W tym momencie znamy **wszystkie zasoby w zakresie**, więc jeśli masz zgodę, możesz uruchomić skaner **podatności** (Nessus, OpenVAS) na wszystkich hostach.\
Możesz także uruchomić niektóre [**skanowania portów**](../pentesting-network/#discovering-hosts-from-the-outside) **lub skorzystać z usług takich jak** shodan **aby znaleźć** otwarte porty **i w zależności od tego, co znajdziesz, powinieneś** sprawdzić w tej książce, jak testować penetracyjnie kilka możliwych usług działających.\
**Warto również wspomnieć, że możesz przygotować listy domyślnych nazw użytkowników i** haseł **i spróbować** próbować siłowo **usługi za pomocą [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray).

## Domeny

> Znamy wszystkie firmy w zakresie i ich zasoby, czas znaleźć domeny w zakresie.

_Proszę zauważyć, że w proponowanych technikach można również znaleźć subdomeny i ta informacja nie powinna być lekceważona._

Po pierwsze, powinieneś szukać **głównej domeny**(ów) każdej firmy. Na przykład, dla _Tesla Inc._ będzie to _tesla.com_.

### **Odwrócone DNS**

Po znalezieniu wszystkich zakresów IP domen, możesz spróbować wykonać **odwrócone wyszukiwanie DNS** na tych **IP, aby znaleźć więcej domen w zakresie**. Spróbuj użyć serwera DNS ofiary lub znanego serwera DNS (1.1.1.1, 8.8.8.8)
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
Aby to działało, administrator musi ręcznie włączyć PTR.\
Możesz także skorzystać z narzędzia online do uzyskania tych informacji: [http://ptrarchive.com/](http://ptrarchive.com)

### **Odwrócony Whois (pętla)**

Wewnątrz **whois** możesz znaleźć wiele interesujących **informacji**, takich jak **nazwa organizacji**, **adres**, **adresy e-mail**, numery telefonów... Ale co jeszcze bardziej interesujące, to że możesz znaleźć **więcej zasobów związanych z firmą**, jeśli wykonasz **odwrócone wyszukiwanie whois według któregokolwiek z tych pól** (na przykład inne rejestry whois, gdzie pojawi się ten sam adres e-mail).\
Możesz skorzystać z narzędzi online, takich jak:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **Darmowe**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **Darmowe**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **Darmowe**
* [https://www.whoxy.com/](https://www.whoxy.com) - **Darmowe** w sieci, nie darmowe API.
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - Płatne
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - Płatne (tylko **100 darmowych** wyszukiwań)
* [https://www.domainiq.com/](https://www.domainiq.com) - Płatne

Możesz zautomatyzować to zadanie, korzystając z [**DomLink** ](https://github.com/vysecurity/DomLink)(wymaga klucza API whoxy).\
Możesz także automatycznie odkrywać odwrócony whois za pomocą [amass](https://github.com/OWASP/Amass): `amass intel -d tesla.com -whois`

**Zauważ, że tę technikę można wykorzystać do odkrywania kolejnych nazw domen za każdym razem, gdy znajdziesz nową domenę.**

### **Śledzenie**

Jeśli znajdziesz **ten sam identyfikator tego samego śledzenia** na 2 różnych stronach, możesz przypuszczać, że **obie strony** są **zarządzane przez ten sam zespół**.\
Na przykład, jeśli zobaczysz ten sam **identyfikator Google Analytics** lub ten sam **identyfikator Adsense** na kilku stronach.

Istnieją strony i narzędzia, które pozwalają wyszukiwać te śledzacze i więcej:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **Favicon**

Czy wiedziałeś, że możemy znaleźć powiązane domeny i subdomeny naszego celu, szukając tego samego skrótu ikony favicon? Dokładnie to robi narzędzie [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) stworzone przez [@m4ll0k2](https://twitter.com/m4ll0k2). Oto jak go użyć:
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
![favihash - odkryj domeny z tym samym haszem ikony favicon](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

Po prostu mówiąc, favihash pozwoli nam odkryć domeny, które mają ten sam hash ikony favicon co nasz cel.

Co więcej, możesz również wyszukiwać technologie za pomocą hasha favicon, jak wyjaśniono w [**tym wpisie na blogu**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139). Oznacza to, że jeśli znasz **hash faviconu podatnej wersji technologii internetowej**, możesz wyszukać go w shodan i **znaleźć więcej podatnych miejsc**:
```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```
To jest sposób, w jaki możesz **obliczyć skrót favicon** strony internetowej:
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
### **Prawo autorskie / Unikalny ciąg znaków**

Wyszukaj na stronach internetowych **ciągi znaków, które mogą być udostępniane na różnych stronach w tej samej organizacji**. **Ciąg znaków praw autorskich** mógłby być dobrym przykładem. Następnie wyszukaj ten ciąg znaków w **Google**, w innych **przeglądarkach** lub nawet w **shodan**: `shodan search http.html:"Ciąg znaków praw autorskich"`

### **Czas CRT**

To powszechne mieć zadanie cron, takie jak
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
### **Metodologia pasywnego przejęcia**

Wygląda na to, że ludzie często przypisują subdomeny do adresów IP należących do dostawców chmurowych i w pewnym momencie **tracą ten adres IP, ale zapominają usunąć rekord DNS**. Dlatego po prostu **uruchomienie maszyny wirtualnej** w chmurze (np. Digital Ocean) faktycznie **przejmie niektóre subdomeny**.

[**Ten post**](https://kmsec.uk/blog/passive-takeover/) wyjaśnia tę kwestię i proponuje skrypt, który **uruchamia maszynę wirtualną w DigitalOcean**, **pobiera** adres **IPv4** nowej maszyny i **szuka w Virustotal rekordów subdomen** wskazujących na nią.

### **Inne sposoby**

**Zauważ, że tę technikę można wykorzystać do odkrywania kolejnych nazw domen za każdym razem, gdy znajdziesz nową domenę.**

**Shodan**

Ponieważ już znasz nazwę organizacji posiadającej przestrzeń IP, możesz wyszukać te dane w Shodan używając: `org:"Tesla, Inc."` Sprawdź znalezione hosty pod kątem nowych nieoczekiwanych domen w certyfikacie TLS.

Możesz uzyskać dostęp do **certyfikatu TLS** głównej strony internetowej, uzyskać **nazwę organizacji** i następnie wyszukać tę nazwę w **certyfikatach TLS** wszystkich znanych stron internetowych w **Shodan** z filtrem: `ssl:"Tesla Motors"` lub użyć narzędzia takiego jak [**sslsearch**](https://github.com/HarshVaragiya/sslsearch).

**Assetfinder**

[**Assetfinder**](https://github.com/tomnomnom/assetfinder) to narzędzie, które szuka **powiązanych domen** z główną domeną i **ich subdomen**, całkiem niesamowite.

### **Wyszukiwanie podatności**

Sprawdź, czy istnieje możliwość [przejęcia domeny](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover). Być może jakaś firma **używa domeny**, ale **utrata własności**. Po prostu zarejestruj ją (jeśli jest wystarczająco tania) i daj znać firmie.

Jeśli znajdziesz jakąkolwiek **domenę z innym adresem IP** niż te, które już znalazłeś w odkrywaniu zasobów, powinieneś przeprowadzić **podstawowe skanowanie podatności** (za pomocą Nessusa lub OpenVAS) oraz [**skan portów**](../pentesting-network/#discovering-hosts-from-the-outside) za pomocą **nmap/masscan/shodan**. W zależności od tego, jakie usługi są uruchomione, możesz znaleźć w **tej książce kilka sztuczek do "atakowania" ich**.\
_Zauważ, że czasami domena jest hostowana wewnątrz adresu IP, który nie jest kontrolowany przez klienta, więc nie jest to w zakresie, bądź ostrożny._

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**Wskazówka dotycząca bug bounty**: **Zarejestruj się** na platformie **Intigriti**, premium platformie **bug bounty stworzonej przez hakerów, dla hakerów**! Dołącz do nas na [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) już dziś i zacznij zarabiać nagrody do **100 000 USD**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Subdomeny

> Znamy wszystkie firmy w zakresie, wszystkie zasoby każdej firmy i wszystkie powiązane z nimi domeny.

Nadszedł czas, aby znaleźć wszystkie możliwe subdomeny każdej znalezionej domeny.

### **DNS**

Spróbujmy uzyskać **subdomeny** z rekordów **DNS**. Powinniśmy również spróbować **Transferu Strefowego** (jeśli jest podatny, powinieneś to zgłosić).
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

Najszybszym sposobem na uzyskanie wielu subdomen jest wyszukiwanie w źródłach zewnętrznych. Najczęściej używane **narzędzia** to:

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
Istnieją **inne interesujące narzędzia/API**, które, nawet jeśli nie są bezpośrednio specjalizowane w znajdowaniu subdomen, mogą być przydatne do ich znalezienia, takie jak:

* [**Crobat**](https://github.com/cgboal/sonarsearch)**:** Korzysta z interfejsu API [https://sonar.omnisint.io](https://sonar.omnisint.io) do uzyskiwania subdomen
```bash
# Get list of subdomains in output from the API
## This is the API the crobat tool will use
curl https://sonar.omnisint.io/subdomains/tesla.com | jq -r ".[]"
```
* [**Darmowe API JLDC**](https://jldc.me/anubis/subdomains/google.com)
```bash
curl https://jldc.me/anubis/subdomains/tesla.com | jq -r ".[]"
```
* [**RapidDNS**](https://rapiddns.io) darmowe API
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
* [**gau**](https://github.com/lc/gau)**:** pobiera znane adresy URL z AlienVault's Open Threat Exchange, Wayback Machine i Common Crawl dla określonej domeny.
```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```
* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **&** [**subscraper**](https://github.com/Cillian-Collins/subscraper): Przeszukują internet w poszukiwaniu plików JS i wydobywają z nich subdomeny.
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
* [**Censys subdomain finder**](https://github.com/christophetd/censys-subdomain-finder)
```bash
export CENSYS_API_ID=...
export CENSYS_API_SECRET=...
python3 censys-subdomain-finder.py tesla.com
```
* [**DomainTrail.py**](https://github.com/gatete/DomainTrail)
```bash
python3 DomainTrail.py -d example.com
```
* [**securitytrails.com**](https://securitytrails.com/) ma darmowe API do wyszukiwania subdomen i historii adresów IP
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

Ten projekt oferuje **darmowo wszystkie subdomeny związane z programami bug bounty**. Możesz uzyskać dostęp do tych danych również za pomocą [chaospy](https://github.com/dr-0x0x/chaospy) lub uzyskać dostęp do zakresu używanego przez ten projekt [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

Możesz znaleźć **porównanie** wielu z tych narzędzi tutaj: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **Atak brutalnej siły DNS**

Spróbujmy znaleźć nowe **subdomeny** za pomocą ataku brutalnej siły na serwery DNS, używając możliwych nazw subdomen.

Do tego działania będziesz potrzebować kilku **często używanych list słów dla subdomen, takich jak**:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

A także adresy IP dobrych resolverów DNS. Aby wygenerować listę zaufanych resolverów DNS, możesz pobrać resolverów z [https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) i użyć [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) do ich filtrowania. Lub możesz użyć: [https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

Najbardziej polecane narzędzia do ataku brutalnej siły DNS to:

* [**massdns**](https://github.com/blechschmidt/massdns): To było pierwsze narzędzie, które przeprowadziło skuteczny atak brutalnej siły DNS. Jest bardzo szybkie, ale podatne na fałszywe wyniki dodatnie.
```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```
* [**gobuster**](https://github.com/OJ/gobuster): Myślę, że ten używa tylko 1 resolver'a
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) jest nakładką na `massdns`, napisaną w języku go, która pozwala na wyliczenie prawidłowych subdomen za pomocą aktywnego bruteforce, a także rozwiązywanie subdomen z obsługą symboli wieloznacznych i łatwym wsparciem wejścia-wyjścia.
```
shuffledns -d example.com -list example-subdomains.txt -r resolvers.txt
```
* [**puredns**](https://github.com/d3mondev/puredns): Wykorzystuje również `massdns`.
```
puredns bruteforce all.txt domain.com
```
* [**aiodnsbrute**](https://github.com/blark/aiodnsbrute) używa asyncio do asynchronicznego brutalnego testowania nazw domen.
```
aiodnsbrute -r resolvers -w wordlist.txt -vv -t 1024 domain.com
```
### Druga runda brutalnej siły DNS

Po znalezieniu subdomen za pomocą otwartych źródeł i brutalnej siły, można wygenerować zmiany znalezionych subdomen, aby spróbować znaleźć jeszcze więcej. Kilka narzędzi jest przydatnych do tego celu:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** Dla domen i subdomen generuje permutacje.
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): Dla podanych domen i subdomen generuje permutacje.
* Możesz pobrać listę permutacji **wordlist** dla goaltdns [**tutaj**](https://github.com/subfinder/goaltdns/blob/master/words.txt).
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** Mając domeny i subdomeny generuj permutacje. Jeśli nie zostanie wskazany plik permutacji, gotator użyje własnego.
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): Oprócz generowania permutacji subdomen, może również próbować je rozwiązać (ale lepiej jest użyć wcześniej skomentowanych narzędzi).
* Możesz pobrać listę słów do permutacji **altdns** [**tutaj**](https://github.com/infosec-au/altdns/blob/master/words.txt).
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): Kolejne narzędzie do wykonywania permutacji, mutacji i zmian poddomen. To narzędzie będzie próbować siłowo wynik (nie obsługuje dzikich kart DNS).
* Możesz pobrać listę słów permutacji dmut [**tutaj**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt).
```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```
* [**alterx**](https://github.com/projectdiscovery/alterx)**:** Na podstawie domeny **generuje nowe potencjalne nazwy subdomen** na podstawie określonych wzorców, aby odkryć więcej subdomen.

#### Inteligentna generacja permutacji

* [**regulator**](https://github.com/cramppet/regulator): Aby uzyskać więcej informacji, przeczytaj ten [**post**](https://cramppet.github.io/regulator/index.html), ale w skrócie pobierze **główne części** z **odkrytych subdomen** i połączy je, aby znaleźć więcej subdomen.
```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ to fuzzer siły brutalnej subdomen połączony z niezwykle prostym, ale skutecznym algorytmem prowadzonym przez odpowiedź DNS. Wykorzystuje dostarczony zestaw danych wejściowych, takich jak dostosowana lista słów lub historyczne rekordy DNS/TLS, aby dokładnie syntetyzować więcej odpowiadających nazw domen i rozwijać je jeszcze bardziej w pętli na podstawie zebranych informacji podczas skanowania DNS.
```
echo www | subzuf facebook.com
```
### **Workflow Odkrywania Subdomen**

Sprawdź ten wpis na blogu, który napisałem o tym, jak **zautomatyzować odkrywanie subdomen** z domeny przy użyciu **najlepszych workflowów**, aby nie musieć ręcznie uruchamiać wielu narzędzi na swoim komputerze:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / Wirtualne Hosty**

Jeśli znalazłeś adres IP zawierający **jedną lub kilka stron internetowych** należących do subdomen, możesz spróbować **znaleźć inne subdomeny z witrynami na tym IP**, szukając w **źródłach OSINT** domen na danym IP lub **próbując siłowo nazwy domen VHost na tym IP**.

#### OSINT

Możesz znaleźć niektóre **VHosty w IP za pomocą** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **lub innych interfejsów API**.

**Atak Siłowy**

Jeśli podejrzewasz, że pewna subdomena może być ukryta na serwerze sieciowym, możesz spróbować ją siłowo odszukać:
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
Z tą techniką możesz nawet uzyskać dostęp do wewnętrznych/ukrytych punktów końcowych.
{% endhint %}

### **CORS Brute Force**

Czasami znajdziesz strony, które zwracają nagłówek _**Access-Control-Allow-Origin**_ tylko wtedy, gdy prawidłowa domena/poddomena jest ustawiona w nagłówku _**Origin**_. W tych scenariuszach możesz wykorzystać to zachowanie do **odkrywania** nowych **poddomen**.
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### **Atak siłowy na kubełki**

Podczas poszukiwania **poddomen** zwróć uwagę, czy nie wskazują one na jakikolwiek rodzaj **kubełka**, a w takim przypadku [**sprawdź uprawnienia**](../../network-services-pentesting/pentesting-web/buckets/)**.**\
Ponadto, w tym momencie będąc już zaznajomionym ze wszystkimi domenami w zakresie, spróbuj [**przeprowadzić atak siłowy na możliwe nazwy kubełków i sprawdź uprawnienia**](../../network-services-pentesting/pentesting-web/buckets/).

### **Monitorowanie**

Możesz **monitorować** tworzenie się **nowych poddomen** domeny, monitorując **logi Transparentności Certyfikatów** [**sublert** ](https://github.com/yassineaboukir/sublert/blob/master/sublert.py) robi to za Ciebie.

### **Poszukiwanie podatności**

Sprawdź możliwe [**przejęcia poddomen**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover).\
Jeśli **poddomena** wskazuje na **kubełek S3**, [**sprawdź uprawnienia**](../../network-services-pentesting/pentesting-web/buckets/).

Jeśli znajdziesz jakąkolwiek **poddomenę z innym adresem IP** niż te, które już znalazłeś podczas odkrywania zasobów, powinieneś przeprowadzić **podstawowe skanowanie podatności** (używając Nessusa lub OpenVAS) oraz [**skan portów**](../pentesting-network/#discovering-hosts-from-the-outside) przy użyciu **nmap/masscan/shodan**. W zależności od tego, jakie usługi są uruchomione, możesz znaleźć w **tej książce kilka sztuczek do ich "atakowania"**.\
_Zauważ, że czasami poddomena jest hostowana pod adresem IP, który nie jest kontrolowany przez klienta, więc nie jest w zakresie, bądź ostrożny._

## Adresy IP

W początkowych krokach możesz **znaleźć pewne zakresy IP, domeny i poddomeny**.\
Nadszedł czas, aby **zbierać wszystkie adresy IP z tych zakresów** oraz dla **domen/poddomen (zapytania DNS).**

Korzystając z usług poniższych **darmowych interfejsów API**, możesz również znaleźć **poprzednie adresy IP używane przez domeny i poddomeny**. Te adresy IP mogą nadal należeć do klienta (i mogą pozwolić Ci na znalezienie [**obejść CloudFlare**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md))

* [**https://securitytrails.com/**](https://securitytrails.com/)

Możesz również sprawdzić domeny wskazujące na określony adres IP za pomocą narzędzia [**hakip2host**](https://github.com/hakluke/hakip2host)

### **Poszukiwanie podatności**

**Skanuj porty wszystkich adresów IP, które nie należą do CDN** (ponieważ prawdopodobnie nie znajdziesz tam nic interesującego). W odkrytych uruchomionych usługach możesz **znaleźć podatności**.

Znajdź **przewodnik** [**tutaj**](../pentesting-network/) **o tym, jak skanować hosty.**

## Polowanie na serwery WWW

> Znaleźliśmy wszystkie firmy i ich zasoby oraz znamy zakresy IP, domeny i poddomeny w zakresie. Czas poszukać serwerów WWW.

W poprzednich krokach prawdopodobnie już przeprowadziłeś pewne **rozpoznanie odkrytych adresów IP i domen**, więc możesz **już znaleźć wszystkie możliwe serwery WWW**. Jeśli jednak nie, teraz zobaczymy kilka **szybkich sztuczek do wyszukiwania serwerów WWW** w zakresie.

Należy zauważyć, że będzie to **zorientowane na odkrywanie aplikacji internetowych**, dlatego powinieneś również **przeprowadzić skanowanie podatności** i **portów** również (**jeśli zezwala na to** zakres).

**Szybka metoda** odkrywania **otwartych portów** związanych z **serwerami WWW** za pomocą [**masscan można znaleźć tutaj**](../pentesting-network/#http-port-discovery).\
Innym przyjaznym narzędziem do poszukiwania serwerów WWW jest [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) i [**httpx**](https://github.com/projectdiscovery/httpx). Wystarczy przekazać listę domen, a narzędzie spróbuje połączyć się z portem 80 (http) i 443 (https). Dodatkowo, można wskazać próbę połączenia z innymi portami:
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
### **Zrzuty ekranu**

Teraz, gdy odkryłeś **wszystkie serwery sieciowe** obecne w zakresie (wśród **adresów IP** firmy oraz wszystkich **domen** i **poddomen**), prawdopodobnie **nie wiesz, od czego zacząć**. Dlatego zróbmy to prosto i zacznijmy od zrobienia zrzutów ekranu wszystkich z nich. Już tylko **spojrzenie** na **stronę główną** może ujawnić **dziwne** punkty końcowe, które są bardziej **podatne** na **zagrożenia**.

Aby wykonać proponowany pomysł, możesz użyć [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/) lub [**webscreenshot**](https://github.com/maaaaz/webscreenshot)**.**

Ponadto, możesz użyć [**eyeballer**](https://github.com/BishopFox/eyeballer), aby przejrzeć wszystkie **zrzuty ekranu** i powiedzieć Ci, co **najprawdopodobniej zawiera podatności**, a co nie.

## Zasoby publiczne w chmurze

Aby znaleźć potencjalne zasoby chmurowe należące do firmy, powinieneś **zaczynać od listy słów kluczowych identyfikujących tę firmę**. Na przykład, dla firmy kryptowalutowej możesz użyć słów takich jak: `"crypto", "wallet", "dao", "<nazwa_domeny>", <"nazwy_poddomen">`.

Będziesz także potrzebował list słów **często używanych w kubełkach**:

* [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
* [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
* [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

Następnie, z tymi słowami powinieneś generować **permutacje** (sprawdź [**Druga runda Brute-Force DNS**](./#second-dns-bruteforce-round) po więcej informacji).

Z uzyskanych list słów możesz użyć narzędzi takich jak [**cloud\_enum**](https://github.com/initstring/cloud\_enum)**,** [**CloudScraper**](https://github.com/jordanpotti/CloudScraper)**,** [**cloudlist**](https://github.com/projectdiscovery/cloudlist) **lub** [**S3Scanner**](https://github.com/sa7mon/S3Scanner)**.**

Pamiętaj, że szukając Zasobów Chmurowych powinieneś **szukać więcej niż tylko kubełków w AWS**.

### **Szukanie podatności**

Jeśli znajdziesz rzeczy takie jak **otwarte kubełki lub narażone funkcje chmurowe**, powinieneś **uzyskać do nich dostęp** i spróbować zobaczyć, co oferują i czy można je wykorzystać.

## Emaile

Dzięki **domenom** i **poddomenom** w zakresie masz praktycznie wszystko, czego potrzebujesz, aby zacząć szukać adresów e-mail. Oto **API** i **narzędzia**, które najlepiej sprawdziły się dla mnie w poszukiwaniu adresów e-mail firmy:

* [**theHarvester**](https://github.com/laramies/theHarvester) - z użyciem API
* API [**https://hunter.io/**](https://hunter.io/) (wersja darmowa)
* API [**https://app.snov.io/**](https://app.snov.io/) (wersja darmowa)
* API [**https://minelead.io/**](https://minelead.io/) (wersja darmowa)

### **Szukanie podatności**

Emaile przydadzą się później do **brute-force'owania logowań na stronach internetowych i usługach uwierzytelniania** (takich jak SSH). Są również potrzebne do **phishingu**. Ponadto te API dostarczą Ci jeszcze więcej **informacji o osobie** za adresem e-mail, co jest przydatne dla kampanii phishingowej.

## Wycieki poświadczeń

Dzięki **domenom**, **poddomenom** i **emailom** możesz zacząć szukać wycieków poświadczeń z przeszłości należących do tych emaili:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **Szukanie podatności**

Jeśli znajdziesz **ważne wycieki** poświadczeń, to bardzo łatwe zwycięstwo.

## Wycieki sekretów

Wycieki poświadczeń są związane z atakami na firmy, w których **wyciekły i zostały sprzedane poufne informacje**. Jednak firmy mogą być dotknięte również przez **inne wycieki**, których informacje nie znajdują się w tych bazach danych:

### Wycieki z Githuba

Poświadczenia i interfejsy API mogą wyciekać w **publicznych repozytoriach** **firmy** lub **użytkowników** pracujących dla tej firmy na Githubie.\
Możesz użyć narzędzia [**Leakos**](https://github.com/carlospolop/Leakos), aby **pobrać** wszystkie **publiczne repozytoria** organizacji i jej deweloperów i automatycznie uruchomić [**gitleaks**](https://github.com/zricethezav/gitleaks) na nich.

**Leakos** można również użyć do uruchomienia **gitleaks** na wszystkich **tekstach** dostarczonych **URL-ach przekazanych** do niego, ponieważ czasami **strony internetowe również zawierają sekrety**.

#### Dorki Githuba

Sprawdź także tę **stronę** w poszukiwaniu potencjalnych **dorków Githuba**, których również możesz szukać w atakowanej organizacji:

{% content-ref url="github-leaked-secrets.md" %}
[github-leaked-secrets.md](github-leaked-secrets.md)
{% endcontent-ref %}

### Wycieki wklejek

Czasami atakujący lub po prostu pracownicy **publikują treści firmy na stronach do wklejania**. Mogą one zawierać **informacje poufne** lub nie, ale jest bardzo interesujące je wyszukiwać.\
Możesz użyć narzędzia [**Pastos**](https://github.com/carlospolop/Pastos), aby jednocześnie przeszukać ponad 80 stron do wklejania.

### Dorki Google

Stare, ale złote dorki Google zawsze są przydatne do znalezienia **ujawnionych informacji, które tam nie powinny być**. Jedynym problemem jest to, że [**baza danych google-hacking**](https://www.exploit-db.com/google-hacking-database) zawiera kilka **tysięcy** możliwych zapytań, których nie można uruchomić ręcznie. Dlatego możesz wybrać swoje ulubione 10 lub skorzystać z **narzędzia takiego jak** [**Gorks**](https://github.com/carlospolop/Gorks) **do uruchomienia ich wszystkich**.

_Zauważ, że narzędzia, które oczekują uruchomienia całej bazy danych za pomocą zwykłej przeglądarki Google, nigdy się nie zakończą, ponieważ Google bardzo szybko Cię zablokuje._

### **Szukanie podatności**

Jeśli znajdziesz **ważne wycieki** poświadczeń lub tokenów API, to bardzo łatwe zwycięstwo.

## Publiczne podatności kodu

Jeśli odkryłeś, że firma ma **kod open-source**, możesz go **analizować** i szukać w nim **podatności**.

W zależności od języka istnieją różne **narzędzia**, których możesz użyć:

{% content-ref url="../../network-services-pentesting/pentesting-web/code-review-tools.md" %}
[code-review-tools.md](../../network-services-pentesting/pentesting-web/code-review-tools.md)
{% endcontent-ref %}

Istnieją również bezpłatne usługi, które pozwalają **skanować publiczne repozytoria**, takie jak:

* [**Snyk**](https://app.snyk.io/)
## [**Metodologia testowania penetracyjnego aplikacji internetowych**](../../network-services-pentesting/pentesting-web/)

**Większość podatności** znalezionych przez łowców błędów znajduje się w **aplikacjach internetowych**, dlatego chciałbym teraz omówić **metodologię testowania aplikacji internetowych**, którą można [**znaleźć tutaj**](../../network-services-pentesting/pentesting-web/).

Chciałbym również wspomnieć o sekcji [**Otwarte narzędzia skanujące automatycznie strony internetowe**](../../network-services-pentesting/pentesting-web/#automatic-scanners), ponieważ, chociaż nie powinieneś oczekiwać, że znajdą one bardzo wrażliwe podatności, są one przydatne do implementacji w **przepływach pracy w celu uzyskania początkowych informacji o stronie internetowej.**

## Podsumowanie

> Gratulacje! W tym momencie już przeprowadziłeś **całą podstawową enumerację**. Tak, jest to podstawowe, ponieważ można wykonać znacznie więcej operacji enumeracji (zobaczymy więcej sztuczek później).

Więc już:

1. Znalazłeś wszystkie **firmy** w zakresie
2. Znalazłeś wszystkie **zasoby** należące do firm (i przeprowadziłeś skanowanie podatności, jeśli jest w zakresie)
3. Znalazłeś wszystkie **domeny** należące do firm
4. Znalazłeś wszystkie **subdomeny** domen (jakiekolwiek przejęcie subdomeny?)
5. Znalazłeś wszystkie **adresy IP** (z i **bez CDN**) w zakresie.
6. Znalazłeś wszystkie **serwery internetowe** i wykonałeś **zrzut ekranu** z nich (czy coś dziwnego warte głębszego zbadania?)
7. Znalazłeś wszystkie **potencjalne publiczne zasoby w chmurze** należące do firmy.
8. **Emaile**, **wycieki poświadczeń** i **wycieki tajemnic**, które mogą dać Ci **łatwe zwycięstwo**.
9. **Testowanie penetracyjne wszystkich stron internetowych, które znalazłeś**

## **Pełne narzędzia automatyczne do rozpoznawania**

Istnieje wiele narzędzi, które wykonają część proponowanych działań w określonym zakresie.

* [**https://github.com/yogeshojha/rengine**](https://github.com/yogeshojha/rengine)
* [**https://github.com/j3ssie/Osmedeus**](https://github.com/j3ssie/Osmedeus)
* [**https://github.com/six2dez/reconftw**](https://github.com/six2dez/reconftw)
* [**https://github.com/hackerspider1/EchoPwn**](https://github.com/hackerspider1/EchoPwn) - Trochę przestarzałe i nieaktualizowane

## **Referencje**

* Wszystkie darmowe kursy od [**@Jhaddix**](https://twitter.com/Jhaddix) takie jak [**Metodologia łowcy błędów v4.0 - Edycja rozpoznawania**](https://www.youtube.com/watch?v=p4JgIu1mceI)

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Jeśli interesuje Cię **kariera w dziedzinie hakowania** i hakowanie niemożliwych do zhakowania - **rekrutujemy!** (_wymagana biegła znajomość języka polskiego w mowie i piśmie_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>Zdobądź wiedzę na temat hakowania AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF** sprawdź [**PLAN SUBSKRYPCYJNY**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**Grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
