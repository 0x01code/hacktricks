# FZ - 125kHz RFID

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

## Вступ

Для отримання додаткової інформації про те, як працюють теги 125 кГц, перегляньте:

{% content-ref url="../../../radio-hacking/pentesting-rfid.md" %}
[pentesting-rfid.md](../../../radio-hacking/pentesting-rfid.md)
{% endcontent-ref %}

## Дії

Для отримання додаткової інформації про ці типи тегів [**прочитайте цей вступ**](../../../radio-hacking/pentesting-rfid.md#low-frequency-rfid-tags-125khz).

### Читання

Спробує **прочитати** інформацію з картки. Потім може **емулювати** їх.

{% hint style="warning" %}
Зверніть увагу, що деякі домофони намагаються захистити себе від копіювання ключів, відправляючи команду запису перед читанням. Якщо запис вдається, цей тег вважається фальшивим. Коли Flipper емулює RFID, читач не може відрізнити його від оригінального, тому такі проблеми не виникають.
{% endhint %}

### Додати вручну

Ви можете створити **фальшиві картки в Flipper Zero, вказавши дані** вручну, а потім емулювати їх.

#### Ідентифікатори на картках

Іноді, коли ви отримуєте картку, ви знайдете ID (або його частину), написаний на картці.

* **EM Marin**

Наприклад, на цій картці EM-Marin на фізичній картці можна **прочитати останні 3 з 5 байтів чітко**.\
Інші 2 можна перебрати методом спроб та помилок, якщо ви не можете прочитати їх з картки.

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

* **HID**

Те ж саме відбувається на цій картці HID, де тільки 2 з 3 байтів можна знайти надрукованими на картці

<figure><img src="../../../.gitbook/assets/image (15) (3).png" alt=""><figcaption></figcaption></figure>

### Емулювати/Записати

Після **копіювання** картки або **введення** ID **вручну**, можна **емулювати** її за допомогою Flipper Zero або **записати** на реальну картку.

## Посилання

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
