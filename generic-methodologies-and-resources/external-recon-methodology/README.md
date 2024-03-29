# Зовнішня методологія реконструкції

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Якщо вас цікавить **кар'єра в хакінгу** та взламати невзламне - **ми шукаємо співробітників!** (_вимагається вільне володіння польською мовою, як письмово, так і усно_).

{% embed url="https://www.stmcyber.com/careers" %}

## Виявлення активів

> Вам сказали, що все, що належить якій-небудь компанії, входить в область дослідження, і ви хочете з'ясувати, що саме належить цій компанії.

Метою цієї фази є отримання всіх **компаній, які належать основній компанії**, а потім всіх **активів** цих компаній. Для цього ми збираємося:

1. Знайти придбання основної компанії, це дозволить нам визначити компанії в області дослідження.
2. Знайти ASN (якщо є) кожної компанії, це дозволить нам отримати діапазони IP-адрес, які належать кожній компанії.
3. Використовувати зворотні пошукові запити whois для пошуку інших записів (назв організацій, доменів...) пов'язаних з першим (це можна робити рекурсивно).
4. Використовувати інші техніки, такі як фільтри shodan `org` та `ssl`, для пошуку інших активів (трюк з `ssl` можна виконувати рекурсивно).

### **Придбання**

По-перше, нам потрібно знати, які **інші компанії належать основній компанії**.\
Один з варіантів - відвідати [https://www.crunchbase.com/](https://www.crunchbase.com), **знайти** **основну компанію** та **клацнути** на "**придбання**". Там ви побачите інші компанії, які були придбані основною.\
Інший варіант - відвідати сторінку **Wikipedia** основної компанії та шукати **придбання**.

> Ок, на цьому етапі ви повинні знати всі компанії в області дослідження. Давайте з'ясуємо, як знайти їх активи.

### **ASN**

Номер автономної системи (**ASN**) - це **унікальний номер**, призначений **автономній системі** (AS) **Internet Assigned Numbers Authority (IANA)**.\
**AS** складається з **блоків** **IP-адрес**, які мають чітко визначену політику доступу до зовнішніх мереж та адмініструються однією організацією, але можуть складатися з кількох операторів.

Цікаво дізнатися, чи має компанія призначений який-небудь ASN, щоб знайти її **діапазони IP-адрес**. Буде корисно провести **вразливість тестування** проти всіх **хостів** в межах **області дослідження** та шукати **домени** в цих IP-адресах.\
Ви можете **шукати** за назвою компанії, за **IP** або за **доменом** на [**https://bgp.he.net/**](https://bgp.he.net)**.**\
**Залежно від регіону компанії ці посилання можуть бути корисними для збору додаткових даних:** [**AFRINIC**](https://www.afrinic.net) **(Африка),** [**Arin**](https://www.arin.net/about/welcome/region/)**(Північна Америка),** [**APNIC**](https://www.apnic.net) **(Азія),** [**LACNIC**](https://www.lacnic.net) **(Латинська Америка),** [**RIPE NCC**](https://www.ripe.net) **(Європа). У будь-якому випадку, ймовірно, усю** корисну інформацію **(діапазони IP та Whois)** вже містить перше посилання.
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
Також, перелік піддоменів [**BBOT**](https://github.com/blacklanternsecurity/bbot) автоматично агрегує та узагальнює ASNs в кінці сканування.
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
Ви можете знайти діапазони IP організації також за допомогою [http://asnlookup.com/](http://asnlookup.com) (в нього є безкоштовний API).\
Ви можете знайти IP та ASN домену за допомогою [http://ipv4info.com/](http://ipv4info.com).

### **Пошук вразливостей**

На цьому етапі ми знаємо **всі активи в межах обсягу**, тому, якщо вам дозволено, ви можете запустити деякі **сканери вразливостей** (Nessus, OpenVAS) на всіх хостах.\
Також, ви можете запустити деякі [**скани портів**](../pentesting-network/#discovering-hosts-from-the-outside) **або використовувати сервіси, такі як** shodan **для пошуку** відкритих портів **і в залежності від того, що ви знайдете, вам слід** переглянути цю книгу, щоб дізнатися, як тестувати на проникнення кілька можливих служб, які працюють.\
**Також, варто зазначити, що ви також можете підготувати деякі** списки стандартних імен користувачів **та** паролів **і спробувати** перебрати служби за допомогою [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray).

## Домени

> Ми знаємо всі компанії в межах обсягу та їх активи, час знайти домени в межах обсягу.

_Зверніть увагу, що в запропонованих техніках ви також можете знайти піддоменини, і цю інформацію не слід недооцінювати._

Спочатку вам слід шукати **основний домен**(и) кожної компанії. Наприклад, для _Tesla Inc._ це буде _tesla.com_.

### **Reverse DNS**

Оскільки ви знайшли всі діапазони IP доменів, ви можете спробувати виконати **обернені DNS-запити** на цих **IP, щоб знайти більше доменів в межах обсягу**. Спробуйте використати деякий DNS-сервер жертви або деякий відомий DNS-сервер (1.1.1.1, 8.8.8.8)
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
Для цього адміністратор повинен вручну увімкнути PTR.\
Ви також можете скористатися онлайн-інструментом для цієї інформації: [http://ptrarchive.com/](http://ptrarchive.com)

### **Зворотній Whois (цикл)**

У **whois** ви можете знайти багато цікавої **інформації**, такої як **назва організації**, **адреса**, **електронні адреси**, номери телефонів... Але ще цікавіше те, що ви можете знайти **більше активів, пов'язаних з компанією**, якщо ви виконаєте **зворотні пошуки whois за будь-якими з цих полів** (наприклад, інші реєстри whois, де з'являється та ж електронна адреса).\
Ви можете скористатися онлайн-інструментами, такими як:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **Безкоштовно**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **Безкоштовно**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **Безкоштовно**
* [https://www.whoxy.com/](https://www.whoxy.com) - **Безкоштовно** веб, не безкоштовний API.
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - Не безкоштовно
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - Не безкоштовно (лише **100 безкоштовних** пошуків)
* [https://www.domainiq.com/](https://www.domainiq.com) - Не безкоштовно

Ви можете автоматизувати це завдання, використовуючи [**DomLink** ](https://github.com/vysecurity/DomLink)(потрібен ключ API whoxy).\
Ви також можете виконати автоматичне виявлення зворотнього whois за допомогою [amass](https://github.com/OWASP/Amass): `amass intel -d tesla.com -whois`

**Зверніть увагу, що ви можете використовувати цю техніку, щоб відкривати більше доменних імен кожного разу, коли ви знаходите новий домен.**

### **Трекери**

Якщо ви знаходите **той самий ідентифікатор того ж трекера** на 2 різних сторінках, ви можете припустити, що **обидві сторінки** керуються **тією ж командою**.\
Наприклад, якщо ви бачите той самий **ідентифікатор Google Analytics** або той самий **ідентифікатор Adsense** на кількох сторінках.

Є деякі сторінки та інструменти, які дозволяють вам шукати за цими трекерами та більше:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **Favicon**

Чи знали ви, що ми можемо знайти пов'язані домени та піддомени до нашої цілі, шукаючи той самий хеш значка favicon? Саме це робить інструмент [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py), створений [@m4ll0k2](https://twitter.com/m4ll0k2). Ось як його використовувати:
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
![favihash - відкрийте домени з тим самим хешем іконки favicon](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

Просто кажучи, favihash дозволить нам відкрити домени, які мають той самий хеш іконки favicon, що й наша ціль.

Більше того, ви також можете шукати технології, використовуючи хеш favicon, як пояснено в [**цьому дописі у блозі**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139). Це означає, що якщо ви знаєте **хеш favicon вразливої версії веб-технології**, ви можете шукати його в shodan і **знайти більше вразливих місць**:
```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```
Ось як ви можете **обчислити хеш favicon** веб-сайту:
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
### **Авторське право / Унікальний рядок**

Шукайте на веб-сторінках **рядки, які можуть бути спільними для різних веб-сайтів в одній організації**. **Рядок авторського права** може бути хорошим прикладом. Потім шукайте цей рядок в **Google**, в інших **браузерах** або навіть в **Shodan**: `shodan search http.html:"Рядок авторського права"`

### **Час CRT**

Зазвичай використовується cron-завдання, таке як
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
### **Пасивне захоплення**

Здається, що люди часто призначають піддомени IP-адресам, які належать постачальникам хмарних послуг і, в певний момент, **втрачають цей IP-адресу, але забувають видалити запис DNS**. Тому, просто **запускаючи віртуальну машину** в хмарі (наприклад, Digital Ocean), ви фактично **захоплюєте деякі піддомени**.

[**У цьому пості**](https://kmsec.uk/blog/passive-takeover/) пояснюється історія про це та пропонується скрипт, який **запускає віртуальну машину в DigitalOcean**, **отримує** IPv4 нової машини та **шукає в Virustotal записи піддоменів**, які на нього вказують.

### **Інші способи**

**Зверніть увагу, що ви можете використовувати цю техніку, щоб виявляти більше доменних імен кожного разу, коли ви знаходите новий домен.**

**Shodan**

Оскільки ви вже знаєте назву організації, яка володіє IP-простором, ви можете шукати за цими даними в Shodan, використовуючи: `org:"Tesla, Inc."` Перевірте знайдені хости на наявність нових неочікуваних доменів у TLS-сертифікаті.

Ви можете отримати доступ до **TLS-сертифікату** основної веб-сторінки, отримати **назву організації** і потім шукати цю назву в **TLS-сертифікатах** всіх веб-сторінок, відомих Shodan, з фільтром: `ssl:"Tesla Motors"` або використовувати інструмент, такий як [**sslsearch**](https://github.com/HarshVaragiya/sslsearch).

**Assetfinder**

[**Assetfinder**](https://github.com/tomnomnom/assetfinder) - це інструмент, який шукає **домени**, пов'язані з основним доменом та їх **піддомени**, досить дивовижний.

### **Пошук вразливостей**

Перевірте на **захоплення домену**. Можливо, якась компанія **використовує домен**, але **втратила власність**. Просто зареєструйте його (якщо це дешево) і повідомте компанію.

Якщо ви знаходите будь-який **домен з іншим IP-адресою**, ніж ті, які ви вже знайшли під час виявлення активів, вам слід виконати **базове сканування вразливостей** (використовуючи Nessus або OpenVAS) та деяке [**сканування портів**](../pentesting-network/#discovering-hosts-from-the-outside) з **nmap/masscan/shodan**. Залежно від того, які служби працюють, ви можете знайти в **цій книзі деякі хитрощі для їх "атаки"**.\
_Зверніть увагу, що іноді домен розміщується на IP-адресі, який не контролюється клієнтом, тому він не входить в область видимості, будьте обережні._

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**Порада з баг-баунті**: **зареєструйтесь** на **Intigriti**, преміальну **платформу для баг-баунті, створену хакерами для хакерів**! Приєднуйтесь до нас на [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) сьогодні та почніть заробляти винагороди до **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Піддомени

> Ми знаємо всі компанії в межах області, всі активи кожної компанії та всі домени, пов'язані з компаніями.

Час знайти всі можливі піддомени кожного знайденого домену.

### **DNS**

Давайте спробуємо отримати **піддомени** з **DNS**-записів. Ми також повинні спробувати **зонний трансфер** (якщо вразливий, ви повинні повідомити про це).
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

Найшвидший спосіб отримати багато піддоменів - це шукати в зовнішніх джерелах. Найбільш використовувані **інструменти** наступні (для кращих результатів налаштуйте ключі API):

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
Є **інші цікаві інструменти/API**, які, навіть якщо не спеціалізовані на пошук піддоменів, можуть бути корисними для їх знаходження, наприклад:

* [**Crobat**](https://github.com/cgboal/sonarsearch)**:** Використовує API [https://sonar.omnisint.io](https://sonar.omnisint.io) для отримання піддоменів
```bash
# Get list of subdomains in output from the API
## This is the API the crobat tool will use
curl https://sonar.omnisint.io/subdomains/tesla.com | jq -r ".[]"
```
* [**Безкоштовний API JLDC**](https://jldc.me/anubis/subdomains/google.com)
```bash
curl https://jldc.me/anubis/subdomains/tesla.com | jq -r ".[]"
```
* [**RapidDNS**](https://rapiddns.io) безкоштовний API
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
* [**gau**](https://github.com/lc/gau)**:** отримує відомі URL-адреси з AlienVault's Open Threat Exchange, Wayback Machine та Common Crawl для будь-якого заданого домену.
```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```
* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **&** [**subscraper**](https://github.com/Cillian-Collins/subscraper): Вони шукають по мережі JS-файли та витягують з них піддомени.
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
* [**Пошук піддоменів Censys**](https://github.com/christophetd/censys-subdomain-finder)
```bash
export CENSYS_API_ID=...
export CENSYS_API_SECRET=...
python3 censys-subdomain-finder.py tesla.com
```
* [**DomainTrail.py**](https://github.com/gatete/DomainTrail)
```bash
python3 DomainTrail.py -d example.com
```
* [**securitytrails.com**](https://securitytrails.com/) має безкоштовний API для пошуку піддоменів та історії IP
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

Цей проект пропонує **безкоштовно всі піддомени, пов'язані з програмами по виявленню помилок**. Ви також можете отримати доступ до цих даних, використовуючи [chaospy](https://github.com/dr-0x0x/chaospy) або навіть отримати доступ до обсягу, використаного цим проектом [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

Ви можете знайти **порівняння** багатьох з цих інструментів тут: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNS Brute force**

Давайте спробуємо знайти нові **піддомени**, використовуючи перебір DNS-серверів за можливими назвами піддоменів.

Для цієї дії вам знадобляться деякі **загальні списки слів для піддоменів, такі як**:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

А також IP-адреси надійних DNS-резольверів. Щоб згенерувати список довірених DNS-резольверів, ви можете завантажити резольвери з [https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) та використовувати [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) для їх фільтрації. Або ви можете використовувати: [https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

Найбільш рекомендовані інструменти для DNS-перебору:

* [**massdns**](https://github.com/blechschmidt/massdns): Це був перший інструмент, який виконував ефективний перебір DNS. Він дуже швидкий, але схильний до помилкових позитивів.
```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```
* [**gobuster**](https://github.com/OJ/gobuster): Цей, на мою думку, просто використовує 1 резольвер.
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) - це обгортка навколо `massdns`, написана на go, яка дозволяє перерахувати дійсні піддомени за допомогою активного перебору, а також вирішувати піддомени з обробкою джокерів та простою підтримкою введення-виведення.
```
shuffledns -d example.com -list example-subdomains.txt -r resolvers.txt
```
* [**puredns**](https://github.com/d3mondev/puredns): Він також використовує `massdns`.
```
puredns bruteforce all.txt domain.com
```
* [**aiodnsbrute**](https://github.com/blark/aiodnsbrute) використовує asyncio для асинхронного перебору доменних імен.
```
aiodnsbrute -r resolvers -w wordlist.txt -vv -t 1024 domain.com
```
### Другий раунд перебору DNS

Після того, як було знайдено піддомени за допомогою відкритих джерел та перебору, ви можете створити варіації знайдених піддоменів, щоб спробувати знайти ще більше. Декілька інструментів корисні для цієї мети:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** Надає домени та піддомени для генерації перестановок.
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): Наданим доменам та піддоменам генерувати перестановки.
* Ви можете отримати перестановки **wordlist** для goaltdns [**тут**](https://github.com/subfinder/goaltdns/blob/master/words.txt).
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** Заданими доменами та піддоменами генерувати перестановки. Якщо файл перестановок не вказано, gotator використовуватиме власний.
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): Окрім генерації перестановок піддоменів, він також може спробувати їх вирішити (але краще використовувати попередні коментовані інструменти).
* Ви можете отримати **список слів** для перестановок altdns [**тут**](https://github.com/infosec-au/altdns/blob/master/words.txt).
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): Ще один інструмент для виконання перестановок, мутацій та змін піддоменів. Цей інструмент буде перебирати результат (він не підтримує дію DNS-маски).
* Ви можете отримати словник перестановок dmut [**тут**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt).
```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```
* [**alterx**](https://github.com/projectdiscovery/alterx)**:** Заснований на домені, **генерує нові потенційні піддомени** на основі вказаних шаблонів для спроби виявлення більше піддоменів.

#### Генерація розумних перестановок

* [**regulator**](https://github.com/cramppet/regulator): Для отримання додаткової інформації прочитайте цей [**пост**](https://cramppet.github.io/regulator/index.html), але він в основному отримує **основні частини** з **виявлених піддоменів** і змішує їх для пошуку більше піддоменів.
```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ - це фазер грубої сили піддоменів, який поєднується з надзвичайно простим, але ефективним алгоритмом, що керується відповідями DNS. Він використовує набір вхідних даних, такий як налаштований словник або історичні записи DNS/TLS, для точного синтезування відповідних доменних імен та подальшого розширення їх у циклі на основі інформації, зібраної під час скану DNS.
```
echo www | subzuf facebook.com
```
### **Послідовність виявлення піддоменів**

Перевірте цей блог-пост, який я написав про те, як **автоматизувати виявлення піддоменів** з домену за допомогою **робочих процесів Trickest**, щоб мені не потрібно було запускати вручну купу інструментів на своєму комп'ютері:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / Віртуальні хости**

Якщо ви знайшли IP-адресу, що містить **одну або кілька веб-сторінок**, належних до піддоменів, ви можете спробувати **знайти інші піддомени з веб-сторінками на цьому IP**, шукаючи в **джерелах OSINT** доменів на IP або **перебираючи імена доменів VHost на цьому IP**.

#### OSINT

Ви можете знайти деякі **VHosts в IP, використовуючи** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **або інші API**.

**Перебірка**

Якщо ви підозрюєте, що деякий піддомен може бути схований на веб-сервері, ви можете спробувати перебрати його:
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
За допомогою цієї техніки ви навіть можете мати доступ до внутрішніх/прихованих кінцевих точок.
{% endhint %}

### **CORS Brute Force**

Іноді ви знайдете сторінки, які повертають лише заголовок _**Access-Control-Allow-Origin**_ коли дійсний домен/піддомен встановлено в заголовку _**Origin**_. У таких сценаріях ви можете зловживати цією поведінкою для **виявлення** нових **піддоменів**.
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### **Перебір кішок**

Під час пошуку **піддоменів** слід уважно стежити, чи вони **вказують** на будь-який тип **кішки**, і у цьому випадку [**перевірте дозволи**](../../network-services-pentesting/pentesting-web/buckets/)**.**\
Також, оскільки на цьому етапі ви будете знати всі домени в межах обсягу, спробуйте [**перебрати можливі назви кішок та перевірити дозволи**](../../network-services-pentesting/pentesting-web/buckets/).

### **Моніторинг**

Ви можете **моніторити**, чи створюються **нові піддомени** домену, моніторивши **логи Transparent Certificate** [**sublert** ](https://github.com/yassineaboukir/sublert/blob/master/sublert.py).

### **Пошук вразливостей**

Перевірте можливі [**захоплення піддоменів**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover).\
Якщо **піддомен** вказує на деяку **кішку S3**, [**перевірте дозволи**](../../network-services-pentesting/pentesting-web/buckets/).

Якщо ви знаходите будь-який **піддомен з IP-адресою, відмінною** від тих, які ви вже знайшли під час виявлення активів, вам слід виконати **базове сканування вразливостей** (використовуючи Nessus або OpenVAS) та деяке [**сканування портів**](../pentesting-network/#discovering-hosts-from-the-outside) з **nmap/masscan/shodan**. Залежно від того, які служби працюють, ви можете знайти в **цій книзі деякі хитрощі для їх "атаки"**.\
_Зверніть увагу, що іноді піддомен розміщений на IP-адресі, який не контролюється клієнтом, тому він не входить в обсяг, будьте обережні._

## IP-адреси

На початкових етапах ви, можливо, **знайшли деякі діапазони IP-адрес, домени та піддомени**.\
Час **зібрати всі IP-адреси з цих діапазонів** та для **доменів/піддоменів (DNS-запити).**

Використовуючи послуги наступних **безкоштовних API**, ви також можете знайти **попередні IP-адреси, використані доменами та піддоменами**. Ці IP-адреси можуть все ще належати клієнту (і дозволити вам знайти [**обхід CloudFlare**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md))

* [**https://securitytrails.com/**](https://securitytrails.com/)

Ви також можете перевірити домени, які вказують на певну IP-адресу, використовуючи інструмент [**hakip2host**](https://github.com/hakluke/hakip2host)

### **Пошук вразливостей**

**Сканувати порти всіх IP-адрес, які не належать до CDN** (оскільки ви ймовірно не знайдете там нічого цікавого). У виявлених працюючих службах ви можете **знайти вразливості**.

**Знайдіть** [**посібник**](../pentesting-network/) **щодо того, як сканувати хости.**

## Пошук веб-серверів

> Ми знайшли всі компанії та їх активи, і ми знаємо діапазони IP-адрес, домени та піддомени в межах обсягу. Час шукати веб-сервери.

На попередніх етапах ви, можливо, вже виконали деякий **рекон активів, виявлених IP-адрес та доменів**, тому ви можливо вже знайшли всі можливі веб-сервери. Однак, якщо ви цього не зробили, ми зараз побачимо деякі **швидкі хитрощі для пошуку веб-серверів** в межах обсягу.

Зверніть увагу, що це буде **орієнтовано на виявлення веб-додатків**, тому ви повинні також **виконати сканування вразливостей** та **портів** (**якщо дозволено** в межах обсягу).

**Швидкий метод** для виявлення **відкритих портів**, пов'язаних з **веб-серверами** за допомогою [**masscan можна знайти тут**](../pentesting-network/#http-port-discovery).\
Ще один зручний інструмент для пошуку веб-серверів - [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) та [**httpx**](https://github.com/projectdiscovery/httpx). Ви просто передаєте список доменів, і він спробує підключитися до порту 80 (http) та 443 (https). Додатково ви можете вказати спробувати інші порти:
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
## **Знімки екрану**

Тепер, коли ви виявили **всі веб-сервери**, присутні в області (серед **IP-адрес компанії** та всіх **доменів** та **піддоменів**), ви, можливо, **не знаєте, з чого почати**. Так що давайте спростимо і почнемо просто роблячи знімки екрану всіх них. Просто **подивившись** на **головну сторінку**, ви можете знайти **дивні** кінцеві точки, які більше **схильні до вразливостей**.

Для виконання запропонованої ідеї ви можете використовувати [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/) або [**webscreenshot**](https://github.com/maaaaz/webscreenshot)**.**

Крім того, ви можете використовуват Fameив ** в lookд **т ** в)** [ ** ** the** [ **tingive ** theingлive ** ** theingл ** **tingive ** the ** youingад** ** theingle **tingive ** the** using) ** theingомs ** the** using) ** theingомs **ing such ** theive ** the theau ** theive).esse]s Net contenteryingедs publics ** you for ** you ** theive).esse) ** theinged a п п п п п п п ** you ** the foringодs public theHarvester, API of [**https://hunter.io/**](https://hunter.io/), API of [**https://app.snov.io/**](https://app.snov.io/), API of [**https://minelead.io/**](https://.minelead.io/).

### **Пошук вразливостей**

Якщо ви знаходите такі речі, як **відкриті бакети або викладені хмарні функції**, ви повинні **отримати до них доступ** і спробуват це, що вони вам пропонують і чи можете ви їх зловживати.

## **Електронні листи**

З **доменами** та **піддоменами**, що входять в область, ви в основному маєте все, що вам **потрібно для початку пошуку електронних листів**. Це **API** та **інструменти**, які найкраще працювали для мене для пошуку електронних листів компанії:

* [**theHarvester**](https://github.com/laramies/theHarvester) - з API
* API [**https://hunter.io/**](https://hunter.io/) (безкоштовна версія)
* API [**https://app.snov.io/**](https://app.snov.io/) (безкоштовна версія)
* API [**https://minelead.io/**](https://minelead.io/) (безкоштовна вер

### **Пошук вразливостей**

Електронні листи будуть корисні пізніше для **брутфорсу входу на веб-сайти та служби автентифікації** (такі як SSH). Крім того, вони необхідні для **фішингу**. Крім того, ці за допомогою цих API ви отримаєте ще більше **інформації про особу**, яка стоїть за електронною адресою, що корисно для кампанії з фішингу.

## **Витоки облікових даних**

З **доменами**, **піддоменами** та **електронними листами** ви можете почати шукати витоки облікових даних, які колись належали цим електронним листам:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **Пошук вразливостей**

Якщо ви знаходите **дійсні витікнуті** облікові дані, це дуже легка перемога.

## **Витоки секретів**

Витоки облікових даних пов'язані з взломами компаній, де **витікала і продавалася конфіденційна інформація**. Однак компанії можуть бути пошкоджені **іншими витоками**, інформація про які не міститься в цих базах даних:

### **Витоки Github**

Облікові дані та API можуть витікати в **публічних репозиторіях** **компанії** або **користувачів**, які працюють у цій компанії GitHub.\
Ви можете використовувати **інструмент** [**Leakos**](https://github.com/carlospolop/Leakos) для **завантаження** всіх **публічних репозиторіїв** організації та її **розробників** та запустити [**gitleaks**](https://github.com/zricethezav/gitleaks) автоматично.

**Leakos** також може бути використаний для запуску **gitleaks** проти всіх **текстів**, наданих **URL**, переданих йому, оскільки іноді **веб-сторінки також містять секрети**.

#### **Github Dorks**

Також перевірте цю **сторінку** на потенційні **github dorks**, які ви також можете шукати в організації, яку ви атакуєте:

{% content-ref url="github-leaked-secrets.md" %}
[github-leaked-secrets.md](github-leaked-secrets.md)
{% endcontent-ref %}

### **Витоки Pastes**

Іноді зловмисники або просто працівники будуть **публікувати вміст компанії на сайті для вставки**. Це може містити або не містити **конфіденційну інформацію**, але це дуже цікаво для пошуку.\
Ви можете використовувати інструмент [**Pastos**](https://github.com/carlospolop/Pastos) для пошуку на більш ніж 80 сайтах для вставки одночасно.

### **Google Dorks**

Старі, але золоті google dorks завжди корисні для пошуку **витоків інформації, якої там не повинно бути**. Єдине, що заважає, це те, що [**база даних google-hacking**](https://www.exploit-db.com/google-hacking-database) містить кілька **тисяч** можливих запитів, які ви не можете запустити вручну. Тому ви можете отримати свої улюблені 10 або ви можете використати **інструмент, такий як** [**Gorks**](https://github.com/carlospolop/Gorks) **для запуску їх всіх**.

_Зауважте, що інструменти, які очікують запуску всієї бази даних за допомогою звичайного браузера Google, ніколи не закінчаться, оскільки google дуже швидко заблокує вас._

### **Пошук вразливостей**

Якщо ви знаходите **дійсні витікнуті** облікові дані або токени API, це дуже легка перемога.

## **Вразливості в публічному коді**

Якщо ви виявили, що у компанії є **відкритий вихідний код**, ви можете **аналізувати** його та шукати на ньому **вразливості**.

**Залежно від мови** існують різні **інструменти**, які ви можете використовувати:

{% content-ref url="../../network-services-pentesting/pentesting-web/code-review-tools.md" %}
[code-review-tools.md](../../network-services-pentesting/pentesting-web/code-review-tools.md)
{% endcontent-ref %}

Є також безкоштовні сервіси, які дозволяють вам **сканувати публічні репозиторії**, такі як:

* [**Snyk**](https://app.snyk.io/)
## [**Методологія тестування веб-додатків**](../../network-services-pentesting/pentesting-web/)

**Більшість вразливостей**, виявлених мисливцями за багами, знаходиться всередині **веб-додатків**, тому на цьому етапі я хотів би поговорити про **методологію тестування веб-додатків**, і ви можете [**знайти цю інформацію тут**](../../network-services-pentesting/pentesting-web/).

Також я хочу зробити особливе згадування про розділ [**Інструменти відкритого коду для автоматизованого сканування вебу**](../../network-services-pentesting/pentesting-web/#automatic-scanners), оскільки, хоча ви не повинні очікувати, що вони знайдуть дуже чутливі вразливості, вони дуже зручні для впровадження їх у **робочі процеси для отримання початкової інформації про веб.**

## Підсумок

> Вітаємо! На цьому етапі ви вже виконали **всю базову енумерацію**. Так, це базово, оскільки можна зробити ще багато енумерації (ми побачимо більше хитрощів пізніше).

Тож ви вже:

1. Знайшли всі **компанії** в межах області
2. Знайшли всі **активи**, що належать компаніям (і виконали деяке сканування вразливостей, якщо входить у область)
3. Знайшли всі **домени**, що належать компаніям
4. Знайшли всі **піддомени** доменів (чи є які-небудь захоплення піддоменів?)
5. Знайшли всі **IP-адреси** (з **CDN** та **не з CDN**) в межах області.
6. Знайшли всі **веб-сервери** і зробили **знімок екрана** (чи є щось дивне, варте більш детального розгляду?)
7. Знайшли всі **потенційні публічні активи хмари**, що належать компанії.
8. **Електронні листи**, **витоки облікових даних** та **витоки секретів**, які можуть дати вам **величезну перемогу дуже легко**.
9. **Тестування на проникнення всіх веб-сайтів, які ви знайшли**

## **Повні автоматичні інструменти реконструкції**

Існує кілька інструментів, які виконають частину запропонованих дій щодо визначеної області.

* [**https://github.com/yogeshojha/rengine**](https://github.com/yogeshojha/rengine)
* [**https://github.com/j3ssie/Osmedeus**](https://github.com/j3ssie/Osmedeus)
* [**https://github.com/six2dez/reconftw**](https://github.com/six2dez/reconftw)
* [**https://github.com/hackerspider1/EchoPwn**](https://github.com/hackerspider1/EchoPwn) - Трохи застарілий і не оновлюється

## **Посилання**

* Всі безкоштовні курси від [**@Jhaddix**](https://twitter.com/Jhaddix) такі як [**Методологія мисливця за багами v4.0 - Видання реконструкції**](https://www.youtube.com/watch?v=p4JgIu1mceI)

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Якщо вас цікавить **кар'єра хакера** і взламати невзламне - **ми наймаємо!** (_потрібно вільно володіти польською мовою, як письмово, так і усно_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>Вивчайте взлом AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
