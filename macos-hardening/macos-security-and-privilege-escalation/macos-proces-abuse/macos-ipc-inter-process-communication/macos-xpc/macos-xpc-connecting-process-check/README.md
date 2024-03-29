# Перевірка процесу підключення XPC macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Перевірка процесу підключення XPC

Коли встановлюється з'єднання з XPC-сервісом, сервер перевіряє, чи дозволено це з'єднання. Ось перевірки, які зазвичай виконуються:

1. Перевірка, чи підключений **процес підписаний Apple-підписом** (наданим лише Apple).
* Якщо це **не підтверджено**, зловмисник може створити **фальшивий сертифікат**, щоб відповідати будь-якій іншій перевірці.
2. Перевірка, чи підключений процес підписаний **сертифікатом організації** (перевірка ідентифікатора команди).
* Якщо це **не підтверджено**, можна використовувати **будь-який сертифікат розробника** від Apple для підпису та підключення до сервісу.
3. Перевірка, чи підключений процес містить **правильний ідентифікатор пакета**.
* Якщо це **не підтверджено**, будь-який інструмент, **підписаний тією ж організацією**, може бути використаний для взаємодії з XPC-сервісом.
4. (4 або 5) Перевірка, чи у підключеному процесі є **правильний номер версії програмного забезпечення**.
* Якщо це **не підтверджено**, можна використовувати старі, небезпечні клієнти, вразливі до впровадження процесу, для підключення до XPC-сервісу навіть за умови інших перевірок.
5. (4 або 5) Перевірка, чи у підключеному процесі є зміцнений час виконання без небезпечних дозволів (таких, як ті, що дозволяють завантажувати довільні бібліотеки або використовувати змінні середовища DYLD)
1. Якщо це **не підтверджено**, клієнт може бути **вразливим до впровадження коду**
6. Перевірка, чи у підключеному процесі є **дозвіл**, що дозволяє йому підключатися до сервісу. Це застосовується до бінарних файлів Apple.
7. **Перевірка** повинна бути **заснована** на **аудиторському токені підключення клієнта** **замість** його **ідентифікатора процесу (PID)**, оскільки перше запобігає **атакам повторного використання PID**.
* Розробники **рідко використовують виклик API аудиторського токена**, оскільки він є **приватним**, тому Apple може **змінити** його в будь-який момент. Крім того, використання приватного API не дозволяється в додатках Mac App Store.
* Якщо використовується метод **`processIdentifier`**, це може бути вразливим
* Слід використовувати **`xpc_dictionary_get_audit_token`** замість **`xpc_connection_get_audit_token`**, оскільки останнє також може бути [вразливим у певних ситуаціях](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/).

### Атаки на комунікацію

Для отримання додаткової інформації про перевірку атаки повторного використання PID див.:

{% content-ref url="macos-pid-reuse.md" %}
[macos-pid-reuse.md](macos-pid-reuse.md)
{% endcontent-ref %}

Для отримання додаткової інформації про атаку **`xpc_connection_get_audit_token`** див.:

{% content-ref url="macos-xpc_connection_get_audit_token-attack.md" %}
[macos-xpc\_connection\_get\_audit\_token-attack.md](macos-xpc\_connection\_get\_audit\_token-attack.md)
{% endcontent-ref %}

### Trustcache - Запобігання атакам на зниження рівня

Trustcache - це захисний метод, введений на машинах Apple Silicon, який зберігає базу даних CDHSAH бінарних файлів Apple, щоб можна було виконувати лише дозволені незмінені бінарні файли. Це запобігає виконанню версій зниження.

### Приклади коду

Сервер реалізує цю **перевірку** у функції з назвою **`shouldAcceptNewConnection`**.

{% code overflow="wrap" %}
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
//Check connection
return YES;
}
```
Об'єкт NSXPCConnection має **приватну** властивість **`auditToken`** (та, яка повинна бути використана, але може змінитися) та **публічну** властивість **`processIdentifier`** (та, яка не повинна використовуватися).

Підключений процес можна перевірити за допомогою чогось на зразок:
```objectivec
[...]
SecRequirementRef requirementRef = NULL;
NSString requirementString = @"anchor apple generic and identifier \"xyz.hacktricks.service\" and certificate leaf [subject.CN] = \"TEAMID\" and info [CFBundleShortVersionString] >= \"1.0\"";
/* Check:
- Signed by a cert signed by Apple
- Check the bundle ID
- Check the TEAMID of the signing cert
- Check the version used
*/

// Check the requirements with the PID (vulnerable)
SecRequirementCreateWithString(requirementString, kSecCSDefaultFlags, &requirementRef);
SecCodeCheckValidity(code, kSecCSDefaultFlags, requirementRef);

// Check the requirements wuing the auditToken (secure)
SecTaskRef taskRef = SecTaskCreateWithAuditToken(NULL, ((ExtendedNSXPCConnection*)newConnection).auditToken);
SecTaskValidateForRequirement(taskRef, (__bridge CFStringRef)(requirementString))
```
{% endcode %}

Якщо розробник не хоче перевіряти версію клієнта, він може перевірити, що клієнт принаймні не вразливий на впровадження процесу:
```objectivec
[...]
CFDictionaryRef csInfo = NULL;
SecCodeCopySigningInformation(code, kSecCSDynamicInformation, &csInfo);
uint32_t csFlags = [((__bridge NSDictionary *)csInfo)[(__bridge NSString *)kSecCodeInfoStatus] intValue];
const uint32_t cs_hard = 0x100;        // don't load invalid page.
const uint32_t cs_kill = 0x200;        // Kill process if page is invalid
const uint32_t cs_restrict = 0x800;    // Prevent debugging
const uint32_t cs_require_lv = 0x2000; // Library Validation
const uint32_t cs_runtime = 0x10000;   // hardened runtime
if ((csFlags & (cs_hard | cs_require_lv)) {
return Yes; // Accept connection
}
```
{% endcode %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>
