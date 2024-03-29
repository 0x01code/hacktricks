# Внутрішні гаджети Python для читання

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Основна інформація

Різні вразливості, такі як [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) або [**Class Pollution**](class-pollution-pythons-prototype-pollution.md), можуть дозволити вам **читати внутрішні дані Python, але не дозволять виконувати код**. Тому пентестеру потрібно максимально використовувати ці дозволи на читання, щоб **отримати чутливі привілеї та ескалювати вразливість**.

### Flask - Читання секретного ключа

На головній сторінці додатка Flask, ймовірно, буде **глобальний об'єкт `app`, де налаштований цей **секрет**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
У цьому випадку можливий доступ до цього об'єкту, просто використовуючи будь-який гаджет для **доступу до глобальних об'єктів** зі сторінки [**Bypass Python sandboxes**](bypass-python-sandboxes/).

У випадку, коли **вразливість знаходиться в іншому файлі Python**, вам потрібен гаджет для перегляду файлів, щоб дістатися до основного файлу для **доступу до глобального об'єкту `app.secret_key`** для зміни секретного ключа Flask і можливості [**підвищення привілеїв** знавши цей ключ](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Пейлоад, подібний до цього [з цього опису](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Використовуйте цей вразливий код, щоб **змінити `app.secret_key`** (назва в вашому додатку може бути іншою), щоб мати можливість підписувати нові та більш привілейовані куки flask.

### Werkzeug - machine\_id та node uuid

[**Використовуючи цей вразливий код з цього опису**](https://vozec.fr/writeups/tweedle-dum-dee/), ви зможете отримати доступ до **machine\_id** та **uuid** вузла, які є **основними секретами**, необхідними для [**генерації піна Werkzeug**](../../network-services-pentesting/pentesting-web/werkzeug.md), який можна використовувати для доступу до консолі Python у `/console`, якщо **увімкнений режим налагодження:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Зверніть увагу, що ви можете отримати **локальний шлях сервера до `app.py`**, спричиняючи деяку **помилку** на веб-сторінці, яка **надасть вам шлях**.
{% endhint %}

Якщо вразливість знаходиться в іншому файлі Python, перевірте попередній трюк Flask, щоб отримати доступ до об'єктів з основного файлу Python.

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
