# Дамп пам'яті macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Артефакти пам'яті

### Файли свопу

Файли свопу, такі як `/private/var/vm/swapfile0`, служать як **кеші, коли фізична пам'ять заповнена**. Коли в фізичній пам'яті не залишається місця, її дані переносяться до файлу свопу, а потім знову переносяться до фізичної пам'яті за потреби. Можуть бути присутні кілька файлів свопу з назвами, такими як swapfile0, swapfile1 і так далі.

### Образ гібернації

Файл, розташований за адресою `/private/var/vm/sleepimage`, є важливим під час **режиму гібернації**. **Дані з пам'яті зберігаються в цьому файлі під час гібернації OS X**. Після прокидання комп'ютера система отримує дані пам'яті з цього файлу, що дозволяє користувачеві продовжити роботу з того місця, де він закінчив.

Варто зауважити, що на сучасних системах MacOS цей файл зазвичай шифрується з метою безпеки, що ускладнює відновлення.

* Щоб перевірити, чи увімкнено шифрування для sleepimage, можна виконати команду `sysctl vm.swapusage`. Це покаже, чи файл зашифрований.

### Журнали тиску на пам'ять

Ще один важливий файл, пов'язаний з пам'яттю в системах MacOS, - це **журнали тиску на пам'ять**. Ці журнали розташовані в `/var/log` і містять докладну інформацію про використання пам'яті системою та події тиску на пам'ять. Вони можуть бути особливо корисні для діагностики питань, пов'язаних з пам'яттю, або для розуміння того, як система керує пам'яттю з часом.

## Дамп пам'яті за допомогою osxpmem

Для дампу пам'яті на комп'ютері з MacOS можна використовувати [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Примітка**: Наведені інструкції працюватимуть лише для Mac з архітектурою Intel. Цей інструмент зараз заархівований, а останнє випуск було в 2017 році. Завантажений бінарний файл, використовуючи наведені нижче інструкції, призначений для чіпів Intel, оскільки Apple Silicon не існував у 2017 році. Можливо, що можна скомпілювати бінарний файл для архітектури arm64, але вам доведеться спробувати самостійно.
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
Якщо ви знайшли цю помилку: `osxpmem.app/MacPmem.kext не вдалося завантажити - (libkern/kext) помилка аутентифікації (власність файлу/дозволи); перевірте системні/ядро журнали на наявність помилок або спробуйте kextutil(8)` Ви можете виправити це, виконавши:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**Інші помилки** можуть бути виправлені, дозволивши завантаження kext у "Безпека та конфіденційність --> Загальне", просто **дозвольте** це.

Ви також можете використати цей **онелайнер**, щоб завантажити програму, завантажити kext та витягнути пам'ять:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДТРИМКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
