# FZ - Sub-GHz

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Вступ <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero може **приймати та передавати радіочастоти в діапазоні від 300 до 928 МГц** за допомогою вбудованого модуля, який може зчитувати, зберігати та емулювати пульт дистанційного керування. Ці пульти використовуються для взаємодії з воротами, бар'єрами, радіозамками, вимикачами дистанційного керування, бездротовими дзвіночками, розумними лампами та іншими пристроями. Flipper Zero може допомогти вам дізнатися, чи була порушена ваша безпека.

<figure><img src="../../../.gitbook/assets/image (3) (2) (1).png" alt=""><figcaption></figcaption></figure>

## Апаратне забезпечення Sub-GHz <a href="#kfpn7" id="kfpn7"></a>

У Flipper Zero є вбудований модуль sub-1 ГГц на основі чіпа [﻿](https://www.st.com/en/nfc/st25r3916.html#overview)﻿[CC1101](https://www.ti.com/lit/ds/symlink/cc1101.pdf) та радіоантени (максимальна дальність - 50 метрів). Як чіп CC1101, так і антена призначені для роботи на частотах у діапазонах 300-348 МГц, 387-464 МГц та 779-928 МГц.

<figure><img src="../../../.gitbook/assets/image (1) (8) (1).png" alt=""><figcaption></figcaption></figure>

## Дії

### Аналізатор частот

{% hint style="info" %}
Як знайти, яку частоту використовує пульт
{% endhint %}

Під час аналізування Flipper Zero сканує сили сигналів (RSSI) на всіх доступних частотах у конфігурації частот. Flipper Zero відображає частоту з найвищим значенням RSSI, зі силою сигналу вище -90 [dBm](https://en.wikipedia.org/wiki/DBm).

Щоб визначити частоту пульта, виконайте наступне:

1. Помістіть пульт дистанційного керування дуже близько до лівої частини Flipper Zero.
2. Перейдіть до **Головного меню** **→ Sub-GHz**.
3. Виберіть **Аналізатор частот**, потім натисніть і утримуйте кнопку на пульті дистанційного керування, який ви хочете проаналізувати.
4. Перегляньте значення частоти на екрані.

### Читати

{% hint style="info" %}
Дізнайтеся інформацію про використану частоту (також інший спосіб знайти використану частоту)
{% endhint %}

Опція **Читати** **прослуховує на налаштованій частоті** на вказаній модуляції: 433,92 AM за замовчуванням. Якщо **щось знайдено** під час читання, **надається інформація** на екрані. Цю інформацію можна використовувати для реплікації сигналу у майбутньому.

Під час використання опції Читати можна натиснути **ліву кнопку** та **налаштувати її**.\
На даний момент є **4 модуляції** (AM270, AM650, FM328 та FM476), і **кілька важливих частот** збережено:

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Ви можете встановити **будь-яку, яка вас цікавить**, однак, якщо ви **не впевнені, яка частота** може бути використана пультом, **ввімкніть Hopping на ON** (за замовчуванням вимкнено) та кілька разів натисніть кнопку, поки Flipper не зафіксує її і не надасть вам інформацію, яку вам потрібно встановити частоту.

{% hint style="danger" %}
Перемикання між частотами займає певний час, тому сигнали, передані під час перемикання, можуть бути пропущені. Для кращого прийому сигналу встановіть фіксовану частоту, визначену Аналізатором частот.
{% endhint %}

### **Читати Raw**

{% hint style="info" %}
Вкрасти (і відтворити) сигнал на налаштованій частоті
{% endhint %}

Опція **Читати Raw** **записує сигнали**, відправлені на прослуховувану частоту. Це можна використовувати для **викрадення** сигналу та **повторного відтворення** його.

За замовчуванням **Читати Raw також в 433,92 у AM650**, але якщо з опцією Читати ви виявили, що сигнал, який вас цікавить, знаходиться на **іншій частоті/модуляції, ви також можете змінити це**, натиснувши ліву кнопку (під час перебування в опції Читати Raw).

### Брутфорс

Якщо ви знаєте протокол, який використовується, наприклад, для гаражних воріт, можна **згенерувати всі коди та відправити їх за допомогою Flipper Zero**. Це приклад, який підтримує загальні типи гаражів: [**https://github.com/tobiabocchi/flipperzero-bruteforce**](https://github.com/tobiabocchi/flipperzero-bruteforce)

### Додати вручну

{% hint style="info" %}
Додати сигнали з налаштованого списку протоколів
{% endhint %}

#### Список [підтримуваних протоколів](https://docs.flipperzero.one/sub-ghz/add-new-remote) <a href="#id-3iglu" id="id-3iglu"></a>

| Princeton\_433 (працює з більшістю статичних кодових систем) | 433,92 | Статичний |
| ----------------------------------------------------------- | ------ | --------- |
| Nice Flo 12bit\_433                                         | 433,92 | Статичний |
| Nice Flo 24bit\_433                                         | 433,92 | Статичний |
| CAME 12bit\_433                                             | 433,92 | Статичний |
| CAME 24bit\_433                                             | 433,92 | Статичний |
| Linear\_300                                                 | 300,00 | Статичний |
| CAME TWEE                                                   | 433,92 | Статичний |
| Gate TX\_433                                                | 433,92 | Статичний |
| DoorHan\_315                                                | 315,00 | Динамічний |
| DoorHan\_433                                                | 433,92 | Динамічний |
| LiftMaster\_315                                             | 315,00 | Динамічний |
| LiftMaster\_390                                             | 390,00 | Динамічний |
| Security+2.0\_310                                           | 310,00 | Динамічний |
| Security+2.0\_315                                           | 315,00 | Динамічний |
| Security+2.0\_390                                           | 390,00 | Динамічний |
### Підтримувані виробники Sub-GHz

Перевірте список за посиланням [https://docs.flipperzero.one/sub-ghz/supported-vendors](https://docs.flipperzero.one/sub-ghz/supported-vendors)

### Підтримувані частоти за регіонами

Перевірте список за посиланням [https://docs.flipperzero.one/sub-ghz/frequencies](https://docs.flipperzero.one/sub-ghz/frequencies)

### Тест

{% hint style="info" %}
Отримайте dBms збережених частот
{% endhint %}

## Посилання

* [https://docs.flipperzero.one/sub-ghz](https://docs.flipperzero.one/sub-ghz)

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>
