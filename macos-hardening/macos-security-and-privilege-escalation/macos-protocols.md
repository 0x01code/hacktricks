# Служби та протоколи мережі macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв GitHub**.

</details>

## Служби віддаленого доступу

Це загальні служби macOS для віддаленого доступу до них.\
Ви можете увімкнути/вимкнути ці служби в `System Settings` --> `Sharing`

* **VNC**, відомий як "Screen Sharing" (tcp:5900)
* **SSH**, відомий як "Remote Login" (tcp:22)
* **Apple Remote Desktop** (ARD), або "Remote Management" (tcp:3283, tcp:5900)
* **AppleEvent**, відомий як "Remote Apple Event" (tcp:3031)

Перевірте, чи яка-небудь з них увімкнена, запустивши:
```bash
rmMgmt=$(netstat -na | grep LISTEN | grep tcp46 | grep "*.3283" | wc -l);
scrShrng=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.5900" | wc -l);
flShrng=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | egrep "\*.88|\*.445|\*.548" | wc -l);
rLgn=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.22" | wc -l);
rAE=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.3031" | wc -l);
bmM=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.4488" | wc -l);
printf "\nThe following services are OFF if '0', or ON otherwise:\nScreen Sharing: %s\nFile Sharing: %s\nRemote Login: %s\nRemote Mgmt: %s\nRemote Apple Events: %s\nBack to My Mac: %s\n\n" "$scrShrng" "$flShrng" "$rLgn" "$rmMgmt" "$rAE" "$bmM";
```
### Пентест ARD

Apple Remote Desktop (ARD) - це розширена версія [Virtual Network Computing (VNC)](https://en.wikipedia.org/wiki/Virtual_Network_Computing), спеціально адаптована для macOS, що пропонує додаткові функції. Значна вразливість ARD полягає в методі аутентифікації для пароля екрану керування, який використовує лише перші 8 символів пароля, що робить його вразливим до [атак перебору паролів](https://thudinh.blogspot.com/2017/09/brute-forcing-passwords-with-thc-hydra.html) за допомогою інструментів, таких як Hydra або [GoRedShell](https://github.com/ahhh/GoRedShell/), оскільки в ньому немає обмежень за замовчуванням.

Вразливі екземпляри можна ідентифікувати за допомогою скрипта `vnc-info` у **nmap**. Служби, які підтримують `VNC Authentication (2)`, особливо схильні до атак перебору через обрізання пароля до 8 символів.

Для активації ARD для різних адміністративних завдань, таких як підвищення привілеїв, доступ до GUI або моніторинг користувачів, використовуйте наступну команду:
```bash
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -activate -configure -allowAccessFor -allUsers -privs -all -clientopts -setmenuextra -menuextra yes
```
ARD надає різні рівні керування, включаючи спостереження, спільне керування та повне керування, з сеансами, які залишаються навіть після зміни пароля користувача. Це дозволяє надсилати команди Unix безпосередньо, виконуючи їх як root для адміністративних користувачів. Завдяки плануванню завдань та віддаленому пошуку Remote Spotlight, ARD дозволяє виконувати віддалені, маловпливові пошуки чутливих файлів на кількох машинах.

## Протокол Bonjour

Bonjour, технологія, розроблена Apple, дозволяє **пристроям в одній мережі виявляти послуги, які пропонуються один одному**. Відомий також як Rendezvous, **Zero Configuration** або Zeroconf, він дозволяє пристрою приєднатися до мережі TCP/IP, **автоматично вибрати IP-адресу** та транслювати свої послуги іншим пристроям у мережі.

Zero Configuration Networking, наданий Bonjour, забезпечує можливість пристроям:
* **Автоматично отримувати IP-адресу** навіть у відсутності DHCP-сервера.
* Виконувати **переклад імен на адреси** без необхідності DNS-сервера.
* **Виявляти послуги**, доступні в мережі.

Пристрої, які використовують Bonjour, призначають собі **IP-адресу з діапазону 169.254/16** та перевіряють її унікальність у мережі. Mac зберігає запис у таблиці маршрутизації для цієї підмережі, який можна перевірити за допомогою `netstat -rn | grep 169`.

Для DNS Bonjour використовує **протокол Multicast DNS (mDNS)**. mDNS працює через **порт 5353/UDP**, використовуючи **стандартні запити DNS**, але спрямовуючи їх на **мультікаст-адресу 224.0.0.251**. Цей підхід забезпечує можливість всім пристроям, які слухають мережу, отримувати та відповідати на запити, сприяючи оновленню їх записів.

При приєднанні до мережі кожен пристрій самостійно обирає ім'я, яке зазвичай закінчується на **.local**, яке може бути похідним від імені хоста або випадково згенерованим.

Виявлення послуг у мережі сприяє **DNS Service Discovery (DNS-SD)**. Використовуючи формат записів DNS SRV, DNS-SD використовує **записи DNS PTR** для можливості переліку кількох послуг. Клієнт, який шукає певну послугу, буде запитувати запис PTR для `<Service>.<Domain>`, отримуючи у відповідь список записів PTR у форматі `<Instance>.<Service>.<Domain>`, якщо послуга доступна з декількох хостів.

Утиліта `dns-sd` може бути використана для **виявлення та реклами послуг у мережі**. Ось деякі приклади її використання:

### Пошук SSH-послуг

Для пошуку SSH-послуг у мережі використовується наступна команда:
```bash
dns-sd -B _ssh._tcp
```
Ця команда ініціює перегляд служб _ssh._tcp та виводить деталі, такі як мітка часу, прапорці, інтерфейс, домен, тип служби та назва екземпляру.

### Рекламування служби HTTP

Для реклами служби HTTP можна використовувати:
```bash
dns-sd -R "Index" _http._tcp . 80 path=/index.html
```
Ця команда реєструє службу HTTP з назвою "Index" на порту 80 з шляхом `/index.html`.

Для пошуку служб HTTP в мережі:
```bash
dns-sd -B _http._tcp
```
Коли служба запускається, вона оголошує свою доступність всім пристроям у підмережі, відправляючи мультикастингові повідомлення про свою присутність. Пристрої, які цікавляться цими службами, не повинні відправляти запити, а просто слухати ці оголошення.

Для більш зручного інтерфейсу додаток **Discovery - DNS-SD Browser**, доступний в Apple App Store, може візуалізувати послуги, які пропонуються в вашій локальній мережі.

Альтернативно, можна написати власні скрипти для перегляду та виявлення служб за допомогою бібліотеки `python-zeroconf`. Скрипт [**python-zeroconf**](https://github.com/jstasiak/python-zeroconf) демонструє створення браузера служб для служб `_http._tcp.local.`, який виводить додані або видалені служби:
```python
from zeroconf import ServiceBrowser, Zeroconf

class MyListener:

def remove_service(self, zeroconf, type, name):
print("Service %s removed" % (name,))

def add_service(self, zeroconf, type, name):
info = zeroconf.get_service_info(type, name)
print("Service %s added, service info: %s" % (name, info))

zeroconf = Zeroconf()
listener = MyListener()
browser = ServiceBrowser(zeroconf, "_http._tcp.local.", listener)
try:
input("Press enter to exit...\n\n")
finally:
zeroconf.close()
```
### Вимкнення Bonjour
Якщо є питання щодо безпеки або інших причин для вимкнення Bonjour, його можна вимкнути за допомогою наступної команди:
```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist
```
## Посилання

* [**Посібник хакера Mac**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html**](https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
