# FZ - NFC

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Працюєте в **кібербезпеці?** Хочете, щоб ваша **компанія рекламувалася на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Вступ <a href="#id-9wrzi" id="id-9wrzi"></a>

Для інформації про RFID та NFC перевірте наступну сторінку:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## Підтримувані NFC-карти <a href="#id-9wrzi" id="id-9wrzi"></a>

{% hint style="danger" %}
Крім NFC-карт Flipper Zero підтримує **інший тип високочастотних карт**, таких як кілька **Mifare** Classic та Ultralight та **NTAG**.
{% endhint %}

Нові типи NFC-карт будуть додані до списку підтримуваних карт. Flipper Zero підтримує наступні **типи NFC-карт A** (ISO 14443A):

* ﻿**Банківські карти (EMV)** — лише читає UID, SAK та ATQA без збереження.
* ﻿**Невідомі карти** — читає (UID, SAK, ATQA) та емулює UID.

Для **типів NFC-карт B, F та V**, Flipper Zero може читати UID без збереження.

### Типи NFC-карт A <a href="#uvusf" id="uvusf"></a>

#### Банківська карта (EMV) <a href="#kzmrp" id="kzmrp"></a>

Flipper Zero може лише читати UID, SAK, ATQA та збережені дані на банківських картах **без збереження**.

Екран читання банківської картиДля банківських карт Flipper Zero може лише читати дані **без збереження та емуляції**.

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### Невідомі карти <a href="#id-37eo8" id="id-37eo8"></a>

Коли Flipper Zero **не може визначити тип NFC-карти**, тоді можна **читати тільки UID, SAK та ATQA** та **зберігати** їх.

Екран читання невідомої картиДля невідомих NFC-карт Flipper Zero може емулювати лише UID.

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### Типи NFC-карт B, F та V <a href="#wyg51" id="wyg51"></a>

Для **типів NFC-карт B, F та V**, Flipper Zero може лише **читати та відображати UID** без збереження.

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w=2704" alt=""><figcaption></figcaption></figure>

## Дії

Для вступу щодо NFC [**прочитайте цю сторінку**](../pentesting-rfid.md#high-frequency-rfid-tags-13.56-mhz).

### Читати

Flipper Zero може **читати NFC-карти**, проте він **не розуміє всі протоколи**, які базуються на ISO 14443. Однак, оскільки **UID є атрибутом низького рівня**, ви можете опинитися в ситуації, коли **UID вже прочитано, але протокол передачі даних високого рівня все ще невідомий**. Ви можете читати, емулювати та вручну вводити UID за допомогою Flipper для примітивних читачів, які використовують UID для авторизації.

#### Читання UID ПРОТИ Читання Даних всередині <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

У Flipper читання тегів 13,56 МГц може бути розділено на дві частини:

* **Читання низького рівня** — читає лише UID, SAK та ATQA. Flipper намагається вгадати протокол високого рівня на основі цих даних, прочитаних з карти. Ви не можете бути на 100% впевненими в цьому, оскільки це лише припущення на основі певних факторів.
* **Читання високого рівня** — читає дані з пам'яті карти за допомогою конкретного протоколу високого рівня. Це буде читання даних на Mifare Ultralight, читання секторів з Mifare Classic або читання атрибутів картки з PayPass/Apple Pay.

### Читати Конкретно

У випадку, якщо Flipper Zero не може визначити тип карти з даних низького рівня, в `Додаткові дії` ви можете вибрати `Читати конкретний тип карти` та **вручну вказати тип карти, яку ви хочете прочитати**.

#### Банківські карти EMV (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

Крім простого читання UID, ви можете витягти багато інших даних з банківської карти. Можливо **отримати повний номер карти** (16 цифр на передній стороні карти), **дату дії**, а в деяких випадках навіть **ім'я власника** разом із списком **останніх транзакцій**.\
Однак **CVV таким чином не можна прочитати** (3 цифри на задній стороні карти). Також **банківські карти захищені від атак повторення**, тому копіювання її за допомогою Flipper і спроба емуляції для оплати чогось не працюватиме.

## Посилання

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете, щоб ваша **компанія рекламувалася на HackTricks**? або ви хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
