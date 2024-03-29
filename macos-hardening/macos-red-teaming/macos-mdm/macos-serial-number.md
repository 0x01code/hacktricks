# Серійний номер macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>


## Основна інформація

У пристроїв Apple після 2010 року серійні номери складаються з **12 буквено-цифрових символів**, кожен з яких несе певну інформацію:

- **Перші 3 символи**: Вказують на **місце виробництва**.
- **Символи 4 та 5**: Позначають **рік та тиждень виготовлення**.
- **Символи 6-8**: Служать як **унікальний ідентифікатор** для кожного пристрою.
- **Останні 4 символи**: Вказують на **номер моделі**.

Наприклад, серійний номер **C02L13ECF8J2** відповідає цій структурі.

### **Місця виробництва (Перші 3 символи)**
Деякі коди вказують на конкретні заводи:
- **FC, F, XA/XB/QP/G8**: Різні місця в США.
- **RN**: Мексика.
- **CK**: Корк, Ірландія.
- **VM**: Foxconn, Чехія.
- **SG/E**: Сінгапур.
- **MB**: Малайзія.
- **PT/CY**: Корея.
- **EE/QT/UV**: Тайвань.
- **FK/F1/F2, W8, DL/DM, DN, YM/7J, 1C/4H/WQ/F7**: Різні місця в Китаї.
- **C0, C3, C7**: Конкретні міста в Китаї.
- **RM**: Відновлені пристрої.

### **Рік виробництва (4-й символ)**
Цей символ змінюється від 'C' (перша половина 2010 року) до 'Z' (друга половина 2019 року), різні літери позначають різні піврічні періоди.

### **Тиждень виробництва (5-й символ)**
Цифри 1-9 відповідають тижням 1-9. Літери C-Y (крім голосних та 'S') позначають тижні 10-27. Для другої половини року до цього числа додається 26.

### **Унікальний ідентифікатор (Символи 6-8)**
Ці три цифри забезпечують, що кожен пристрій, навіть однієї моделі та партії, має унікальний серійний номер.

### **Номер моделі (Останні 4 символи)**
Ці цифри ідентифікують конкретну модель пристрою.

### Посилання

* [https://beetstech.com/blog/decode-meaning-behind-apple-serial-number](https://beetstech.com/blog/decode-meaning-behind-apple-serial-number)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
