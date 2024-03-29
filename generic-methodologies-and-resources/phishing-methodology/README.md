# Методологія рибалки

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

## Методологія

1. Розвідка жертви
1. Виберіть **домен жертви**.
2. Виконайте деяку базову веб-енумерацію, **шукаючи портали входу**, використовані жертвою, та **вирішіть**, який саме ви будете **імітувати**.
3. Використовуйте деякі **OSINT**, щоб **знайти електронні адреси**.
2. Підготуйте середовище
1. **Купіть домен**, який ви будете використовувати для оцінки рибальства
2. Налаштуйте записи, пов'язані з електронною поштою (SPF, DMARC, DKIM, rDNS)
3. Налаштуйте VPS з **gophish**
3. Підготуйте кампанію
1. Підготуйте **шаблон електронного листа**
2. Підготуйте **веб-сторінку** для крадіжки облікових даних
4. Запустіть кампанію!

## Генерація схожих доменних імен або купівля довіреного домену

### Техніки варіації доменних імен

* **Ключове слово**: Доменне ім'я **містить важливе ключове слово** оригінального домену (наприклад, zelster.com-management.com).
* **Піддомен з дефісом**: Змініть **крапку на дефіс** піддомену (наприклад, www-zelster.com).
* **Новий TLD**: Те ж саме доменне ім'я з використанням **нового TLD** (наприклад, zelster.org)
* **Гомогліф**: Воно **замінює** літеру в доменному імені на **літери, які схожі** (наприклад, zelfser.com).
* **Транспозиція**: Воно **обмінює дві літери** в межах доменного імені (наприклад, zelsetr.com).
* **Однини/Множини**: Додає або видаляє "s" в кінці доменного імені (наприклад, zeltsers.com).
* **Пропуск**: Воно **видаляє одну** з літер з доменного імені (наприклад, zelser.com).
* **Повторення**: Воно **повторює одну** з літер в доменному імені (наприклад, zeltsser.com).
* **Заміна**: Подібно до гомогліфа, але менш приховано. Воно замінює одну з літер в доменному імені, можливо, літерою, що знаходиться поруч з оригінальною літерою на клавіатурі (наприклад, zektser.com).
* **Піддоменоване**: Введіть **крапку** всередині доменного імені (наприклад, ze.lster.com).
* **Вставка**: Воно **вставляє літеру** в доменне ім'я (наприклад, zerltser.com).
* **Відсутність крапки**: Додайте TLD до доменного імені. (наприклад, zelstercom.com)

**Автоматичні Інструменти**

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy)

**Веб-сайти**

