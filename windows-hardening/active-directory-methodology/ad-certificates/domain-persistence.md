# Постійність домену AD CS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв GitHub**.

</details>

**Це підсумок технік постійності домену, які були опубліковані в [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)**. Перевірте це для отримання додаткових деталей.

## Підробка сертифікатів за допомогою викрадених сертифікатів ЦС - DPERSIST1

Як ви можете визначити, що сертифікат є сертифікатом ЦС?

Можна визначити, що сертифікат є сертифікатом ЦС, якщо виконуються кілька умов:

- Сертифікат зберігається на сервері ЦС, а його приватний ключ захищений DPAPI машини або апаратними засобами, такими як TPM/HSM, якщо операційна система підтримує це.
- Як видаючий орган, так і суб'єкт сертифіката відповідають відмінному імені ЦС.
- У сертифіката виключно присутнє розширення "Версія ЦС".
- У сертифіката відсутні поля розширеного використання ключа (EKU).

Для вилучення приватного ключа цього сертифіката підтримується інструмент `certsrv.msc` на сервері ЦС через вбудований GUI. Однак цей сертифікат не відрізняється від інших, збережених у системі; тому можна застосовувати методи, такі як [техніка THEFT2](certificate-theft.md#user-certificate-theft-via-dpapi-theft2) для вилучення.

Сертифікат та приватний ключ також можна отримати за допомогою Certipy за допомогою наступної команди:
```bash
certipy ca 'corp.local/administrator@ca.corp.local' -hashes :123123.. -backup
```
Після отримання сертифіката ЦС та його приватного ключа у форматі `.pfx`, можна використовувати інструменти, такі як [ForgeCert](https://github.com/GhostPack/ForgeCert), для генерації дійсних сертифікатів:
```bash
# Generating a new certificate with ForgeCert
ForgeCert.exe --CaCertPath ca.pfx --CaCertPassword Password123! --Subject "CN=User" --SubjectAltName localadmin@theshire.local --NewCertPath localadmin.pfx --NewCertPassword Password123!

# Generating a new certificate with certipy
certipy forge -ca-pfx CORP-DC-CA.pfx -upn administrator@corp.local -subject 'CN=Administrator,CN=Users,DC=CORP,DC=LOCAL'

# Authenticating using the new certificate with Rubeus
Rubeus.exe asktgt /user:localdomain /certificate:C:\ForgeCert\localadmin.pfx /password:Password123!

# Authenticating using the new certificate with certipy
certipy auth -pfx administrator_forged.pfx -dc-ip 172.16.126.128
```
{% hint style="warning" %}
Користувач, який є метою підробки сертифікатів, повинен бути активним і мати можливість аутентифікації в Active Directory для успішного процесу. Підробка сертифікатів для спеціальних облікових записів, таких як krbtgt, є неефективною.
{% endhint %}

Цей підроблений сертифікат буде **дійсним** до вказаної дати закінчення і **доки кореневий сертифікат ЦС** є дійсним (зазвичай від 5 до **10+ років**). Він також є дійсним для **машин**, тому разом з **S4U2Self** зловмисник може **зберігати постійність на будь-якій машині домену** доти, поки сертифікат ЦС є дійсним.\
Більше того, **сертифікати, згенеровані** цим методом, **не можуть бути відкликані**, оскільки ЦС не відомо про них.

## Довіра до сертифікатів Rogue CA - DPERSIST2

Об'єкт `NTAuthCertificates` визначено для містить один або кілька **сертифікатів ЦС** у своєму атрибуті `cacertificate`, які використовує Active Directory (AD). Процес перевірки **контролера домену** включає перевірку об'єкта `NTAuthCertificates` на наявність запису, який відповідає **ЦС, вказаному** в полі Видавця аутентифікуючого **сертифіката**. Аутентифікація відбувається, якщо знайдено відповідність.

Самопідписаний сертифікат ЦС може бути доданий до об'єкта `NTAuthCertificates` зловмисником, за умови, що вони мають контроль над цим об'єктом AD. Зазвичай дозвіл на зміну цього об'єкта мають лише члени групи **Enterprise Admin**, разом з **Domain Admins** або **Адміністраторами** в **домені кореня лісу**. Вони можуть редагувати об'єкт `NTAuthCertificates`, використовуючи `certutil.exe` з командою `certutil.exe -dspublish -f C:\Temp\CERT.crt NTAuthCA126`, або використовуючи [**інструмент здоров'я PKI**](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/import-third-party-ca-to-enterprise-ntauth-store#method-1---import-a-certificate-by-using-the-pki-health-tool).

Ця можливість особливо актуальна, коли використовується разом з раніше описаним методом, що включає ForgeCert для динамічного генерування сертифікатів.

## Зловживання зловмисною конфігурацією - DPERSIST3

Можливості **постійності** через **модифікації дескрипторів безпеки компонентів AD CS** є багато. Модифікації, описані в розділі "[Підвищення домену](domain-escalation.md)", можуть бути зловмисно реалізовані зловмисником з підвищеним доступом. Це включає додавання "прав контролю" (наприклад, WriteOwner/WriteDACL/тощо) до чутливих компонентів, таких як:

- Об'єкт **комп'ютера AD сервера ЦС**
- Сервер **RPC/DCOM сервера ЦС**
- Будь-який **нащадковий об'єкт або контейнер AD** в **`CN=Public Key Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<COM>`** (наприклад, контейнер Шаблонів сертифікатів, контейнер Видачі сертифікатів, об'єкт NTAuthCertificates тощо)
- **Групи AD делеговані права на управління AD CS** за замовчуванням або організацією (такі як вбудована група Cert Publishers та будь-який з її членів)

Прикладом зловживання може бути додавання зловмисником, який має **підвищені дозволи** в домені, права **`WriteOwner`** до типового **шаблону сертифіката `User`**, зловмисник буде принципалом для цього права. Для експлуатації цього зловмисник спочатку змінив би власника **шаблону `User`** на себе. Після цього **`mspki-certificate-name-flag`** буде встановлено на **1** на шаблоні для активації **`ENROLLEE_SUPPLIES_SUBJECT`**, що дозволяє користувачу вказати альтернативне ім'я в запиті. Після цього зловмисник може **зареєструватися** за допомогою **шаблону**, вибравши ім'я **адміністратора домену** як альтернативне ім'я, і використовувати отриманий сертифікат для аутентифікації як DA.
