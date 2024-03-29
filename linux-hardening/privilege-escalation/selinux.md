<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>


# SELinux у контейнерах

[Вступ та приклад з документації Red Hat](https://www.redhat.com/sysadmin/privileged-flag-container-engines)

[SELinux](https://www.redhat.com/en/blog/latest-container-exploit-runc-can-be-blocked-selinux) - це **система маркування**. Кожен **процес** та кожен **об'єкт файлової системи має **мітку**. Політики SELinux визначають правила щодо того, що **процес з міткою дозволено робити з усіма іншими мітками** на системі.

Двигуни контейнерів запускають **процеси контейнерів з однією обмеженою міткою SELinux**, зазвичай `container_t`, а потім встановлюють контейнер всередині контейнера з міткою `container_file_t`. Правила політики SELinux фактично стверджують, що **процеси `container_t` можуть лише читати/писати/виконувати файли з міткою `container_file_t`**. Якщо процес контейнера виходить за межі контейнера та намагається записати вміст на хості, ядро Linux відмовляє у доступі та дозволяє лише процесу контейнера записувати вміст з міткою `container_file_t`.
```shell
$ podman run -d fedora sleep 100
d4194babf6b877c7100e79de92cd6717166f7302113018686cea650ea40bd7cb
$ podman top -l label
LABEL
system_u:system_r:container_t:s0:c647,c780
```
# Користувачі SELinux

Існують користувачі SELinux, крім звичайних користувачів Linux. Користувачі SELinux є частиною політики SELinux. Кожен користувач Linux відображений на користувача SELinux як частина політики. Це дозволяє користувачам Linux успадковувати обмеження та правила безпеки, а також механізми, що застосовуються до користувачів SELinux.
