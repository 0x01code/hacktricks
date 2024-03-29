<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

## Цілісність прошивки

**Спеціальні прошивки та/або скомпільовані бінарні файли можуть бути завантажені для використання уразливостей цілісності або перевірки підпису**. Можна дотримуватися наступних кроків для компіляції backdoor bind shell:

1. Прошивку можна видобути за допомогою firmware-mod-kit (FMK).
2. Повинна бути визначена архітектура та порядок байтів цільової прошивки.
3. Можна побудувати крос-компілятор, використовуючи Buildroot або інші відповідні методи для середовища.
4. Backdoor можна побудувати за допомогою крос-компілятора.
5. Backdoor можна скопіювати до каталогу /usr/bin видобутої прошивки.
6. Відповідний бінарний файл QEMU можна скопіювати до кореневої файлової системи видобутої прошивки.
7. Backdoor можна емулювати за допомогою chroot та QEMU.
8. До backdoor можна отримати доступ через netcat.
9. Бінарний файл QEMU слід видалити з кореневої файлової системи видобутої прошивки.
10. Змінену прошивку можна упакувати за допомогою FMK.
11. Backdoored прошивку можна протестувати, емулюючи її за допомогою набору інструментів для аналізу прошивки (FAT) та підключаючись до IP-адреси та порту цільового backdoor за допомогою netcat.

Якщо кореневий shell вже було отримано за допомогою динамічного аналізу, маніпулювання завантажувачем або тестуванням безпеки обладнання, можна виконати попередньо скомпільовані шкідливі бінарні файли, такі як імпланти або зворотні оболонки. Автоматизовані інструменти для завантаження/імплантування, такі як фреймворк Metasploit та 'msfvenom', можна використовувати за допомогою наступних кроків:

1. Повинна бути визначена архітектура та порядок байтів цільової прошивки.
2. Msfvenom може бути використаний для вказання цільового навантаження, IP-адреси атакуючого хоста, номера порту для прослуховування, типу файлу, архітектури, платформи та вихідного файлу.
3. Навантаження може бути передане на компрометований пристрій та переконанося, що воно має права на виконання.
4. Metasploit може бути підготовлений для обробки вхідних запитів, запустивши msfconsole та налаштувавши параметри відповідно до навантаження.
5. Зворотня оболонка meterpreter може бути виконана на компрометованому пристрої.
6. Сеанси meterpreter можуть бути моніторені під час їх відкриття.
7. Можна виконувати дії після експлуатації.

У разі можливості можна використовувати уразливості в скриптах запуску для отримання постійного доступу до пристрою під час перезавантаження. Ці уразливості виникають, коли скрипти запуску посилаються, [створюють символічні посилання](https://www.chromium.org/chromium-os/chromiumos-design-docs/hardening-against-malicious-stateful-data) або залежать від коду, розташованого в ненадійних змонтованих місцях, таких як SD-карти та флеш-накопичувачі, які використовуються для зберігання даних поза кореневими файловими системами.

## Посилання
* Для отримання додаткової інформації перегляньте [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
