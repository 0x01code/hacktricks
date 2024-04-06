# PsExec/Winexec/ScExec

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>

## Як вони працюють

Процес описаний у кроках нижче, показуючи, як бінарні файли служби маніпулюються для досягнення віддаленого виконання на цільовій машині через SMB:

1. **Копіювання бінарного файлу служби на розділ ADMIN$ через SMB** виконується.
2. **Створення служби на віддаленій машині** виконується, вказуючи на бінарний файл.
3. Служба **запускається віддалено**.
4. Після виходу служба **зупиняється, а бінарний файл видаляється**.

### **Процес Ручного Виконання PsExec**

Припускаючи, що є виконавчий навантаження (створений за допомогою msfvenom та затемнений за допомогою Veil для ухилення від виявлення антивірусом), під назвою 'met8888.exe', що представляє віддалене навантаження meterpreter reverse_http, виконуються наступні кроки:

- **Копіювання бінарного файлу**: Виконавчий файл копіюється на розділ ADMIN$ з командного рядка, хоча його можна розмістити де завгодно на файловій системі, щоб залишатися прихованим.

- **Створення служби**: Використовуючи команду Windows `sc`, яка дозволяє запитувати, створювати та видаляти служби Windows віддалено, створюється служба з назвою "meterpreter", щоб вказати на завантажений бінарний файл.

- **Запуск служби**: Останнім кроком є запуск служби, що, ймовірно, призведе до помилки "тайм-ауту" через те, що бінарний файл не є справжнім бінарним файлом служби та не повертає очікуваний код відповіді. Ця помилка не має значення, оскільки основна мета - виконання бінарного файлу.

Спостереження за прослуховувачем Metasploit покаже, що сесія була успішно ініційована.

[Дізнайтеся більше про команду `sc`](https://technet.microsoft.com/en-us/library/bb490995.aspx).


Знайдіть більше деталейних кроків за посиланням: [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

**Ви також можете використовувати бінарний файл Windows Sysinternals PsExec.exe:**

![](<../../.gitbook/assets/image (165).png>)

Ви також можете використовувати [**SharpLateral**](https://github.com/mertdas/SharpLateral):

{% code overflow="wrap" %}
```
SharpLateral.exe redexec HOSTNAME C:\\Users\\Administrator\\Desktop\\malware.exe.exe malware.exe ServiceName
```
{% endcode %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
