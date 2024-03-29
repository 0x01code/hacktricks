# Зловживання Docker Socket для підвищення привілеїв

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

Існують випадки, коли у вас є **доступ до docker socket** і ви хочете використати його для **підвищення привілеїв**. Деякі дії можуть бути дуже підозрілими, і ви, можливо, захочете їх уникнути, тому тут ви знайдете різні прапорці, які можуть бути корисні для підвищення привілеїв:

### Через монтування

Ви можете **монтувати** різні частини **файлової системи** в контейнері, який працює як root і **отримувати до них доступ**.\
Ви також можете **зловживати монтуванням для підвищення привілеїв** всередині контейнера.

* **`-v /:/host`** -> Підключіть файлову систему хоста до контейнера, щоб ви могли **читати файлову систему хоста.**
* Якщо ви хочете **відчувати себе на хості**, але бути в контейнері, ви можете вимкнути інші захисні механізми, використовуючи прапорці, такі як:
* `--privileged`
* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `-security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* \*\*`--device=/dev/sda1 --cap-add=SYS_ADMIN --security-opt apparmor=unconfined` \*\* -> Це схоже на попередній метод, але тут ми **підключаємо диск пристрою**. Потім всередині контейнера запустіть `mount /dev/sda1 /mnt`, і ви зможете **отримати доступ** до **файлової системи хоста** в `/mnt`
* Виконайте `fdisk -l` на хості, щоб знайти пристрій `</dev/sda1>`, який потрібно підключити
* **`-v /tmp:/host`** -> Якщо з якоїсь причини ви можете **підключити лише деякий каталог** з хоста, і у вас є доступ всередині хоста. Підключіть його і створіть **`/bin/bash`** з **suid** в підключеному каталозі, щоб ви могли **виконати його з хоста та підвищити привілеї до root**.

{% hint style="info" %}
Зверніть увагу, що можливо ви не зможете підключити каталог `/tmp`, але ви можете підключити **інший записний каталог**. Ви можете знайти записні каталоги за допомогою: `find / -writable -type d 2>/dev/null`

**Зверніть увагу, що не всі каталоги на лінукс-машині підтримують біт suid!** Щоб перевірити, які каталоги підтримують біт suid, виконайте `mount | grep -v "nosuid"` Наприклад, зазвичай `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` та `/var/lib/lxcfs` не підтримують біт suid.

Також зверніть увагу, що якщо ви можете **підключити `/etc`** або будь-який інший каталог **із конфігураційними файлами**, ви можете змінювати їх з контейнера Docker як root, щоб **зловживати ними на хості** та підвищити привілеї (можливо, змінюючи `/etc/shadow`)
{% endhint %}

### Виходження з контейнера

* **`--privileged`** -> З цим прапорцем ви [видаляєте всю ізоляцію від контейнера](docker-privileged.md#what-affects). Перевірте техніки для [виходу з привілейованих контейнерів як root](docker-breakout-privilege-escalation/#automatic-enumeration-and-escape).
* **`--cap-add=<CAPABILITY/ALL> [--security-opt apparmor=unconfined] [--security-opt seccomp=unconfined] [-security-opt label:disable]`** -> Для [підвищення привілеїв за допомогою можливостей](../linux-capabilities.md), **надайте цю можливість контейнеру** та вимкніть інші методи захисту, які можуть запобігти роботі експлойта.

### Curl

На цій сторінці ми обговорили способи підвищення привілеїв за допомогою прапорців docker, ви можете знайти **способи зловживання цими методами за допомогою команди curl** на сторінці:

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
