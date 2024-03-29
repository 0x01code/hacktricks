<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>


# JTAGenum

[**JTAGenum** ](https://github.com/cyphunk/JTAGenum) - це інструмент, який можна використовувати з Raspberry PI або Arduino для пошуку JTAG-контактів на невідомому чіпі.\
У **Arduino** підключіть **контакти від 2 до 11 до 10 контактів, які можуть належати до JTAG**. Завантажте програму в Arduino, і вона спробує перебрати всі контакти, щоб знайти, чи належать які-небудь контакти до JTAG, і які саме.\
У **Raspberry PI** ви можете використовувати лише **контакти від 1 до 6** (6 контактів, тому ви будете повільніше перевіряти кожен потенційний JTAG-контакт).

## Arduino

У Arduino, після підключення кабелів (контакти від 2 до 11 до контактів JTAG та GND Arduino до GND базової плати), **завантажте програму JTAGenum в Arduino** і відправте **`h`** в моніторі послідовного порту, щоб отримати довідку:

![](<../../.gitbook/assets/image (643).png>)

![](<../../.gitbook/assets/image (650).png>)

Налаштуйте **"Без завершення рядка" та 115200 бод**.\
Надішліть команду s, щоб почати сканування:

![](<../../.gitbook/assets/image (651) (1) (1) (1).png>)

Якщо ви звертаєтеся до JTAG, ви знайдете один або кілька **рядків, що починаються з FOUND!**, що вказують на контакти JTAG.

</details>
