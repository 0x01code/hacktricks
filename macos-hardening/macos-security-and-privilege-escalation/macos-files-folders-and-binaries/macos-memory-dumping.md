# Дамп пам'яті macOS

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковик, який працює на **темному вебі** та пропонує **безкоштовні** функціональні можливості для перевірки, чи були **компанія або її клієнти скомпрометовані** **вірусами-крадіями**.

Основною метою WhiteIntel є боротьба з захопленням облікових записів та атаками вірусів-вимагачів, що виникають внаслідок вірусів, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їхній двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

***

## Артефакти пам'яті

### Файли обміну

Файли обміну, такі як `/private/var/vm/swapfile0`, служать як **кеші**, коли фізична пам'ять заповнена. Коли в фізичній пам'яті не залишається місця, її дані переносяться до файлу обміну, а потім знову переносяться до фізичної пам'яті за потреби. Можуть бути присутні кілька файлів обміну з назвами, такими як swapfile0, swapfile1 та інші.

### Образ гібернації

Файл, розташований за шляхом `/private/var/vm/sleepimage`, є важливим під час **режиму гібернації**. **Дані з пам'яті зберігаються в цьому файлі під час гібернації OS X**. Після прокидання комп'ютера система отримує дані пам'яті з цього файлу, що дозволяє користувачеві продовжити роботу з того місця, де він закінчив.

Варто зауважити, що на сучасних системах MacOS цей файл зазвичай шифрується з метою безпеки, що ускладнює відновлення.

* Щоб перевірити, чи увімкнено шифрування для sleepimage, можна виконати команду `sysctl vm.swapusage`. Це покаже, чи файл зашифрований.

### Журнали тиску на пам'ять

Ще одним важливим файлом, пов'язаним з пам'яттю в системах MacOS, є **журнал тиску на пам'ять**. Ці журнали розташовані в `/var/log` і містять докладну інформацію про використання пам'яті системою та події тиску на пам'ять. Вони можуть бути особливо корисні для діагностики питань, пов'язаних з пам'яттю, або для розуміння того, як система керує пам'яттю з часом.

## Дамп пам'яті за допомогою osxpmem

Для дампу пам'яті на комп'ютері з MacOS ви можете використовувати [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Примітка**: Наведені інструкції працюватимуть лише для Mac з архітектурою Intel. Цей інструмент зараз заархівований, а останнє оновлення було у 2017 році. Завантажений бінарний файл за допомогою наведених нижче інструкцій призначений для чіпів Intel, оскільки Apple Silicon не існував у 2017 році. Можливо, що можна скомпілювати бінарний файл для архітектури arm64, але вам доведеться спробувати самостійно.
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
**Інші помилки** можуть бути виправлені шляхом **дозволу завантаження kext** в "Безпека та конфіденційність --> Загальне", просто **дозвольте** це.

Ви також можете використати цей **онелайнер** для завантаження програми, завантаження kext та вивантаження пам'яті:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковий двигун, який працює на **темному вебі** і пропонує **безкоштовні** функціональні можливості для перевірки, чи були **компрометовані** компанія або її клієнти **викрадачами шкідливих програм**.

Основна мета WhiteIntel - боротьба з захопленням облікових записів та атаками від шифрувального програмного забезпечення, що виникають від шкідливих програм, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їх двигун **безкоштовно** за посиланням:

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
