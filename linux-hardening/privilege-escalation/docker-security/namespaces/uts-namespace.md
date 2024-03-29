# Простір імен UTS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Основна інформація

Простір імен UTS (UNIX Time-Sharing System) - це функція ядра Linux, яка забезпечує **ізоляцію двох системних ідентифікаторів**: **ім'я хоста** та **доменне ім'я NIS** (мережевий інформаційний сервіс). Ця ізоляція дозволяє кожному простору імен UTS мати своє **власне незалежне ім'я хоста та доменне ім'я NIS**, що є особливо корисним у сценаріях контейнеризації, де кожен контейнер повинен виглядати як окрема система з власним ім'ям хоста.

### Як це працює:

1. При створенні нового простору імен UTS він починається з **копії імені хоста та доменного імені NIS від батьківського простору імен**. Це означає, що при створенні нового простору імен він **спільний з ідентифікаторами батьківського простору імен**. Однак будь-які подальші зміни імені хоста або доменного імені NIS в межах простору імен не впливатимуть на інші простори імен.
2. Процеси в межах простору імен UTS **можуть змінювати ім'я хоста та доменне ім'я NIS**, використовуючи відповідно системні виклики `sethostname()` та `setdomainname()`. Ці зміни є локальними для простору імен і не впливають на інші простори імен або головну систему.
3. Процеси можуть переміщатися між просторами імен за допомогою системного виклику `setns()` або створювати нові простори імен за допомогою системних викликів `unshare()` або `clone()` з прапорцем `CLONE_NEWUTS`. Коли процес переходить до нового простору імен або створює його, він почне використовувати ім'я хоста та доменне ім'я NIS, що пов'язані з цим простором імен.

## Лабораторія:

### Створення різних просторів імен

#### CLI
```bash
sudo unshare -u [--mount-proc] /bin/bash
```
Монтувавши новий екземпляр файлової системи `/proc`, якщо використовуєте параметр `--mount-proc`, ви забезпечуєте, що новий простір імен має **точний та ізольований вид інформації про процес, специфічний для цього простору імен**.

<details>

<summary>Помилка: bash: fork: Неможливо виділити пам'ять</summary>

Коли `unshare` виконується без опції `-f`, виникає помилка через те, як Linux обробляє нові простори імен PID (ідентифікатори процесів). Основні деталі та рішення наведено нижче:

1. **Пояснення проблеми**:
- Ядро Linux дозволяє процесу створювати нові простори імен за допомогою системного виклику `unshare`. Однак процес, який ініціює створення нового простору імен PID (відомий як "процес unshare"), не входить в новий простір імен; лише його дочірні процеси.
- Виконання `%unshare -p /bin/bash%` запускає `/bin/bash` в тому ж процесі, що і `unshare`. В результаті `/bin/bash` та його дочірні процеси знаходяться в початковому просторі імен PID.
- Перший дочірній процес `/bin/bash` в новому просторі імен стає PID 1. Коли цей процес завершується, він спричиняє очищення простору імен, якщо інших процесів немає, оскільки PID 1 має спеціальну роль у прийнятті сирітських процесів. Після цього ядро Linux вимикає виділення PID в цьому просторі імен.

2. **Наслідок**:
- Виходження PID 1 в новому просторі імен призводить до очищення прапорця `PIDNS_HASH_ADDING`. Це призводить до невдалого виділення PID під час створення нового процесу, що призводить до помилки "Неможливо виділити пам'ять".

3. **Рішення**:
- Проблему можна вирішити, використовуючи опцію `-f` з `unshare`. Ця опція змушує `unshare` розгалужувати новий процес після створення нового простору імен PID.
- Виконання `%unshare -fp /bin/bash%` забезпечує, що сама команда `unshare` стає PID 1 в новому просторі імен. `/bin/bash` та його дочірні процеси потім безпечно знаходяться в цьому новому просторі імен, запобігаючи передчасному виходу PID 1 та дозволяючи нормальне виділення PID.

Забезпечуючи, що `unshare` працює з прапорцем `-f`, новий простір імен PID правильно підтримується, що дозволяє `/bin/bash` та його підпроцесам працювати без зустрічі помилки виділення пам'яті.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;Перевірте, в якому просторі імені знаходиться ваш процес
```bash
ls -l /proc/self/ns/uts
lrwxrwxrwx 1 root root 0 Apr  4 20:49 /proc/self/ns/uts -> 'uts:[4026531838]'
```
### Знайдіть всі простори імен UTS

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name uts -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name uts -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
### Увійдіть всередину простору імен UTS
```bash
nsenter -u TARGET_PID --pid /bin/bash
```
Також, ви можете **увійти в інший простір процесу лише як root**. І ви **не можете** **увійти** в інший простір без дескриптора, що вказує на нього (наприклад, `/proc/self/ns/uts`).

### Змінити ім'я хоста
```bash
unshare -u /bin/bash
hostname newhostname # Hostname won't be changed inside the host UTS ns
```
## Посилання
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
