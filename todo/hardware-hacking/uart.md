<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>


# Основна інформація

UART - це послідовний протокол, що означає, що він передає дані між компонентами по одному біту за раз. У порівнянні з цим, паралельні комунікаційні протоколи передають дані одночасно через кілька каналів. До загальних послідовних протоколів відносяться RS-232, I2C, SPI, CAN, Ethernet, HDMI, PCI Express та USB.

Загалом, лінія утримується на високому рівні (значення логічної 1), коли UART перебуває в стані очікування. Потім, для сигналізації початку передачі даних, передавач надсилає приймачу стартовий біт, під час якого сигнал утримується на низькому рівні (значення логічної 0). Далі передавач надсилає п'ять до восьми бітів даних, що містять фактичне повідомлення, за якими слідує необов'язковий біт парності та один або два стоп-біти (зі значенням логічної 1), в залежності від конфігурації. Біт парності, який використовується для перевірки помилок, рідко бачиться на практиці. Стоп-біт (або біти) позначають кінець передачі.

Ми називаємо найбільш поширену конфігурацію 8N1: вісім бітів даних, без парності та один стоп-біт. Наприклад, якщо ми хочемо відправити символ C, або 0x43 у ASCII, в конфігурації UART 8N1, ми відправимо наступні біти: 0 (стартовий біт); 0, 1, 0, 0, 0, 0, 1, 1 (значення 0x43 у двійковій системі), та 0 (стоп-біт).

![](<../../.gitbook/assets/image (648) (1) (1) (1) (1).png>)

Апаратні засоби для спілкування з UART:

* USB-серійний адаптер
* Адаптери з чіпами CP2102 або PL2303
* Універсальний інструмент, такий як: Bus Pirate, Adafruit FT232H, Shikra або Attify Badge

## Визначення портів UART

UART має 4 порти: **TX**(Transmit), **RX**(Receive), **Vcc**(Voltage) та **GND**(Ground). Ви можете знайти 4 порти з літерами **`TX`** та **`RX`** **написаними** на платі. Але якщо немає позначення, вам може знадобитися спробувати знайти їх самостійно за допомогою **мультиметра** або **логічного аналізатора**.

З **мультиметром** та вимкненим пристроєм:

* Для визначення піна **GND** використовуйте режим **Тесту на з'єднаність**, помістіть задній вивід в землю та перевірте червоним до тих пір, поки не почуєте звук від мультиметра. На платі можна знайти кілька пінів **GND**, тому ви можете знайти або не знайти той, що належить до UART.
* Для визначення порту **VCC**, встановіть режим **постійної напруги** та встановіть його на 20 Вольт напруги. Чорна зонд на землю, а червона на пін. Увімкніть пристрій. Якщо мультиметр вимірює постійну напругу 3,3 В або 5 В, ви знайшли пін Vcc. Якщо ви отримуєте інші напруги, спробуйте із іншими портами.
* Для визначення порту **TX**, режим **постійної напруги** до 20 Вольт напруги, чорна зонд на землю, червона на пін, та увімкніть пристрій. Якщо ви виявите, що напруга коливається протягом кількох секунд, а потім стабілізується на значенні Vcc, ви, ймовірно, знайшли порт TX. Це тому, що при увімкненні він відправляє деякі відлагоджувальні дані.
* Порт **RX** буде найближчим до інших 3, він має найменші коливання напруги та найменше загальне значення серед усіх пінів UART.

Ви можете поміняти порти TX та RX, і нічого не станеться, але якщо ви помилитеся між GND та портом VCC, ви можете зруйнувати схему.

З логічним аналізатором:

## Визначення швидкості передачі UART

Найпростіший спосіб визначити правильну швидкість передачі - це подивитися на вивід піна **TX та спробувати прочитати дані**. Якщо отримані дані нечитабельні, переключайтеся на наступну можливу швидкість передачі, поки дані не стануть читабельними. Для цього можна використовувати USB-серійний адаптер або універсальний пристрій, такий як Bus Pirate, у поєднанні з допоміжним скриптом, таким як [baudrate.py](https://github.com/devttys0/baudrate/). Найпоширеніші швидкості передачі - 9600, 38400, 19200, 57600 та 115200.

{% hint style="danger" %}
Важливо зауважити, що в цьому протоколі потрібно підключити TX одного пристрою до RX іншого!
{% endhint %}

# Bus Pirate

У цьому сценарії ми будемо перехоплювати UART-комунікацію Arduino, яка надсилає всі виводи програми на монітор Серійного порту.
```bash
# Check the modes
UART>m
1. HiZ
2. 1-WIRE
3. UART
4. I2C
5. SPI
6. 2WIRE
7. 3WIRE
8. KEYB
9. LCD
10. PIC
11. DIO
x. exit(without change)

# Select UART
(1)>3
Set serial port speed: (bps)
1. 300
2. 1200
3. 2400
4. 4800
5. 9600
6. 19200
7. 38400
8. 57600
9. 115200
10. BRG raw value

# Select the speed the communication is occurring on (you BF all this until you find readable things)
# Or you could later use the macro (4) to try to find the speed
(1)>5
Data bits and parity:
1. 8, NONE *default
2. 8, EVEN
3. 8, ODD
4. 9, NONE

# From now on pulse enter for default
(1)>
Stop bits:
1. 1 *default
2. 2
(1)>
Receive polarity:
1. Idle 1 *default
2. Idle 0
(1)>
Select output type:
1. Open drain (H=Hi-Z, L=GND)
2. Normal (H=3.3V, L=GND)

(1)>
Clutch disengaged!!!
To finish setup, start up the power supplies with command 'W'
Ready

# Start
UART>W
POWER SUPPLIES ON
Clutch engaged!!!

# Use macro (2) to read the data of the bus (live monitor)
UART>(2)
Raw UART input
Any key to exit
Escritura inicial completada:
AAA Hi Dreg! AAA
waiting a few secs to repeat....
```
<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
