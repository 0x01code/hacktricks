# Змінні середовища Linux

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

**Група з безпеки Try Hard**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Глобальні змінні

Глобальні змінні **будуть** успадковані **дочірніми процесами**.

Ви можете створити глобальну змінну для вашої поточної сесії, виконавши:
```bash
export MYGLOBAL="hello world"
echo $MYGLOBAL #Prints: hello world
```
Ця змінна буде доступна у вашому поточному сеансі та його дочірніх процесах.

Ви можете **видалити** змінну, виконавши:
```bash
unset MYGLOBAL
```
## Локальні змінні

**Локальні змінні** можуть бути доступні тільки для **поточного оболонки/скрипта**.
```bash
LOCAL="my local"
echo $LOCAL
unset LOCAL
```
## Вивести поточні змінні

```bash
printenv
```
```bash
set
env
printenv
cat /proc/$$/environ
cat /proc/`python -c "import os; print(os.getppid())"`/environ
```
## Загальні змінні

З: [https://geek-university.com/linux/common-environment-variables/](https://geek-university.com/linux/common-environment-variables/)

* **DISPLAY** – відображення, яке використовує **X**. Ця змінна зазвичай встановлена ​​на **:0.0**, що означає перше відображення на поточному комп'ютері.
* **EDITOR** – улюблений текстовий редактор користувача.
* **HISTFILESIZE** – максимальна кількість рядків, що містяться в файлі історії.
* **HISTSIZE** – Кількість рядків, доданих до файлу історії, коли користувач завершує свою сесію.
* **HOME** – ваш домашній каталог.
* **HOSTNAME** – ім'я хоста комп'ютера.
* **LANG** – ваша поточна мова.
* **MAIL** – розташування поштового ящика користувача. Зазвичай **/var/spool/mail/USER**.
* **MANPATH** – список каталогів для пошуку сторінок посібника.
* **OSTYPE** – тип операційної системи.
* **PS1** – типовий промпт у bash.
* **PATH** – зберігає шлях до всіх каталогів, які містять виконувані файли, які ви хочете виконати, просто вказавши ім'я файлу, а не відносний або абсолютний шлях.
* **PWD** – поточний робочий каталог.
* **SHELL** – шлях до поточної оболонки команд (наприклад, **/bin/bash**).
* **TERM** – поточний тип терміналу (наприклад, **xterm**).
* **TZ** – ваш часовий пояс.
* **USER** – ваше поточне ім'я користувача.

## Цікаві змінні для хакінгу

### **HISTFILESIZE**

Змініть **значення цієї змінної на 0**, щоб при **завершенні сесії** файл історії (\~/.bash\_history) **був видалений**.
```bash
export HISTFILESIZE=0
```
### **HISTSIZE**

Змініть **значення цієї змінної на 0**, щоб при **завершенні сеансу** будь-яка команда не додавалася до **файлу історії** (\~/.bash\_history).
```bash
export HISTSIZE=0
```
### http\_proxy & https\_proxy

Процеси будуть використовувати **проксі**, вказаний тут, для підключення до Інтернету через **http або https**.
```bash
export http_proxy="http://10.10.10.10:8080"
export https_proxy="http://10.10.10.10:8080"
```
### SSL\_CERT\_FILE & SSL\_CERT\_DIR

Процеси будуть довіряти сертифікатам, вказаним у **цих змінних середовища**.
```bash
export SSL_CERT_FILE=/path/to/ca-bundle.pem
export SSL_CERT_DIR=/path/to/ca-certificates
```
### PS1

Змініть вигляд вашого промпта.

[**Це приклад**](https://gist.github.com/carlospolop/43f7cd50f3deea972439af3222b68808)

Root:

![](<../.gitbook/assets/image (87).png>)

Звичайний користувач:

![](<../.gitbook/assets/image (88).png>)

Один, два та три фонові завдання:

![](<../.gitbook/assets/image (89).png>)

Одне фонове завдання, одне зупинене та остання команда не завершилася правильно:

![](<../.gitbook/assets/image (90).png>)

**Група з високим рівнем безпеки**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
