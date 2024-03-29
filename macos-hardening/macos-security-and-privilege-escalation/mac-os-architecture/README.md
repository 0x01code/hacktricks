# Ядро macOS та Системні Розширення

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>

## Ядро XNU

**Основа macOS - це XNU**, що означає "X не є Unix". Це ядро фундаментально складається з **мікроядра Mach** (про яке буде розповідь пізніше), **та** елементів з Berkeley Software Distribution (**BSD**). XNU також надає платформу для **ядерних драйверів через систему, що називається I/O Kit**. Ядро XNU є частиною проекту з відкритим кодом Darwin, що означає, що **його вихідний код доступний безкоштовно**.

З погляду дослідника з безпеки або розробника Unix, **macOS** може виглядати досить **схожим** на систему **FreeBSD** з елегантним графічним інтерфейсом та низкою власних додатків. Більшість додатків, розроблених для BSD, будуть компілюватися та працювати на macOS без необхідності внесення змін, оскільки командні інструменти, знайомі користувачам Unix, присутні в macOS. Однак, через те, що ядро XNU включає Mach, існують деякі значні відмінності між традиційною системою, схожою на Unix, та macOS, і ці відмінності можуть викликати потенційні проблеми або надавати унікальні переваги.

Відкрита версія XNU: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach - це **мікроядро**, призначене бути **сумісним з UNIX**. Одним з його ключових принципів дизайну було **мінімізування** кількості **коду**, що виконується в **просторі ядра**, і натомість дозволяти багатьом типовим функціям ядра, таким як файлова система, мережа та введення/виведення, **виконуватися як завдання на рівні користувача**.

У XNU Mach **відповідає за багато критичних операцій на низькому рівні**, які зазвичай обробляє ядро, такі як планування процесора, багатозадачність та управління віртуальною пам'яттю.

### BSD

Ядро XNU також **включає** значну кількість коду, походящого з проекту **FreeBSD**. Цей код **виконується як частина ядра разом з Mach**, в тому ж адресному просторі. Однак код FreeBSD в XNU може суттєво відрізнятися від оригінального коду FreeBSD через необхідні модифікації для забезпечення його сумісності з Mach. FreeBSD сприяє багатьом операціям ядра, включаючи:

* Управління процесами
* Обробка сигналів
* Основні механізми безпеки, включаючи управління користувачами та групами
* Інфраструктура системних викликів
* TCP/IP стек та сокети
* Брандмауер та фільтрація пакетів

Розуміння взаємодії між BSD та Mach може бути складним через їх різні концептуальні каркаси. Наприклад, BSD використовує процеси як свою фундаментальну одиницю виконання, тоді як Mach працює на основі потоків. Ця розбіжність узгоджується в XNU шляхом **асоціювання кожного процесу BSD з завданням Mach**, яке містить рівно один потік Mach. Коли використовується системний виклик fork() BSD, код BSD в ядрі використовує функції Mach для створення структури завдання та потоку.

Більше того, **Mach та BSD кожен мають власні моделі безпеки**: **модель безпеки Mach базується на правах портів**, тоді як модель безпеки BSD працює на основі **власності процесу**. Розбіжності між цими двома моделями іноді призводили до вразливостей локального підвищення привілеїв. Окрім типових системних викликів, також існують **пастки Mach, які дозволяють програмам простору користувача взаємодіяти з ядром**. Ці різні елементи разом формують багатогранну, гібридну архітектуру ядра macOS.

### I/O Kit - Драйвери

I/O Kit - це відкритий, об'єктно-орієнтований **фреймворк драйверів пристроїв** в ядрі XNU, який обробляє **динамічно завантажені драйвери пристроїв**. Він дозволяє додавати модульний код до ядра на льоту, підтримуючи різноманітне обладнання.

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - Міжпроцесова Комунікація

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Kernelcache

