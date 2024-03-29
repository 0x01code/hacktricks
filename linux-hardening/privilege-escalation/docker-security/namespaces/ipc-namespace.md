# Простір імен IPC

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв GitHub**.

</details>

## Основна інформація

Простір імен IPC (Inter-Process Communication) - це функція ядра Linux, яка забезпечує **ізоляцію** об'єктів System V IPC, таких як черги повідомлень, сегменти спільної пам'яті та семафори. Ця ізоляція гарантує, що процеси в **різних просторах імен IPC не можуть безпосередньо отримувати доступ або змінювати об'єкти IPC один одного**, забезпечуючи додатковий рівень безпеки та конфіденційності між групами процесів.

### Як це працює:

1. При створенні нового простору імен IPC він починається з **повністю ізольованого набору об'єктів System V IPC**. Це означає, що процеси, які працюють в новому просторі імен IPC, не можуть отримувати доступ або втручатися в об'єкти IPC в інших просторах імен або на системі хоста за замовчуванням.
2. Об'єкти IPC, створені в межах простору імен, видимі та **доступні лише процесам у цьому просторі імен**. Кожен об'єкт IPC ідентифікується унікальним ключем у межах свого простору імен. Хоча ключ може бути ідентичним у різних просторах імен, самі об'єкти ізольовані і не можуть бути доступні в інших просторах імен.
3. Процеси можуть переміщатися між просторами імен за допомогою системного виклику `setns()` або створювати нові простори імен за допомогою системних викликів `unshare()` або `clone()` з прапорцем `CLONE_NEWIPC`. Коли процес переходить до нового простору імен або створює його, він почне використовувати об'єкти IPC, пов'язані з цим простором імен.

## Лабораторія:

### Створення різних просторів імен

#### CLI
```bash
sudo unshare -i [--mount-proc] /bin/bash
```
Монтувавши новий екземпляр файлової системи `/proc`, якщо використовується параметр `--mount-proc`, ви забезпечуєте, що новий простір імен має **точний та ізольований вид інформації про процес, специфічний для цього простору імен**.

<details>

<summary>Помилка: bash: fork: Неможливо виділити пам'ять</summary>

Коли `unshare` виконується без опції `-f`, виникає помилка через те, як Linux обробляє нові простори імен PID (ідентифікатори процесів). Основні деталі та рішення наведено нижче:

1. **Пояснення проблеми**:
- Ядро Linux дозволяє процесу створювати нові простори імен за допомогою системного виклику `unshare`. Однак процес, який ініціює створення нового простору імен PID (відомий як "процес unshare"), не входить в новий простір імен; лише його дочірні процеси роблять це.
- Виконання `%unshare -p /bin/bash%` запускає `/bin/bash` в тому ж процесі, що і `unshare`. В результаті `/bin/bash` та його дочірні процеси знаходяться в початковому просторі імен PID.
- Перший дочірній процес `/bin/bash` в новому просторі імен стає PID 1. Коли цей процес завершується, він спричиняє очищення простору імен, якщо інших процесів немає, оскільки PID 1 має спеціальну роль усиновлення сиріт. Після цього ядро Linux вимикає виділення PID в цьому просторі імен.

2. **Наслідок**:
- Виходження PID 1 в новому просторі імен призводить до очищення прапорця `PIDNS_HASH_ADDING`. Це призводить до невдалого виділення PID при створенні нового процесу, що призводить до помилки "Неможливо виділити пам'ять".

3. **Рішення**:
- Проблему можна вирішити, використовуючи опцію `-f` з `unshare`. Ця опція змушує `unshare` розгалужувати новий процес після створення нового простору імен PID.
- Виконання `%unshare -fp /bin/bash%` забезпечує, що сама команда `unshare` стає PID 1 в новому просторі імен. `/bin/bash` та його дочірні процеси потім безпечно знаходяться в цьому новому просторі імен, запобігаючи передчасному виходу PID 1 та дозволяючи нормальне виділення PID.

Забезпечивши, що `unshare` працює з прапорцем `-f`, новий простір імен PID правильно підтримується, що дозволяє `/bin/bash` та його підпроцесам працювати без зустрічі помилки виділення пам'яті.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;Перевірте, в якому просторі імені знаходиться ваш процес
```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```
### Знайдіть всі IPC-простори імен

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
### Увійдіть всередину простору імен IPC
```bash
nsenter -i TARGET_PID --pid /bin/bash
```
Також, ви можете **увійти в простір імен іншого процесу лише як root**. І ви **не можете** **увійти** в інший простір імен **без дескриптора**, що вказує на нього (наприклад, `/proc/self/ns/net`).

### Створення об'єкту IPC
```bash
# Container
sudo unshare -i /bin/bash
ipcmk -M 100
Shared memory id: 0
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x2fba9021 0          root       644        100        0

# From the host
ipcs -m # Nothing is seen
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
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
