# SmbExec/ScExec

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

## Як це працює

**Smbexec** - це інструмент, який використовується для віддаленого виконання команд на системах Windows, схожий на **Psexec**, але він уникає розміщення будь-яких шкідливих файлів на цільовій системі.

### Ключові моменти про **SMBExec**

- Він працює шляхом створення тимчасової служби (наприклад, "BTOBTO") на цільовій машині для виконання команд через cmd.exe (%COMSPEC%), не залишаючи жодних бінарних файлів.
- Незважаючи на його прихований підхід, він генерує журнали подій для кожної виконаної команди, пропонуючи форму неінтерактивного "шелу".
- Команда для підключення за допомогою **Smbexec** виглядає наступним чином:
```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```
### Виконання команд без бінарних файлів

- **Smbexec** дозволяє пряме виконання команд через шляхи до служб, усуваючи потребу в фізичних бінарних файлах на цільовій системі.
- Цей метод корисний для виконання одноразових команд на цільовій системі Windows. Наприклад, сполучення з модулем `web_delivery` в Metasploit дозволяє виконати обернену пейлоад Meterpreter, орієнтовану на PowerShell.
- Створивши віддалену службу на машині зловмисника з binPath, встановленим для виконання наданої команди через cmd.exe, можливо успішно виконати пейлоад, досягнувши зворотного виклику та виконання пейлоаду з прослуховувачем Metasploit, навіть якщо виникають помилки відповіді служби.

### Приклади команд

Створення та запуск служби можна виконати за допомогою наступних команд:
```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
Для отримання додаткових відомостей перевірте [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)


## Посилання
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
