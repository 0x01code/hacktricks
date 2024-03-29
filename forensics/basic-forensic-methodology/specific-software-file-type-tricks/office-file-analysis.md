# Аналіз офісних файлів

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв**.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш **продвинутих** інструментів спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

Для отримання додаткової інформації перегляньте [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/). Це лише короткий огляд:

Microsoft створив багато форматів офісних документів, з двома основними типами: **формати OLE** (наприклад, RTF, DOC, XLS, PPT) та **формати Office Open XML (OOXML)** (такі як DOCX, XLSX, PPTX). Ці формати можуть містити макроси, що робить їх цілями для розсилання спаму та шкідливих програм. Файли OOXML структуровані як контейнери zip, що дозволяє їхню інспекцію шляхом розпакування, розкриваючи файл та ієрархію папок та вміст файлів XML.

Для дослідження структур файлів OOXML наведено команду для розпакування документа та структуру виводу. Документовано техніки приховування даних у цих файлах, що свідчить про постійні інновації у приховуванні даних у завданнях CTF.

Для аналізу **oletools** та **OfficeDissector** пропонують комплексні набори інструментів для дослідження як OLE, так і OOXML документів. Ці інструменти допомагають в ідентифікації та аналізі вбудованих макросів, які часто служать векторами для доставки шкідливих програм, зазвичай завантажуючи та виконуючи додаткові зловмисні навантаження. Аналіз макросів VBA може бути проведений без Microsoft Office за допомогою Libre Office, що дозволяє відлагодження з точками зупинки та змінними перегляду.

Встановлення та використання **oletools** прості, з наданими командами для встановлення через pip та вилучення макросів з документів. Автоматичне виконання макросів спрацьовує за допомогою функцій, таких як `AutoOpen`, `AutoExec` або `Document_Open`.
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів**, які працюють на найбільш **продвинутих** інструментах спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Вивчіть хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану на HackTricks** або **завантажити HackTricks у PDF-форматі**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи телеграм**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
