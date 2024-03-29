# Аналіз дампу пам'яті

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити свою **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи у Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) є найбільш важливою подією з кібербезпеки в **Іспанії** та однією з найважливіших в **Європі**. З місією просування технічних знань, цей конгрес є кипучою точкою зустрічі для професіоналів технологій та кібербезпеки у будь-якій дисципліні.

{% embed url="https://www.rootedcon.com/" %}

## Початок

Почніть **шукати** **шкідливе ПЗ** всередині pcap. Використовуйте **інструменти**, згадані в [**Аналізі шкідливого ПЗ**](../malware-analysis.md).

## [Volatility](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)

**Volatility - це основний відкритий фреймворк для аналізу дампів пам'яті**. Цей інструмент на Python аналізує дампи зовнішніх джерел або віртуальних машин VMware, ідентифікуючи дані, такі як процеси та паролі на основі профілю ОС дампу. Він розширюється за допомогою плагінів, що робить його дуже гнучким для судових розслідувань.

**[Тут знайдете шпаргалку](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)**

## Звіт про аварійне завершення міні-дампу

Якщо дамп невеликий (лише кілька КБ, можливо, кілька МБ), то це, ймовірно, звіт про аварійне завершення міні-дампу, а не дамп пам'яті.

![](<../../../.gitbook/assets/image (216).png>)

Якщо у вас встановлено Visual Studio, ви можете відкрити цей файл та отримати деяку базову інформацію, таку як назва процесу, архітектура, інформація про виняток та виконувані модулі:

![](<../../../.gitbook/assets/image (217).png>)

Ви також можете завантажити виняток та переглянути декомпільовані інструкції

![](<../../../.gitbook/assets/image (219).png>)

![](<../../../.gitbook/assets/image (218) (1).png>)

В будь-якому випадку, Visual Studio не є найкращим інструментом для проведення аналізу глибини дампу.

Ви повинні **відкрити** його за допомогою **IDA** або **Radare**, щоб докладно **інспектувати** його.



​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) є найбільш важливою подією з кібербезпеки в **Іспанії** та однією з найважливіших в **Європі**. З місією просування технічних знань, цей конгрес є кипучою точкою зустрічі для професіоналів технологій та кібербезпеки у будь-якій дисципліні.

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити свою **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи у Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
