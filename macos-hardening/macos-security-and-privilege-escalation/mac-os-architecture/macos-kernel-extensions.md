# macOS Ядерні розширення

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди AWS HackTricks)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете, щоб ваша **компанія була рекламована на HackTricks**? Або ви хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перегляньте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу ексклюзивну колекцію [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний мерч PEASS та HackTricks**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) **групи Discord** або до [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Основна інформація

Ядерні розширення (Kexts) - це **пакети** з розширенням **`.kext`**, які **завантажуються безпосередньо в простір ядра macOS**, надаючи додатковий функціонал основній операційній системі.

### Вимоги

Очевидно, що це настільки потужно, що **складно завантажити ядерне розширення**. Ось **вимоги**, яким повинно відповідати ядерне розширення для завантаження:

* При **вході в режим відновлення**, ядерні **розширення повинні бути дозволені** для завантаження:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Ядерне розширення повинно бути **підписане сертифікатом підпису коду ядра**, який може видати лише **Apple**. Хто докладно перегляне компанію та причини, чому це потрібно.
* Ядерне розширення також повинно бути **підтверджене**, Apple зможе перевірити його на віруси.
* Тоді **користувач root** може **завантажити ядерне розширення**, а файли всередині пакету повинні **належати користувачеві root**.
* Під час процесу завантаження пакет повинен бути підготовлений в **захищеному некореневому місці**: `/Library/StagedExtensions` (потрібно дозвіл `com.apple.rootless.storage.KernelExtensionManagement`).
* Нарешті, при спробі завантажити його, користувач отримає [**запит на підтвердження**](https://developer.apple.com/library/archive/technotes/tn2459/\_index.html), і, якщо його прийнято, комп'ютер повинен бути **перезавантажений** для завантаження.

### Процес завантаження

У Catalina це було так: Цікаво відзначити, що процес **перевірки** відбувається в **userland**. Однак тільки додатки з дозволом **`com.apple.private.security.kext-management`** можуть **запитати ядро про завантаження розширення**: `kextcache`, `kextload`, `kextutil`, `kextd`, `syspolicyd`

1. **`kextutil`** cli **запускає** процес **перевірки** для завантаження розширення
* Він буде спілкуватися з **`kextd`**, використовуючи **Mach service**.
2. **`kextd`** перевірить кілька речей, таких як **підпис**
* Він буде спілкуватися з **`syspolicyd`**, щоб **перевірити**, чи можна **завантажити** розширення.
3. **`syspolicyd`** **запитає** **користувача**, якщо розширення раніше не було завантажено.
* **`syspolicyd`** повідомить результат **`kextd`**
4. **`kextd`** нарешті зможе **повідомити ядро про завантаження** розширення

Якщо **`kextd`** недоступний, **`kextutil`** може виконати ті самі перевірки.

## Посилання

* [https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/](https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/)
* [https://www.youtube.com/watch?v=hGKOskSiaQo](https://www.youtube.com/watch?v=hGKOskSiaQo)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди AWS HackTricks)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете, щоб ваша **компанія була рекламована на HackTricks**? Або ви хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перегляньте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу ексклюзивну колекцію [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний мерч PEASS та HackTricks**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) **групи Discord** або до [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
