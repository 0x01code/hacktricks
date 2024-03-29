<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-прийомами, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

Рекомендовані кроки для модифікації конфігурацій запуску пристрою та завантажувачів, таких як U-boot:

1. **Доступ до інтерпретатора завантажувача**:
- Під час завантаження натисніть "0", пробіл або інші визначені "магічні коди", щоб отримати доступ до інтерпретатора завантажувача.

2. **Змінити аргументи завантаження**:
- Виконайте наступні команди, щоб додати '`init=/bin/sh`' до аргументів завантаження, що дозволить виконання команди оболонки:
%%%
#printenv
#setenv bootargs=console=ttyS0,115200 mem=63M root=/dev/mtdblock3 mtdparts=sflash:<partitiionInfo> rootfstype=<fstype> hasEeprom=0 5srst=0 init=/bin/sh
#saveenv
#boot
%%%

3. **Налаштувати сервер TFTP**:
- Налаштуйте сервер TFTP для завантаження зображень через локальну мережу:
%%%
#setenv ipaddr 192.168.2.2 #локальна IP-адреса пристрою
#setenv serverip 192.168.2.1 #IP-адреса сервера TFTP
#saveenv
#reset
#ping 192.168.2.1 #перевірте доступ до мережі
#tftp ${loadaddr} uImage-3.6.35 #loadaddr бере адресу для завантаження файлу та ім'я файлу зображення на сервері TFTP
%%%

4. **Використовуйте `ubootwrite.py`**:
- Використовуйте `ubootwrite.py` для запису зображення U-boot та відправлення модифікованої прошивки для отримання root-доступу.

5. **Перевірте відладкові функції**:
- Перевірте, чи увімкнені відладкові функції, такі як розширене ведення журналу, завантаження довільних ядер або завантаження з ненадійних джерел.

6. **Обережне втручання у апаратне забезпечення**:
- Будьте обережні при підключенні одного контакту до землі та взаємодії з чіпами SPI або NAND Flash під час послідовності завантаження пристрою, зокрема до розпакування ядра. Перед замиканням контактів перевірте технічні характеристики чіпа NAND Flash.

7. **Налаштувати підроблений сервер DHCP**:
- Налаштуйте підроблений сервер DHCP зі зловмисними параметрами для пристрою, які він прийматиме під час PXE-завантаження. Використовуйте інструменти, такі як допоміжний сервер DHCP Metasploit (MSF). Змініть параметр 'FILENAME' на команди введення команд впровадження, такі як `'a";/bin/sh;#'` для перевірки валідації введення для процедур запуску пристрою.

**Примітка**: Кроки, що передбачають фізичну взаємодію з контактами пристрою (*позначені зірочкою), слід підходити з надзвичайною обережністю, щоб уникнути пошкодження пристрою.


## References
* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-прийомами, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
