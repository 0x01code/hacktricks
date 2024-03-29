<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>


Є кілька блогів в Інтернеті, які **підкреслюють небезпеку залишення принтерів налаштованими з LDAP зі стандартними/слабкими** обліковими даними для входу.\
Це через те, що зловмисник може **обманути принтер, щоб аутентифікуватися проти підробленого LDAP сервера** (зазвичай достатньо `nc -vv -l -p 444`) та захопити облікові дані принтера **у відкритому тексті**.

Крім того, деякі принтери міститимуть **журнали з іменами користувачів** або навіть можуть **завантажити всі імена користувачів** з контролера домену.

Усю цю **чутливу інформацію** та загальний **відсутність безпеки** робить принтери дуже цікавими для зловмисників.

Деякі блоги на цю тему:

* [https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/](https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/)
* [https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856](https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856)

## Налаштування принтера
- **Місце розташування**: Список серверів LDAP знаходиться за адресою: `Мережа > Налаштування LDAP > Налаштування LDAP`.
- **Поведінка**: Інтерфейс дозволяє внесення змін до сервера LDAP без повторного введення облікових даних, спрямовуючи на зручність користувача, але створюючи ризики для безпеки.
- **Експлойт**: Експлойт полягає в перенаправленні адреси сервера LDAP на керовану машину та використанні функції "Тест підключення" для захоплення облікових даних.

## Захоплення облікових даних

**Для докладніших кроків дивіться оригінальне [джерело](https://grimhacker.com/2018/03/09/just-a-printer/).**

### Метод 1: Прослуховувач Netcat
Можливо, досить простий прослуховувач netcat:
```bash
sudo nc -k -v -l -p 386
```
Однак, успішність цього методу варіюється.

### Метод 2: Повний LDAP-сервер з Slapd
Більш надійний підхід передбачає налаштування повного LDAP-сервера, оскільки принтер виконує нульове зв'язування, а потім запит перед спробою прив'язки облікових даних.

1. **Налаштування сервера LDAP**: Цей посібник слідує крокам з [цього джерела](https://www.server-world.info/en/note?os=Fedora_26&p=openldap).
2. **Ключові кроки**:
- Встановити OpenLDAP.
- Налаштувати пароль адміністратора.
- Імпортувати базові схеми.
- Встановити доменне ім'я на LDAP DB.
- Налаштувати LDAP TLS.
3. **Виконання служби LDAP**: Після налаштування службу LDAP можна запустити за допомогою:
```bash
slapd -d 2
```
## Посилання
* [https://grimhacker.com/2018/03/09/just-a-printer/](https://grimhacker.com/2018/03/09/just-a-printer/)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримати HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
