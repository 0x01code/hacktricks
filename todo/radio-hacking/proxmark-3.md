# Proxmark 3

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпецівій компанії**? Хочете побачити вашу **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Атака на системи RFID з використанням Proxmark3

Перше, що вам потрібно зробити, це мати [**Proxmark3**](https://proxmark.com) та [**встановити програмне забезпечення та його залежності**](https://github.com/Proxmark/proxmark3/wiki/Kali-Linux)[**s**](https://github.com/Proxmark/proxmark3/wiki/Kali-Linux).

### Атака на MIFARE Classic 1KB

У ньому є **16 секторів**, кожен з них має **4 блоки**, а кожен блок містить **16B**. UID знаходиться в секторі 0 блок 0 (і не може бути змінений).\
Для доступу до кожного сектора вам потрібно **2 ключі** (**A** та **B**), які зберігаються в **блоці 3 кожного сектора** (заголовок сектора). У заголовку сектора також зберігаються **біти доступу**, які надають дозвіл на **читання та запис** на **кожний блок** за допомогою 2 ключів.\
2 ключі корисні для надання дозволів на читання, якщо ви знаєте перший, і на запис, якщо ви знаєте другий (наприклад).

Можна виконати кілька атак.
```bash
proxmark3> hf mf #List attacks

proxmark3> hf mf chk *1 ? t ./client/default_keys.dic #Keys bruteforce
proxmark3> hf mf fchk 1 t # Improved keys BF

proxmark3> hf mf rdbl 0 A FFFFFFFFFFFF # Read block 0 with the key
proxmark3> hf mf rdsc 0 A FFFFFFFFFFFF # Read sector 0 with the key

proxmark3> hf mf dump 1 # Dump the information of the card (using creds inside dumpkeys.bin)
proxmark3> hf mf restore # Copy data to a new card
proxmark3> hf mf eload hf-mf-B46F6F79-data # Simulate card using dump
proxmark3> hf mf sim *1 u 8c61b5b4 # Simulate card using memory

proxmark3> hf mf eset 01 000102030405060708090a0b0c0d0e0f # Write those bytes to block 1
proxmark3> hf mf eget 01 # Read block 1
proxmark3> hf mf wrbl 01 B FFFFFFFFFFFF 000102030405060708090a0b0c0d0e0f # Write to the card
```
Процесор Proxmark3 дозволяє виконувати інші дії, такі як **прослуховування** **комунікації між тегом та читачем**, щоб спробувати знайти чутливі дані. На цій картці ви можете лише перехопити комунікацію та розрахувати використаний ключ, оскільки **криптографічні операції, що використовуються, є слабкими**, і знаючи відкритий текст та шифротекст, ви можете розрахувати його (інструмент `mfkey64`).

### Сирий Команди

Системи Інтернету речей іноді використовують **небрендовані або некомерційні теги**. У цьому випадку ви можете використовувати Proxmark3 для відправлення користувацьких **сиріх команд на теги**.
```bash
proxmark3> hf search UID : 80 55 4b 6c ATQA : 00 04
SAK : 08 [2]
TYPE : NXP MIFARE CLASSIC 1k | Plus 2k SL1
proprietary non iso14443-4 card found, RATS not supported
No chinese magic backdoor command detected
Prng detection: WEAK
Valid ISO14443A Tag Found - Quiting Search
```
З цією інформацією ви можете спробувати знайти інформацію про картку та спосіб спілкування з нею. Proxmark3 дозволяє відправляти сирі команди, наприклад: `hf 14a raw -p -b 7 26`

### Сценарії

Програмне забезпечення Proxmark3 поставляється з попередньо завантаженим списком **автоматизованих сценаріїв**, які можна використовувати для виконання простих завдань. Щоб отримати повний список, скористайтеся командою `script list`. Потім використовуйте команду `script run`, за якою слідує назва сценарію:
```
proxmark3> script run mfkeys
```
Ви можете створити скрипт для **фазування читачів тегів**, тому копіюючи дані **дійсної картки**, просто напишіть **Lua-скрипт**, який **випадковим чином** змінює один або кілька випадкових **байтів** і перевіряє, чи **зависає читач** під час будь-якої ітерації.

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпецівій компанії**? Хочете побачити свою **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **і** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
