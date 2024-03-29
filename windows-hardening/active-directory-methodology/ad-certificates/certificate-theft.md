# Викрадення сертифікатів AD CS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

**Це невеликий огляд розділів про крадіжку з дослідження від [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)**


## Що я можу зробити з сертифікатом

Перед тим, як дізнатися, як вкрасти сертифікати, ось деяка інформація про те, на що може бути корисний сертифікат:
```powershell
# Powershell
$CertPath = "C:\path\to\cert.pfx"
$CertPass = "P@ssw0rd"
$Cert = New-Object
System.Security.Cryptography.X509Certificates.X509Certificate2 @($CertPath, $CertPass)
$Cert.EnhancedKeyUsageList

# cmd
certutil.exe -dump -v cert.pfx
```
## Експорт сертифікатів за допомогою криптографічних API – THEFT1

У **інтерактивній сеансі робочого столу** вилучення сертифікату користувача або машини разом з закритим ключем може бути легко виконано, особливо якщо **закритий ключ можна експортувати**. Це можна зробити, перейшовши до сертифікату в `certmgr.msc`, клацнувши правою кнопкою миші на ньому та вибравши `All Tasks → Export`, щоб створити файл .pfx, захищений паролем.

Для **програмного підходу** доступні інструменти, такі як командлет PowerShell `ExportPfxCertificate` або проекти, наприклад [проект CertStealer на C# від TheWover](https://github.com/TheWover/CertStealer). Вони використовують **Microsoft CryptoAPI** (CAPI) або Cryptography API: Next Generation (CNG) для взаємодії з сховищем сертифікатів. Ці API надають широкий спектр криптографічних послуг, включаючи ті, що необхідні для зберігання та аутентифікації сертифікатів.

Однак, якщо закритий ключ встановлено як неможливий до експорту, як CAPI, так і CNG зазвичай блокують вилучення таких сертифікатів. Для обходу цього обмеження можна використовувати інструменти, такі як **Mimikatz**. Mimikatz пропонує команди `crypto::capi` та `crypto::cng` для патчення відповідних API, що дозволяє експортувати закриті ключі. Зокрема, `crypto::capi` патчить CAPI в поточному процесі, тоді як `crypto::cng` спрямований на пам'ять **lsass.exe** для патчення.

## Крадіжка сертифікатів користувача через DPAPI – THEFT2

Додаткова інформація про DPAPI в:

{% content-ref url="../../windows-local-privilege-escalation/dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](../../windows-local-privilege-escalation/dpapi-extracting-passwords.md)
{% endcontent-ref %}

У Windows **закриті ключі сертифікатів захищені за допомогою DPAPI**. Важливо розуміти, що **місця зберігання закритих ключів для користувача та машини** є різними, а структури файлів варіюються в залежності від криптографічного API, яке використовується операційною системою. **SharpDPAPI** - це інструмент, який може автоматично навігувати цими різницями при розшифруванні блоків DPAPI.

**Сертифікати користувача** переважно знаходяться в реєстрі під `HKEY_CURRENT_USER\SOFTWARE\Microsoft\SystemCertificates`, але деякі з них також можна знайти в каталозі `%APPDATA%\Microsoft\SystemCertificates\My\Certificates`. Відповідні **закриті ключі** для цих сертифікатів зазвичай зберігаються в `%APPDATA%\Microsoft\Crypto\RSA\User SID\` для ключів **CAPI** та `%APPDATA%\Microsoft\Crypto\Keys\` для ключів **CNG**.

Для **вилучення сертифіката та відповідного закритого ключа** процес включає в себе:

1. **Вибір цільового сертифіката** зі сховища користувача та отримання назви його ключового сховища.
2. **Пошук необхідного майстер-ключа DPAPI** для розшифрування відповідного закритого ключа.
3. **Розшифрування закритого ключа** за допомогою майстер-ключа DPAPI у відкритому вигляді.

Для **отримання майстер-ключа DPAPI у відкритому вигляді** можна використовувати наступні підходи:
```bash
# With mimikatz, when running in the user's context
dpapi::masterkey /in:"C:\PATH\TO\KEY" /rpc

# With mimikatz, if the user's password is known
dpapi::masterkey /in:"C:\PATH\TO\KEY" /sid:accountSid /password:PASS
```
Для оптимізації розшифрування файлів masterkey та файлів приватних ключів команда `certificates` з [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI) є корисною. Вона приймає `/pvk`, `/mkfile`, `/password` або `{GUID}:KEY` як аргументи для розшифрування приватних ключів та пов'язаних сертифікатів, що в результаті генерує файл `.pem`.
```bash
# Decrypting using SharpDPAPI
SharpDPAPI.exe certificates /mkfile:C:\temp\mkeys.txt

# Converting .pem to .pfx
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
## Викрадення машинного сертифікату через DPAPI – THEFT3

Машинні сертифікати, збережені Windows у реєстрі за адресою `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SystemCertificates`, та пов'язані з ними приватні ключі, розташовані в `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\RSA\MachineKeys` (для CAPI) та `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\Keys` (для CNG), шифруються за допомогою майстер-ключів DPAPI машини. Ці ключі не можуть бути розшифровані за допомогою резервного ключа DPAPI домену; замість цього потрібний **секрет LSA DPAPI_SYSTEM**, до якого має доступ лише користувач SYSTEM.

Ручне розшифрування можливе за допомогою виконання команди `lsadump::secrets` в **Mimikatz** для видобуття секрету LSA DPAPI_SYSTEM, а потім використання цього ключа для розшифрування майстер-ключів машини. Альтернативно, можна скористатися командою `crypto::certificates /export /systemstore:LOCAL_MACHINE` в Mimikatz після патчення CAPI/CNG, як описано раніше.

**SharpDPAPI** пропонує більш автоматизований підхід за допомогою команди certificates. Коли використовується прапорець `/machine` з підвищеними дозволами, він ескалює до SYSTEM, виводить секрет LSA DPAPI_SYSTEM, використовує його для розшифрування майстер-ключів DPAPI машини, а потім використовує ці текстові ключі як таблицю пошуку для розшифрування будь-яких приватних ключів машинного сертифікату.


## Пошук файлів сертифікатів – THEFT4

Сертифікати іноді знаходяться безпосередньо в файловій системі, наприклад, на мережевих дисках або в папці Завантаження. Найчастіше зустрічаються типи файлів сертифікатів, спрямованих на середовища Windows, - це файли з розширеннями `.pfx` та `.p12`. Хоча рідше, зустрічаються файли з розширеннями `.pkcs12` та `.pem`. Додаткові важливі розширення файлів, пов'язаних з сертифікатами, включають:
- `.key` для приватних ключів,
- `.crt`/`.cer` для лише сертифікатів,
- `.csr` для запитів на підпис сертифікату, які не містять сертифікатів або приватних ключів,
- `.jks`/`.keystore`/`.keys` для Java Keystores, які можуть містити сертифікати разом з приватними ключами, які використовуються Java-додатками.

Ці файли можна знайти за допомогою PowerShell або командного рядка, шукаючи зазначені розширення.

У випадках, коли знайдено файл сертифіката PKCS#12, і він захищений паролем, можливе видобування хешу за допомогою `pfx2john.py`, доступного на [fossies.org](https://fossies.org/dox/john-1.9.0-jumbo-1/pfx2john_8py_source.html). Після цього можна використати JohnTheRipper для спроби розкрити пароль.
```powershell
# Example command to search for certificate files in PowerShell
Get-ChildItem -Recurse -Path C:\Users\ -Include *.pfx, *.p12, *.pkcs12, *.pem, *.key, *.crt, *.cer, *.csr, *.jks, *.keystore, *.keys

# Example command to use pfx2john.py for extracting a hash from a PKCS#12 file
pfx2john.py certificate.pfx > hash.txt

# Command to crack the hash with JohnTheRipper
john --wordlist=passwords.txt hash.txt
```
## Викрадення облікових даних NTLM через PKINIT - THEFT5

Наданий вміст пояснює метод викрадення облікових даних NTLM через PKINIT, зокрема через метод викрадення, позначений як THEFT5. Ось переформульоване у пасивному стані пояснення, з анонімізованим вмістом та підсумованим, де це можливо:

Для підтримки аутентифікації NTLM [MS-NLMP] для додатків, які не підтримують аутентифікацію Kerberos, КДК розроблений для повернення односторонньої функції NTLM користувача (OWF) у привілейованому атрибутному сертифікаті (PAC), зокрема в буфері `PAC_CREDENTIAL_INFO`, коли використовується PKCA. Отже, якщо обліковий запис аутентифікується та забезпечує Квиток для отримання квитка (TGT) через PKINIT, вбудований механізм надає можливість поточному хосту витягти хеш NTLM з TGT для підтримки застарілих протоколів аутентифікації. Цей процес передбачає розшифрування структури `PAC_CREDENTIAL_DATA`, яка в сутності є NDR серіалізованим зображенням тексту NTLM.

Утиліта **Kekeo**, доступна за посиланням [https://github.com/gentilkiwi/kekeo](https://github.com/gentilkiwi/kekeo), згадується як здатна запитувати TGT, що містить ці конкретні дані, тим самим сприяючи отриманню NTLM користувача. Використана для цієї мети команда виглядає наступним чином:
```bash
tgt::pac /caname:generic-DC-CA /subject:genericUser /castore:current_user /domain:domain.local
```
Додатково відзначено, що Kekeo може обробляти захищені сертифікати зі смарт-карткою, якщо можна отримати PIN-код, з посиланням на [https://github.com/CCob/PinSwipe](https://github.com/CCob/PinSwipe). Таку ж можливість, як повідомляється, підтримує **Rubeus**, доступний за посиланням [https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus).

Ця пояснювальна записка охоплює процес та інструменти, що використовуються для крадіжки облікових даних NTLM через PKINIT, зосереджуючись на отриманні хешів NTLM через TGT, отриманий за допомогою PKINIT, та утиліти, які полегшують цей процес.
