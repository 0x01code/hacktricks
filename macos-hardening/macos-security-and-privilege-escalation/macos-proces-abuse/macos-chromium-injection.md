# Впровадження Chromium на macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>

## Основна інформація

Браузери на основі Chromium, такі як Google Chrome, Microsoft Edge, Brave та інші. Ці браузери побудовані на відкритому проекті Chromium, що означає, що вони мають спільну базу та, отже, схожі функціональні можливості та опції розробника.

#### Прапорець `--load-extension`

Прапорець `--load-extension` використовується при запуску браузера на основі Chromium з командного рядка або скрипту. Цей прапорець дозволяє **автоматично завантажувати один або кілька розширень** в браузер при запуску.

#### Прапорець `--use-fake-ui-for-media-stream`

Прапорець `--use-fake-ui-for-media-stream` є ще однією опцією командного рядка, яку можна використовувати для запуску браузерів на основі Chromium. Цей прапорець призначений для **обхіду звичайних запитів користувача, які запитують дозвіл на доступ до медіапотоків з камери та мікрофону**. Коли використовується цей прапорець, браузер автоматично надає дозвіл на будь-який веб-сайт або додаток, який запитує доступ до камери або мікрофону.

### Інструменти

* [https://github.com/breakpointHQ/snoop](https://github.com/breakpointHQ/snoop)
* [https://github.com/breakpointHQ/VOODOO](https://github.com/breakpointHQ/VOODOO)

### Приклад
```bash
# Intercept traffic
voodoo intercept -b chrome
```
Знайдіть більше прикладів у посиланнях на інструменти

## Посилання

* [https://twitter.com/RonMasas/status/1758106347222995007](https://twitter.com/RonMasas/status/1758106347222995007)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
