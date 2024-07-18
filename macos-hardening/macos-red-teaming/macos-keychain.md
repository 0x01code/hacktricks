# Ключовий ланцюг macOS

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа взлому AWS для Червоної Команди (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа взлому GCP для Червоної Команди (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковик, який працює на **темному вебі** та надає **безкоштовні** функції для перевірки, чи були **компанія або її клієнти скомпрометовані** **викрадачами шкідливих програм**.

Основною метою WhiteIntel є боротьба з захопленням облікових записів та атаками вимагання викупу, що виникають внаслідок шкідливих програм, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їхній двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

***

## Основні ключові ланцюги

* **Ключовий ланцюг користувача** (`~/Library/Keychains/login.keycahin-db`), який використовується для зберігання **користувацьких облікових даних**, таких як паролі додатків, паролі для Інтернету, сертифікати, паролі мережі та користувацькі публічні/приватні ключі.
* **Системний ключовий ланцюг** (`/Library/Keychains/System.keychain`), який зберігає **системні облікові дані**, такі як паролі WiFi, кореневі сертифікати системи, приватні ключі системи та паролі додатків системи.

### Доступ до ключового ланцюга паролів

Ці файли, хоча вони не мають вбудованого захисту і можуть бути **завантажені**, зашифровані та вимагають **пароль користувача для розшифрування**. Інструмент, такий як [**Chainbreaker**](https://github.com/n0fate/chainbreaker), може бути використаний для розшифрування.

## Захист записів ключового ланцюга

### Списки керування доступом (ACL)

Кожен запис у ключовому ланцюзі керується **списками керування доступом (ACL)**, які вказують, хто може виконувати різні дії з записом ключового ланцюга, включаючи:

* **ACLAuhtorizationExportClear**: Дозволяє власнику отримати чіткий текст секрету.
* **ACLAuhtorizationExportWrapped**: Дозволяє власнику отримати зашифрований чіткий текст із наданим іншим паролем.
* **ACLAuhtorizationAny**: Дозволяє власнику виконувати будь-яку дію.

ACL супроводжуються **списком довірених додатків**, які можуть виконувати ці дії без підтвердження. Це може бути:

* **N`il`** (не потрібно авторизації, **всім довіряють**)
* Порожній список (**ніхто не довіряє**)
* **Список** конкретних **додатків**.

Також запис може містити ключ **`ACLAuthorizationPartitionID`,** який використовується для ідентифікації **teamid, apple,** та **cdhash.**

* Якщо вказано **teamid**, то для **доступу до значення запису без** підказки використовуваний додаток повинен мати **той самий teamid**.
* Якщо вказано **apple**, то додаток повинен бути **підписаний Apple**.
* Якщо вказано **cdhash**, то додаток повинен мати конкретний **cdhash**.

### Створення запису ключового ланцюга

При створенні **нового запису** за допомогою **`Keychain Access.app`**, застосовуються наступні правила:

* Усі додатки можуть шифрувати.
* **Жоден додаток** не може експортувати/розшифровувати (без підказки користувача).
* Усі додатки можуть бачити перевірку цілісності.
* Жоден додаток не може змінювати ACL.
* **PartitionID** встановлено на **`apple`**.

Коли **додаток створює запис у ключовому ланцюзі**, правила трохи відрізняються:

* Усі додатки можуть шифрувати.
* Тільки **створюючий додаток** (або будь-які інші додатки, які явно додані) можуть експортувати/розшифровувати (без підказки користувача).
* Усі додатки можуть бачити перевірку цілісності.
* Жоден додаток не може змінювати ACL.
* **PartitionID** встановлено на **`teamid:[teamID тут]`**.

## Доступ до ключового ланцюга

### `security`
```bash
# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S
```
### APIs

{% hint style="success" %}
**Перелік ключових перелічень та витягування** секретів, які **не викличуть запиту**, можна виконати за допомогою інструменту [**LockSmith**](https://github.com/its-a-feature/LockSmith)
{% endhint %}

Перелік та отримання **інформації** про кожен запис у ключовому ланцюжку:

* API **`SecItemCopyMatching`** надає інформацію про кожен запис та є деякі атрибути, які можна встановити при його використанні:
* **`kSecReturnData`**: Якщо true, він спробує розшифрувати дані (встановіть false, щоб уникнути можливих спливаючих вікон)
* **`kSecReturnRef`**: Отримати також посилання на елемент ключового ланцюжка (встановіть true у випадку, якщо пізніше ви побачите, що ви можете розшифрувати без спливаючого вікна)
* **`kSecReturnAttributes`**: Отримати метадані про записи
* **`kSecMatchLimit`**: Скільки результатів повернути
* **`kSecClass`**: Який тип запису у ключовому ланцюжку

Отримайте **ACLs** кожного запису:

* За допомогою API **`SecAccessCopyACLList`** ви можете отримати **ACL для елемента ключового ланцюжка**, і він поверне список ACL (наприклад, `ACLAuhtorizationExportClear` та інші раніше згадані), де кожен список має:
* Опис
* **Список довірених додатків**. Це може бути:
* Додаток: /Applications/Slack.app
* Бінарний файл: /usr/libexec/airportd
* Група: group://AirPort

Експорт даних:

* API **`SecKeychainItemCopyContent`** отримує текст
* API **`SecItemExport`** експортує ключі та сертифікати, але можливо доведеться встановити паролі для експорту вмісту зашифрованим

І ось **вимоги**, щоб мати можливість **експортувати секрет без запиту**:

* Якщо **1+ довірених** додатків перелічено:
* Потрібні відповідні **авторизації** (**`Nil`**, або бути **частиною** списку дозволених додатків у авторизації для доступу до конфіденційної інформації)
* Потрібно, щоб підпис коду відповідав **PartitionID**
* Потрібно, щоб підпис коду відповідав підпису одного **довіреного додатка** (або бути членом правильної групи KeychainAccessGroup)
* Якщо **всі додатки довірені**:
* Потрібні відповідні **авторизації**
* Потрібно, щоб підпис коду відповідав **PartitionID**
* Якщо **немає PartitionID**, тоді це не потрібно

{% hint style="danger" %}
Отже, якщо перелічено **1 додаток**, вам потрібно **впровадити код у цей додаток**.

Якщо **apple** вказано в **partitionID**, ви можете отримати до нього доступ за допомогою **`osascript`**, тому все, що довіряє всім додаткам з apple в partitionID. **`Python`** також може бути використаний для цього.
{% endhint %}

### Два додаткові атрибути

* **Невидимий**: Це булевий прапорець для **приховання** запису від програми **UI** Keychain
* **Загальний**: Це для зберігання **метаданих** (тому це НЕ ШИФРОВАНО)
* Компанія Microsoft зберігала відкритий текст всіх оновлювальних токенів для доступу до чутливих кінцевих точок.

## Посилання

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковий двигун, живлений **темним вебом**, який пропонує **безкоштовні** функціональні можливості для перевірки, чи були компанія або її клієнти **пошкоджені** **викрадачами шкідливих програм**.

Основною метою WhiteIntel є боротьба з захопленням облікових записів та атаками вимагання викупу, що виникають внаслідок шкідливих програм, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їх двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
