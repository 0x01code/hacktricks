# macOS Proces Abuse

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Зловживання процесами macOS

macOS, як і будь-яка інша операційна система, надає різноманітні методи та механізми для **взаємодії процесів, комунікації та обміну даними**. Хоча ці техніки є важливими для ефективної роботи системи, їх також можна зловживати зловмисниками для **виконання зловмисних дій**.

### Впровадження бібліотеки

Впровадження бібліотеки - це техніка, при якій зловмисник **змушує процес завантажити зловісну бібліотеку**. Після впровадження бібліотека працює в контексті цільового процесу, надаючи зловмиснику ті ж дозволи та доступ, що й процес.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Перехоплення функцій

Перехоплення функцій передбачає **перехоплення викликів функцій** або повідомлень у коді програмного забезпечення. Перехоплюючи функції, зловмисник може **змінювати поведінку** процесу, спостерігати за чутливими даними або навіть отримувати контроль над потоком виконання.

{% content-ref url="macos-function-hooking.md" %}
[macos-function-hooking.md](macos-function-hooking.md)
{% endcontent-ref %}

### Міжпроцесне спілкування

Міжпроцесне спілкування (IPC) відноситься до різних методів, за допомогою яких окремі процеси **обмінюються даними**. Хоча IPC є фундаментальним для багатьох законних додатків, його також можна зловживати для обхіду ізоляції процесів, витоку чутливої інформації або виконання несанкціонованих дій.

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Впровадження додатків Electron

Додатки Electron, що виконуються з конкретними змінними середовища, можуть бути вразливі для впровадження процесу:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Впровадження Chromium

Можливо використовувати прапорці `--load-extension` та `--use-fake-ui-for-media-stream` для виконання **атаки людини в браузері**, що дозволяє крадіжку натискань клавіш, трафіку, файлів cookie, впровадження скриптів на сторінках...:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### Брудний NIB

Файли NIB **визначають елементи користувацького інтерфейсу (UI)** та їх взаємодію в додатку. Однак вони можуть **виконувати довільні команди** і **Gatekeeper не зупиняє** вже виконаний додаток від виконання, якщо **файл NIB змінено**. Тому їх можна використовувати для виконання довільних програм довільних команд:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Впровадження додатків Java

Можливо зловживати деякими можливостями Java (наприклад, змінною середовища **`_JAVA_OPTS`**) для виконання **довільного коду/команд** в додатку Java.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### Впровадження додатків .Net

Можливо впроваджувати код в додатки .Net, **зловживаючи функціональністю налагодження .Net** (не захищеною від заходів захисту macOS, таких як жорстке виконання).

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Впровадження Perl

Перевірте різні варіанти, як зробити виконання довільного коду у Perl-скрипті:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Впровадження Ruby

Також можливо зловживати змінними середовища Ruby для виконання довільних скриптів довільним кодом:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Впровадження Python

Якщо змінна середовища **`PYTHONINSPECT`** встановлена, процес Python перейде до Python CLI після завершення. Також можна використовувати **`PYTHONSTARTUP`** для вказівки сценарію Python для виконання на початку інтерактивної сесії.\
Проте слід зауважити, що сценарій **`PYTHONSTARTUP`** не буде виконаний, коли **`PYTHONINSPECT`** створює інтерактивну сесію.

Інші змінні середовища, такі як **`PYTHONPATH`** та **`PYTHONHOME`**, також можуть бути корисними для виконання довільного коду Python.

Зауважте, що виконувані файли, скомпільовані за допомогою **`pyinstaller`**, не будуть використовувати ці змінні середовища, навіть якщо вони виконуються за допомогою вбудованого Python.

{% hint style="danger" %}
Загалом я не зміг знайти способу змусити Python виконати довільний код, зловживаючи змінними середовища.\
Проте більшість людей встановлюють Python за допомогою **Hombrew**, який встановить Python в **записувальне місце** для типового адміністратора. Ви можете захопити його щось на зразок:

```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```

Навіть **root** виконає цей код при запуску python.
{% endhint %}

## Виявлення

### Захист

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) - це відкрите додаток, який може **виявляти та блокувати дії з ін'єкцією процесів**:

* Використання **змінних середовища**: Він буде відстежувати наявність будь-яких з наступних змінних середовища: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** та **`ELECTRON_RUN_AS_NODE`**
* Використання викликів **`task_for_pid`**: Щоб знайти, коли один процес хоче отримати **порт завдання іншого**, що дозволяє впроваджувати код у процес.
* **Параметри програм Electron**: Хтось може використовувати аргумент командного рядка **`--inspect`**, **`--inspect-brk`** та **`--remote-debugging-port`** для запуску програми Electron у режимі налагодження, і таким чином впроваджувати код в неї.
* Використання **символічних посилань** або **жорстких посилань**: Зазвичай найпоширенішим зловживанням є **розміщення посилання з нашими привілеями користувача** та **вказування його на місце з вищими привілеями**. Виявлення дуже просте як для жорстких, так і для символічних посилань. Якщо процес, який створює посилання, має **інший рівень привілеїв** від цільового файлу, ми створює **сповіщення**. Нажаль, у випадку символічних посилань блокування неможливе, оскільки ми не маємо інформації про призначення посилання перед створенням. Це обмеження фреймворку Apple для безпеки кінцевих точок.

### Виклики, здійснені іншими процесами

У [**цьому блозі**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) ви можете дізнатися, як можна використовувати функцію **`task_name_for_pid`** для отримання інформації про інші **процеси, які впроваджують код у процес**, а потім отримати інформацію про цей інший процес.

Зверніть увагу, що для виклику цієї функції вам потрібно бути **тим самим uid**, що і той, що запускає процес, або **root** (і вона повертає інформацію про процес, а не спосіб впровадження коду).

## Посилання

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