**Kernelcache** - це **попередньо скомпільована та попередньо зв'язана версія ядра XNU**, разом з основними драйверами пристроїв та **розширеннями ядра**. Він зберігається у **стиснутому** форматі та розпаковується в пам'ять під час процесу завантаження. Kernelcache сприяє **швидшому часу завантаження** за рахунок наявності готової до запуску версії ядра та важливих драйверів, що дозволяє зменшити час та ресурси, які інакше б витрачалися на динамічне завантаження та зв'язування цих компонентів під час завантаження.

У iOS він розташований у **`/System/Library/Caches/com.apple.kernelcaches/kernelcache`**, у macOS ви можете знайти його за допомогою **`find / -name kernelcache 2>/dev/null`** або **`mdfind kernelcache | grep kernelcache`**

Можливо виконати **`kextstat`** для перевірки завантажених розширень ядра.

#### IMG4

Формат файлу IMG4 - це контейнерний формат, який використовується Apple у своїх пристроях iOS та macOS для безпечного **зберігання та перевірки компонентів прошивки** (наприклад, **kernelcache**). Формат IMG4 включає заголовок та кілька тегів, які упаковують різні частини даних, включаючи фактичний навантаження (наприклад, ядро або завантажувач), підпис, та набір властивостей маніфесту. Формат підтримує криптографічну перевірку, що дозволяє пристрою підтвердити автентичність та цілісність компонента прошивки перед його виконанням.

Зазвичай складається з наступних компонентів:

* **Навантаження (IM4P)**:
* Часто стиснене (LZFSE4, LZSS, ...)
* Опціонально зашифроване
* **Маніфест (IM4M)**:
* Містить підпис
* Додатковий словник Ключ/Значення
* **Інформація про відновлення (IM4R)**:
* Також відомий як APNonce
* Запобігає повторному відтворенню деяких оновлень
* ОПЦІЙНО: Зазвичай цього не знаходять

Розпакуйте Kernelcache:
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### Символи ядра

Іноді Apple випускає **ядро кешу** з **символами**. Ви можете завантажити деякі прошивки з символами, перейшовши за посиланням на [https://theapplewiki.com](https://theapplewiki.com/).

### IPSW

Це прошивки Apple, які можна завантажити з [**https://ipsw.me/**](https://ipsw.me/). Серед інших файлів вони містять **ядро кешу**.\
Для **вилучення** файлів ви можете просто їх **розпакувати**.

Після розпакування прошивки ви отримаєте файл, наприклад: **`kernelcache.release.iphone14`**. Він у форматі **IMG4**, ви можете витягнути цікаву інформацію за допомогою:

* [**pyimg4**](https://github.com/m1stadev/PyIMG4)

{% code overflow="wrap" %}
```bash
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
{% endcode %}

* [**img4tool**](https://github.com/tihmstar/img4tool)
```bash
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
Ви можете перевірити витягнутий ядро для символів за допомогою: **`nm -a kernelcache.release.iphone14.e | wc -l`**

З цим ми тепер можемо **витягнути всі розширення** або **те, що вас цікавить:**
```bash
# List all extensions
kextex -l kernelcache.release.iphone14.e
## Extract com.apple.security.sandbox
kextex -e com.apple.security.sandbox kernelcache.release.iphone14.e

# Extract all
kextex_all kernelcache.release.iphone14.e

# Check the extension for symbols
nm -a binaries/com.apple.security.sandbox | wc -l
```
## Розширення ядра macOS

macOS **дуже обмежує завантаження розширень ядра** (.kext) через високі привілеї, з якими цей код буде виконуватися. Фактично, за замовчуванням це практично неможливо (якщо не знайдено обхід).

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### Розширення системи macOS

Замість використання Розширень ядра macOS створила Розширення системи, які надають API рівня користувача для взаємодії з ядром. Таким чином, розробники можуть уникнути використання розширень ядра.

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## Посилання

* [**Посібник хакера Mac**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
