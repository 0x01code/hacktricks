# Локальне зберігання у хмарі

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш **продвинутих** інструментів спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## OneDrive

У Windows ви можете знайти папку OneDrive за шляхом `\Users\<username>\AppData\Local\Microsoft\OneDrive`. А всередині `logs\Personal` можна знайти файл `SyncDiagnostics.log`, в якому містяться деякі цікаві дані щодо синхронізованих файлів:

* Розмір у байтах
* Дата створення
* Дата модифікації
* Кількість файлів у хмарі
* Кількість файлів у папці
* **CID**: Унікальний ідентифікатор користувача OneDrive
* Час генерації звіту
* Розмір жорсткого диска ОС

Як тільки ви знайдете CID, рекомендується **шукати файли, що містять цей ідентифікатор**. Можливо, ви зможете знайти файли з назвою: _**\<CID>.ini**_ та _**\<CID>.dat**_, які можуть містити цікаву інформацію, наприклад, назви файлів, які синхронізовані з OneDrive.

## Google Drive

У Windows ви можете знайти основну папку Google Drive за шляхом `\Users\<username>\AppData\Local\Google\Drive\user_default`\
Ця папка містить файл під назвою Sync\_log.log з інформацією, такою як адреса електронної пошти облікового запису, назви файлів, мітки часу, MD5-хеші файлів тощо. Навіть видалені файли з'являються у цьому файлі журналу з відповідними MD5.

Файл **`Cloud_graph\Cloud_graph.db`** є базою даних sqlite, яка містить таблицю **`cloud_graph_entry`**. У цій таблиці ви можете знайти **назву** **синхронізованих** **файлів**, час модифікації, розмір та контрольну суму MD5 файлів.

Дані таблиці бази даних **`Sync_config.db`** містять адресу електронної пошти облікового запису, шляхи до спільних папок та версію Google Drive.

## Dropbox

Dropbox використовує **бази даних SQLite** для управління файлами. У цьому\
Ви можете знайти бази даних у папках:

* `\Users\<username>\AppData\Local\Dropbox`
* `\Users\<username>\AppData\Local\Dropbox\Instance1`
* `\Users\<username>\AppData\Roaming\Dropbox`

І основні бази даних:

* Sigstore.dbx
* Filecache.dbx
* Deleted.dbx
* Config.dbx

Розширення ".dbx" означає, що **бази даних** **зашифровані**. Dropbox використовує **DPAPI** ([https://docs.microsoft.com/en-us/previous-versions/ms995355(v=msdn.10)?redirectedfrom=MSDN](https://docs.microsoft.com/en-us/previous-versions/ms995355\(v=msdn.10\)?redirectedfrom=MSDN))

Щоб краще зрозуміти шифрування, яке використовує Dropbox, ви можете прочитати [https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html](https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html).

Однак основна інформація:

* **Ентропія**: d114a55212655f74bd772e37e64aee9b
* **Сіль**: 0D638C092E8B82FC452883F95F355B8E
* **Алгоритм**: PBKDF2
* **Ітерації**: 1066

Крім цієї інформації, для розшифрування баз даних вам все ще потрібно:

* **Зашифрований ключ DPAPI**: Ви можете знайти його в реєстрі всередині `NTUSER.DAT\Software\Dropbox\ks\client` (експортуйте ці дані як бінарні)
* Гілки **`SYSTEM`** та **`SECURITY`**
* Майстер-ключі DPAPI: Які можна знайти в `\Users\<username>\AppData\Roaming\Microsoft\Protect`
* Ім'я користувача та пароль Windows

Потім ви можете використовувати інструмент [**DataProtectionDecryptor**](https://nirsoft.net/utils/dpapi\_data\_decryptor.html)**:**

![](<../../../.gitbook/assets/image (448).png>)

Якщо все пройде як очікувалося, інструмент покаже **основний ключ**, який вам потрібно **використовувати для відновлення оригінального**. Щоб відновити оригінальний, просто використовуйте цей [рецепт cyber\_chef](https://gchq.github.io/CyberChef/#recipe=Derive\_PBKDF2\_key\(%7B'option':'Hex','string':'98FD6A76ECB87DE8DAB4623123402167'%7D,128,1066,'SHA1',%7B'option':'Hex','string':'0D638C092E8B82FC452883F95F355B8E'%7D\)) вставляючи основний ключ як "пароль" всередині рецепту.

Отриманий шістнадцятковий код - це кінцевий ключ, який використовується для шифрування баз даних, які можна розшифрувати за допомогою:
```bash
sqlite -k <Obtained Key> config.dbx ".backup config.db" #This decompress the config.dbx and creates a clear text backup in config.db
```
**База даних `config.dbx` містить:**

- **Електронна пошта**: Електронна адреса користувача
- **usernamedisplayname**: Ім'я користувача
- **dropbox\_path**: Шлях, де знаходиться папка Dropbox
- **Host\_id: Хеш**, використовується для аутентифікації в хмарі. Це можна скасувати лише з веб-версії.
- **Root\_ns**: Ідентифікатор користувача

**База даних `filecache.db` містить інформацію про всі файли та папки, які синхронізовані з Dropbox. Таблиця `File_journal` містить найбільш корисну інформацію:**

- **Server\_path**: Шлях, де знаходиться файл на сервері (цей шлях передує `host_id` клієнта).
- **local\_sjid**: Версія файлу
- **local\_mtime**: Дата модифікації
- **local\_ctime**: Дата створення

Інші таблиці в цій базі даних містять цікаву інформацію:

- **block\_cache**: Хеш усіх файлів та папок Dropbox
- **block\_ref**: Пов'язаний ідентифікатор хешу таблиці `block_cache` з ідентифікатором файлу в таблиці `file_journal`
- **mount\_table**: Спільні папки Dropbox
- **deleted\_fields**: Видалені файли Dropbox
- **date\_added**

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), щоб легко створювати та **автоматизувати робочі процеси** за допомогою найбільш **продвинутих** інструментів спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
