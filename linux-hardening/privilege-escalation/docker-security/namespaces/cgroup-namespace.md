# Простір імен CGroup

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

## Основна інформація

Простір імен CGroup - це функція ядра Linux, яка забезпечує **ізоляцію ієрархій cgroup для процесів, що працюють у межах простору імен**. Cgroups, що скорочено від **control groups**, - це функція ядра, яка дозволяє організовувати процеси в ієрархічні групи для управління та забезпечення **обмежень на системні ресурси**, такі як ЦП, пам'ять та введення/виведення.

Хоча простори імен cgroup не є окремим типом простору імен, як інші, про які ми говорили раніше (PID, mount, network і т. д.), вони пов'язані з концепцією ізоляції просторів імен. **Простори імен cgroup віртуалізують вид ієрархії cgroup**, так що процеси, що працюють у просторі імен cgroup, мають інший вид ієрархії порівняно з процесами, що працюють на хості або в інших просторах імен.

### Як це працює:

1. При створенні нового простору імен cgroup **він починається з виду ієрархії cgroup на основі cgroup створюючого процесу**. Це означає, що процеси, що працюють у новому просторі імен cgroup, побачать лише підмножину всієї ієрархії cgroup, обмежену піддеревом cgroup, що має корінь у cgroup створюючого процесу.
2. Процеси у просторі імен cgroup **будуть бачити свій власний cgroup як корінь ієрархії**. Це означає, що з точки зору процесів у просторі імен, їх власний cgroup виглядає як корінь, і вони не можуть бачити або отримати доступ до cgroup поза власним піддеревом.
3. Простори імен cgroup не надають прямої ізоляції ресурсів; **вони надають лише ізоляцію виду ієрархії cgroup**. **Контроль та ізоляція ресурсів все ще забезпечуються підсистемами cgroup** (наприклад, центральний процесор, пам'ять і т. д.).

Для отримання додаткової інформації про CGroups перегляньте:

{% content-ref url="../cgroups.md" %}
[cgroups.md](../cgroups.md)
{% endcontent-ref %}

## Лабораторія:

### Створення різних просторів імен

#### CLI
```bash
sudo unshare -C [--mount-proc] /bin/bash
```
Монтувавши новий екземпляр файлової системи `/proc`, якщо використовується параметр `--mount-proc`, ви забезпечуєте, що новий простір імен має **точний та ізольований вид інформації про процес, специфічний для цього простору імен**.

<details>

<summary>Помилка: bash: fork: Неможливо виділити пам'ять</summary>

Коли `unshare` виконується без опції `-f`, виникає помилка через те, як Linux обробляє нові простори імен PID (ідентифікатори процесів). Основні деталі та рішення наведено нижче:

1. **Пояснення проблеми**:
- Ядро Linux дозволяє процесу створювати нові простори імен за допомогою системного виклику `unshare`. Однак процес, який ініціює створення нового простору імен PID (відомий як "процес unshare"), не входить в новий простір імен; це роблять лише його дочірні процеси.
- Виконання `%unshare -p /bin/bash%` запускає `/bin/bash` в тому ж процесі, що і `unshare`. В результаті `/bin/bash` та його дочірні процеси знаходяться в початковому просторі імен PID.
- Перший дочірній процес `/bin/bash` в новому просторі імен стає PID 1. Коли цей процес завершується, він спричиняє очищення простору імен, якщо немає інших процесів, оскільки PID 1 має спеціальну роль усиновлення сиріт. Після цього ядро Linux вимикає виділення PID в цьому просторі імен.

2. **Наслідок**:
- Виходження PID 1 в новому просторі імен призводить до очищення прапорця `PIDNS_HASH_ADDING`. Це призводить до невдалого виділення PID під час створення нового процесу, що призводить до помилки "Неможливо виділити пам'ять".

3. **Рішення**:
- Проблему можна вирішити, використовуючи опцію `-f` з `unshare`. Ця опція змушує `unshare` розгалужувати новий процес після створення нового простору імен PID.
- Виконання `%unshare -fp /bin/bash%` забезпечує, що сама команда `unshare` стає PID 1 в новому просторі імен. `/bin/bash` та його дочірні процеси потім безпечно знаходяться в цьому новому просторі імен, запобігаючи передчасному виходу PID 1 та дозволяючи нормальне виділення PID.

Забезпечуючи виконання `unshare` з прапорцем `-f`, новий простір імен PID правильно підтримується, що дозволяє `/bin/bash` та його підпроцесам працювати без зустрічі помилки виділення пам'яті.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### Перевірте, в якому просторі імені знаходиться ваш процес
```bash
ls -l /proc/self/ns/cgroup
lrwxrwxrwx 1 root root 0 Apr  4 21:19 /proc/self/ns/cgroup -> 'cgroup:[4026531835]'
```
### Знайдіть всі простори імен CGroup

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name cgroup -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name cgroup -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
### Увійдіть всередину простору імен CGroup
```bash
nsenter -C TARGET_PID --pid /bin/bash
```
Також, ви можете **увійти в інший простір процесу лише як root**. І ви **не можете** **увійти** в інший простір без дескриптора, що вказує на нього (наприклад, `/proc/self/ns/cgroup`).

## Посилання
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