* [https://dnstwist.it/](https://dnstwist.it)
* [https://dnstwister.report/](https://dnstwister.report)
* [https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/](https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/)

### Бітовий перевертень

Є **можливість того, що один або деякі біти, збережені або в комунікації, можуть автоматично перевертатися** через різні фактори, такі як сонячні вибухи, космічні промені або помилки обладнання.

Коли цей концепт **застосовується до запитів DNS**, можливо, що **домен, отриманий DNS-сервером**, не співпадає з доменом, який спочатку запитано.

Наприклад, зміна одного біту в домені "windows.com" може змінити його на "windnws.com."

Атакувальники можуть **скористатися цим, зареєструвавши кілька доменів з перевертанням бітів**, які схожі на домен жертви. Їхнім наміром є перенаправлення законних користувачів на свою власну інфраструктуру.

Для отримання додаткової інформації читайте [https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/](https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/)

### Купівля довіреного домену

Ви можете шукати на [https://www.expireddomains.net/](https://www.expireddomains.net) за просроченим доменом, який ви могли б використовувати.\
Щоб переконатися, що просрочений домен, який ви збираєтеся купити, **вже має хороший SEO**, ви можете перевірити, як він категоризований в:

* [http://www.fortiguard.com/webfilter](http://www.fortiguard.com/webfilter)
* [https://urlfiltering.paloaltonetworks.com/query/](https://urlfiltering.paloaltonetworks.com/query/)

## Виявлення електронних адрес

* [https://github.com/laramies/theHarvester](https://github.com/laramies/theHarvester) (100% безкоштовно)
* [https://phonebook.cz/](https://phonebook.cz) (100% безкоштовно)
* [https://maildb.io/](https://maildb.io)
* [https://hunter.io/](https://hunter.io)
* [https://anymailfinder.com/](https://anymailfinder.com)

Для **виявлення більше** дійсних адрес електронної пошти або **перевірки тих**, які ви вже виявили, ви можете перевірити, чи можете ви зламати їх smtp-сервери жертви. [Дізнайтеся, як перевірити/виявити адресу електронної пошти тут](../../network-services-pentesting/pentesting-smtp/#username-bruteforce-enumeration).\
Крім того, не забувайте, що якщо користувачі використовують **будь-який веб-портал для доступу до своїх листів**, ви можете перевірити, чи він вразливий до **брутфорсу імен користувачів**, та використовувати вразливість, якщо це можливо.

## Налаштування GoPhish

### Встановлення

Ви можете завантажити його з [https://github.com/gophish/gophish/releases/tag/v0.11.0](https://github.com/gophish/gophish/releases/tag/v0.11.0)

Завантажте й розпакуйте його всередину `/opt/gophish` та виконайте `/opt/gophish/gophish`\
Ви отримаєте пароль для адміністратора на порту 3333 у виведенні. Тому, зверніться до цього порту та використовуйте ці облікові дані для зміни пароля адміністратора. Можливо, вам знадобиться тунель до локального:
```bash
ssh -L 3333:127.0.0.1:3333 <user>@<ip>
```
### Налаштування

**Налаштування TLS-сертифіката**

Перед цим кроком ви повинні **вже купити домен**, який ви збираєтеся використовувати, і він повинен бути **спрямований** на **IP-адресу VPS**, де ви налаштовуєте **gophish**.
```bash
DOMAIN="<domain>"
wget https://dl.eff.org/certbot-auto
chmod +x certbot-auto
sudo apt install snapd
sudo snap install core
sudo snap refresh core
sudo apt-get remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
certbot certonly --standalone -d "$DOMAIN"
mkdir /opt/gophish/ssl_keys
cp "/etc/letsencrypt/live/$DOMAIN/privkey.pem" /opt/gophish/ssl_keys/key.pem
cp "/etc/letsencrypt/live/$DOMAIN/fullchain.pem" /opt/gophish/ssl_keys/key.crt​
```
**Налаштування пошти**

Почніть встановлення: `apt-get install postfix`

Потім додайте домен до наступних файлів:

* **/etc/postfix/virtual\_domains**
* **/etc/postfix/transport**
* **/etc/postfix/virtual\_regexp**

**Також змініть значення наступних змінних всередині /etc/postfix/main.cf**

`myhostname = <domain>`\
`mydestination = $myhostname, <domain>, localhost.com, localhost`

Нарешті, змініть файли **`/etc/hostname`** та **`/etc/mailname`** на назву вашого домену та **перезапустіть свій VPS.**

Тепер створіть **DNS A запис** `mail.<domain>`, спрямований на **IP-адресу** VPS та **DNS MX запис**, спрямований на `mail.<domain>`

Тепер спробуйте відправити електронного листа:
```bash
apt install mailutils
echo "This is the body of the email" | mail -s "This is the subject line" test@email.com
```
**Налаштування Gophish**

Зупиніть виконання gophish та налаштуйте його.\
Змініть `/opt/gophish/config.json` на наступне (зверніть увагу на використання https):
```bash
{
"admin_server": {
"listen_url": "127.0.0.1:3333",
"use_tls": true,
"cert_path": "gophish_admin.crt",
"key_path": "gophish_admin.key"
},
"phish_server": {
"listen_url": "0.0.0.0:443",
"use_tls": true,
"cert_path": "/opt/gophish/ssl_keys/key.crt",
"key_path": "/opt/gophish/ssl_keys/key.pem"
},
"db_name": "sqlite3",
"db_path": "gophish.db",
"migrations_prefix": "db/db_",
"contact_address": "",
"logging": {
"filename": "",
"level": ""
}
}
```
**Налаштування служби gophish**

Для створення служби gophish, щоб вона автоматично запускалася та керувалася як служба, ви можете створити файл `/etc/init.d/gophish` з наступним вмістом:
```bash
#!/bin/bash
# /etc/init.d/gophish
# initialization file for stop/start of gophish application server
#
# chkconfig: - 64 36
# description: stops/starts gophish application server
# processname:gophish
# config:/opt/gophish/config.json
# From https://github.com/gophish/gophish/issues/586

# define script variables

processName=Gophish
process=gophish
appDirectory=/opt/gophish
logfile=/var/log/gophish/gophish.log
errfile=/var/log/gophish/gophish.error

start() {
echo 'Starting '${processName}'...'
cd ${appDirectory}
nohup ./$process >>$logfile 2>>$errfile &
sleep 1
}

stop() {
echo 'Stopping '${processName}'...'
pid=$(/bin/pidof ${process})
kill ${pid}
sleep 1
}

status() {
pid=$(/bin/pidof ${process})
if [["$pid" != ""| "$pid" != "" ]]; then
echo ${processName}' is running...'
else
echo ${processName}' is not running...'
fi
}

case $1 in
start|stop|status) "$1" ;;
esac
```
Завершіть налаштування служби та перевірте її, виконавши:
```bash
mkdir /var/log/gophish
chmod +x /etc/init.d/gophish
update-rc.d gophish defaults
#Check the service
service gophish start
service gophish status
ss -l | grep "3333\|443"
service gophish stop
```
## Налаштування поштового сервера та домену

### Зачекайте та будьте легітними

Чим старший домен, тим менше ймовірно, що його спіймуть як спам. Тому вам слід зачекати якомога більше часу (принаймні 1 тиждень) перед оцінкою рибалки. Крім того, якщо ви розмістите сторінку про репутаційний сектор, отримана репутація буде кращою.

Зверніть увагу, що навіть якщо вам доведеться зачекати тиждень, ви можете завершити налаштування всього зараз.

### Налаштування оберненого DNS (rDNS)

Встановіть запис оберненого DNS (PTR), який розгадує IP-адресу VPS на доменне ім'я.

### Запис SPF (Sender Policy Framework)

Вам **необхідно налаштувати запис SPF для нового домену**. Якщо ви не знаєте, що таке запис SPF, [**прочитайте цю сторінку**](../../network-services-pentesting/pentesting-smtp/#spf).

Ви можете скористатися [https://www.spfwizard.net/](https://www.spfwizard.net), щоб згенерувати свою політику SPF (використовуйте IP-адресу машини VPS)

![](<../../.gitbook/assets/image (388).png>)

Це вміст, який повинен бути встановлений у запис TXT всередині домену:
```bash
v=spf1 mx a ip4:ip.ip.ip.ip ?all
```
### Запис про автентифікацію, звітність та відповідність повідомлень на основі домену (DMARC)

Вам необхідно **налаштувати запис DMARC для нового домену**. Якщо ви не знаєте, що таке запис DMARC, [**прочитайте цю сторінку**](../../network-services-pentesting/pentesting-smtp/#dmarc).

Вам потрібно створити новий DNS TXT запис, який вказує на ім'я хоста `_dmarc.<домен>` з наступним вмістом:
```bash
v=DMARC1; p=none
```
### DomainKeys Identified Mail (DKIM)

Вам потрібно **налаштувати DKIM для нового домену**. Якщо ви не знаєте, що таке запис DMARC, [**прочитайте цю сторінку**](../../network-services-pentesting/pentesting-smtp/#dkim).

Цей підручник базується на: [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)

{% hint style="info" %}
Вам потрібно конкатенувати обидва значення B64, які генерує ключ DKIM:
```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0wPibdqPtzYk81njjQCrChIcHzxOp8a1wjbsoNtka2X9QXCZs+iXkvw++QsWDtdYu3q0Ofnr0Yd/TmG/Y2bBGoEgeE+YTUG2aEgw8Xx42NLJq2D1pB2lRQPW4IxefROnXu5HfKSm7dyzML1gZ1U0pR5X4IZCH0wOPhIq326QjxJZm79E1nTh3xj" "Y9N/Dt3+fVnIbMupzXE216TdFuifKM6Tl6O/axNsbswMS1TH812euno8xRpsdXJzFlB9q3VbMkVWig4P538mHolGzudEBg563vv66U8D7uuzGYxYT4WS8NVm3QBMg0QKPWZaKp+bADLkOSB9J2nUpk4Aj9KB5swIDAQAB
```
{% endhint %}

### Перевірте бал електронної пошти вашої конфігурації

Ви можете зробити це, використовуючи [https://www.mail-tester.com/](https://www.mail-tester.com)\
Просто перейдіть на сторінку і надішліть електронного листа на адресу, яку вони вам вказують:
```bash
echo "This is the body of the email" | mail -s "This is the subject line" test-iimosa79z@srv1.mail-tester.com
```
Ви також можете **перевірити конфігурацію своєї електронної пошти**, відправивши листа на `check-auth@verifier.port25.com` та **прочитавши відповідь** (для цього вам потрібно **відкрити** порт **25** та переглянути відповідь у файлі _/var/mail/root_, якщо ви відправляєте листа як root).\
Переконайтеся, що ви пройшли всі тести:
```bash
==========================================================
Summary of Results
==========================================================
SPF check:          pass
DomainKeys check:   neutral
DKIM check:         pass
Sender-ID check:    pass
SpamAssassin check: ham
```
Ви також можете **надіслати повідомлення на Gmail, який перебуває під вашим контролем**, і перевірити **заголовки електронної пошти** у своїй скриньці Gmail, `dkim=pass` повинен бути присутній у полі заголовка `Authentication-Results`.
```
Authentication-Results: mx.google.com;
spf=pass (google.com: domain of contact@example.com designates --- as permitted sender) smtp.mail=contact@example.com;
dkim=pass header.i=@example.com;
```
### Видалення з чорного списку Spamhouse

Сторінка [www.mail-tester.com](www.mail-tester.com) може показати вам, чи ваш домен заблоковано Spamhouse. Ви можете запросити видалення вашого домену/IP за посиланням: [https://www.spamhaus.org/lookup/](https://www.spamhaus.org/lookup/)

### Видалення з чорного списку Microsoft

Ви можете запросити видалення вашого домену/IP за посиланням [https://sender.office.com/](https://sender.office.com).

## Створення та запуск кампанії GoPhish

### Профіль відправника

* Встановіть **ім'я для ідентифікації** профілю відправника
* Вирішіть, з якого облікового запису ви будете відправляти листи з фішингом. Рекомендації: _noreply, support, servicedesk, salesforce..._
* Ви можете залишити порожніми ім'я користувача та пароль, але переконайтеся, що ви встановили прапорець "Ігнорувати помилки сертифіката"

![](<../../.gitbook/assets/image (253) (1) (2) (1) (1) (2) (2) (3) (3) (5) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (17).png>)

{% hint style="info" %}
Рекомендується використовувати функціонал "**Надіслати тестовий лист**", щоб перевірити, що все працює.\
Я рекомендую **надсилати тестові листи на адреси 10min mails**, щоб уникнути потрапляння в чорний список під час тестування.
{% endhint %}

### Шаблон електронної пошти

* Встановіть **ім'я для ідентифікації** шаблону
* Потім напишіть **тему** (нічого дивного, просто щось, що ви могли б очікувати прочитати в звичайному листі)
* Переконайтеся, що ви встановили прапорець "**Додати відстежувальне зображення**"
* Напишіть **шаблон електронного листа** (ви можете використовувати змінні, як у наступному прикладі):
```markup
<html>
<head>
<title></title>
</head>
<body>
<p class="MsoNormal"><span style="font-size:10.0pt;font-family:&quot;Verdana&quot;,sans-serif;color:black">Dear {{.FirstName}} {{.LastName}},</span></p>
<br />
Note: We require all user to login an a very suspicios page before the end of the week, thanks!<br />
<br />
Regards,</span></p>

WRITE HERE SOME SIGNATURE OF SOMEONE FROM THE COMPANY

<p>{{.Tracker}}</p>
</body>
</html>
```
Зауважте, що **для підвищення вірогідності електронної пошти** рекомендується використовувати якусь підпис з електронної пошти клієнта. Рекомендації:

* Надішліть електронного листа на **несуществуючу адресу** і перевірте, чи є у відповіді який-небудь підпис.
* Шукайте **публічні електронні адреси** типу info@ex.com або press@ex.com або public@ex.com та надсилайте їм електронний лист, очікуючи відповіді.
* Спробуйте зв'язатися з **якоюсь валідною виявленою** електронною адресою та зачекайте на відповідь

![](<../../.gitbook/assets/image (393).png>)

{% hint style="info" %}
Шаблон електронної пошти також дозволяє **долучати файли для відправки**. Якщо ви також хочете вкрасти виклики NTLM, використовуючи спеціально створені файли/документи, [прочитайте цю сторінку](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md).
{% endhint %}

### Посадова сторінка

* Напишіть **ім'я**
* **Напишіть HTML-код** веб-сторінки. Зверніть увагу, що ви можете **імпортувати** веб-сторінки.
* Позначте **Захоплення надісланих даних** та **Захоплення паролів**
* Встановіть **перенаправлення**

![](<../../.gitbook/assets/image (394).png>)

{% hint style="info" %}
Зазвичай вам потрібно буде змінити HTML-код сторінки та провести деякі тести локально (можливо, використовуючи деякий сервер Apache) **доки вам не сподобаються результати**. Потім напишіть цей HTML-код у відповідному полі.\
Зверніть увагу, що якщо вам потрібно **використовувати деякі статичні ресурси** для HTML (можливо, деякі CSS та JS сторінки), ви можете зберегти їх у _**/opt/gophish/static/endpoint**_ та потім отримати до них доступ з _**/static/\<filename>**_
{% endhint %}

{% hint style="info" %}
Щодо перенаправлення ви можете **перенаправити користувачів на легітимну головну веб-сторінку** жертви або перенаправити їх на _/static/migration.html_ наприклад, показати якусь **крутящуся колесо** ([**https://loading.io/**](https://loading.io)) протягом 5 секунд, а потім вказати, що процес був успішним.
{% endhint %}

### Користувачі та Групи

* Встановіть ім'я
* **Імпортуйте дані** (зверніть увагу, що для використання шаблону для прикладу вам потрібно прізвище, ім'я та електронну адресу кожного користувача)

![](<../../.gitbook/assets/image (395).png>)

### Кампанія

Нарешті, створіть кампанію, вибравши ім'я, шаблон електронної пошти, посадову сторінку, URL, профіль відправлення та групу. Зверніть увагу, що URL буде посиланням, відправленим жертвам

Зауважте, що **Профіль відправлення дозволяє відправити тестовий електронний лист, щоб побачити, як буде виглядати кінцевий лист з фішингом**:

![](<../../.gitbook/assets/image (396).png>)

{% hint style="info" %}
Я рекомендую **відправляти тестові листи на адреси 10min**, щоб уникнути блокування під час тестування.
{% endhint %}

Як тільки все готово, просто запустіть кампанію!

## Клонування веб-сайту

Якщо з якоїсь причини вам потрібно клонувати веб-сайт, перевірте наступну сторінку:

{% content-ref url="clone-a-website.md" %}
[clone-a-website.md](clone-a-website.md)
{% endcontent-ref %}

## Документи та файли з задніми дверима

У деяких оцінках фішингу (головним чином для Червоних Команд) ви також захочете **надіслати файли, що містять якусь форму задніх дверей** (можливо, C2 або просто щось, що спричинить аутентифікацію).\
Перегляньте наступну сторінку для деяких прикладів:

{% content-ref url="phishing-documents.md" %}
[phishing-documents.md](phishing-documents.md)
{% endcontent-ref %}

## Фішинг MFA

### Через Проксі MitM

Попередній атака досить хитра, оскільки ви підробляєте реальний веб-сайт та збираєте інформацію, введену користувачем. Незважаючи на це, якщо користувач не ввів правильний пароль або якщо застосунок, який ви підробили, налаштований з двофакторною аутентифікацією, **ця інформація не дозволить вам підробити користувача**.

Тут корисні інструменти, такі як [**evilginx2**](https://github.com/kgretzky/evilginx2)**,** [**CredSniper**](https://github.com/ustayready/CredSniper) та [**muraena**](https://github.com/muraenateam/muraena). Цей інструмент дозволить вам створити атаку типу MitM. Фактично, атаки працюють наступним чином:

1. Ви **підробляєте форму входу** реальної веб-сторінки.
2. Користувач **надсилає** свої **відомості для входу** на вашу фальшиву сторінку, а інструмент надсилає їх на реальну веб-сторінку, **перевіряючи, чи працюють відомості для входу**.
3. Якщо обліковий запис налаштований з **двофакторною аутентифікацією**, сторінка MitM попросить про неї, і як тільки **користувач введе** її, інструмент надішле її на реальну веб-сторінку.
4. Як тільки користувач автентифікується, ви (як атакуючий) **захопите відомості для входу, двофакторну аутентифікацію, куки та будь-яку інформацію** з кожної взаємодії, поки інструмент виконує атаку MitM.

### Через VNC

Що, якщо замість **направлення жертви на зловмисну сторінку** з таким самим виглядом, як оригінальна, ви направите його на **сесію VNC з браузером, підключеним до реальної веб-сторінки**? Ви зможете бачити, що він робить, вкрасти пароль, використану двофакторну аутентифікацію, куки...\
Ви можете це зробити за допомогою [**EvilnVNC**](https://github.com/JoelGMSec/EvilnoVNC)

## Виявлення виявлення

Очевидно, один з найкращих способів дізнатися, чи вас викрили, це **пошук вашого домену в чорних списках**. Якщо він з'являється в списку, то якимось чином ваш домен був виявлений як підозрілий.\
Один простий спосіб перевірити, чи ваш домен з'являється в будь-якому чорному списку, це використовувати [https://malwareworld.com/](https://malwareworld.com)

Однак є інші способи дізнатися, чи жертва **активно шукає підозріву фішингову діяльність в мережі**, як пояснено в:

{% content-ref url="detecting-phising.md" %}
[detecting-phising.md](detecting-phising.md)
{% endcontent-ref %}

Ви можете **купити домен з дуже схожою назвою** на домен жертви **і/або згенерувати сертифікат** для **піддомену** домена, яким ви керуєте **із ключовим словом** домена жертви. Якщо **жертва** взаємодіє з ними **через DNS або HTTP**, ви будете знати, що **він активно шукає** підозрілі домени, і вам потрібно буде діяти дуже обережно.

### Оцініть фішинг

Використовуйте [**Phishious** ](https://github.com/Rices/Phishious), щоб оцінити, чи вашу електронну пошту потрапить у спам або вона буде заблокована чи успішно доставлена.

## Посилання

* [https://zeltser.com/domain-name-variations-in-phishing/](https://zeltser.com/domain-name-variations-in-phishing/)
* [https://0xpatrik.com/phishing-domains/](https://0xpatrik.com/phishing-domains/)
* [https://darkbyte.net/robando-sesiones-y-bypasseando-2fa-con-evilnovnc/](https://darkbyte.net/robando-sesiones-y-bypasseando-2fa-con-evilnovnc/)
* [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)
