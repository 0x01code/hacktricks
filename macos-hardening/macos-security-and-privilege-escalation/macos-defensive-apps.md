# Захисні програми для macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди HackTricks AWS)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

## Брандмауери

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): Він буде моніторити кожне з'єднання, створене кожним процесом. Залежно від режиму (тихе дозволення з'єднань, тихе відмова від з'єднання та сповіщення) він **покаже вам сповіщення** кожного разу, коли встановлюється нове з'єднання. Також має дуже зручний графічний інтерфейс для перегляду всієї цієї інформації.
* [**LuLu**](https://objective-see.org/products/lulu.html): Брандмауер Objective-See. Це базовий брандмауер, який буде сповіщати вас про підозрілі з'єднання (він має графічний інтерфейс, але він не такий красивий, як у Little Snitch).

## Виявлення постійності

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Додаток Objective-See, який буде шукати в кількох місцях, де **зловмисне програмне забезпечення може зберігатися** (це інструмент одноразового використання, а не служба моніторингу).
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): Подібно до KnockKnock, він моніторить процеси, які генерують постійність.

## Виявлення кейлогерів

* [**ReiKey**](https://objective-see.org/products/reikey.html): Додаток Objective-See для пошуку **кейлогерів**, які встановлюють клавіатурні "події натискання клавіш"&#x20;

## Виявлення вимагання викупу

* [**RansomWhere**](https://objective-see.org/products/ransomwhere.html): Додаток Objective-See для виявлення дій з **шифрування файлів**.

## Виявлення мікрофону та вебкамери

* [**OverSight**](https://objective-see.org/products/oversight.html): Додаток Objective-See для виявлення **додатків, які починають використовувати вебкамеру та мікрофон.**

## Виявлення внесення процесу

* [**Shield**](https://theevilbit.github.io/shield/): Додаток, який **виявляє різні техніки внесення процесу**.
