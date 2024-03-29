# Безпека Docker

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш **продвинутих інструментів спільноти** у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## **Основна безпека Docker Engine**

**Движок Docker** використовує **Простори імен** та **Cgroups** ядра Linux для ізоляції контейнерів, надаючи базовий рівень безпеки. Додатковий захист забезпечується за допомогою **відмови в можливостях**, **Seccomp** та **SELinux/AppArmor**, підвищуючи ізоляцію контейнерів. **Плагін автентифікації** може додатково обмежувати дії користувача.

![Безпека Docker](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Безпечний доступ до Docker Engine

До движка Docker можна отримати доступ або локально через Unix-сокет, або віддалено за допомогою HTTP. Для віддаленого доступу важливо використовувати HTTPS та **TLS** для забезпечення конфіденційності, цілісності та аутентифікації.

Движок Docker за замовчуванням прослуховує Unix-сокет за адресою `unix:///var/run/docker.sock`. На системах Ubuntu параметри запуску Docker визначаються в `/etc/default/docker`. Щоб дозволити віддалений доступ до API та клієнта Docker, викладіть демона Docker через HTTP-сокет, додавши наступні налаштування:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
Однак, викладання Docker-демона через HTTP не рекомендується через проблеми з безпекою. Рекомендується застосовувати HTTPS для захисту з'єднань. Існують два основні підходи до захисту з'єднання:

1. Клієнт перевіряє ідентичність сервера.
2. Як клієнт, так і сервер взаємно перевіряють ідентичність один одного.

Для підтвердження ідентичності сервера використовуються сертифікати. Для докладних прикладів обох методів див. [**цей посібник**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/).

### Безпека образів контейнерів

Зображення контейнерів можна зберігати як у приватних, так і у публічних репозиторіях. Docker пропонує кілька варіантів зберігання зображень контейнерів:

* [**Docker Hub**](https://hub.docker.com): Публічний реєстр сервіс від Docker.
* [**Docker Registry**](https://github.com/docker/distribution): Проект з відкритим кодом, який дозволяє користувачам розміщувати власний реєстр.
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Комерційний реєстр Docker, який має аутентифікацію користувачів на основі ролей та інтеграцію з каталоговими службами LDAP.

### Сканування зображень

Контейнери можуть мати **вразливості безпеки** через базове зображення або через програмне забезпечення, встановлене поверх базового зображення. Docker працює над проектом під назвою **Nautilus**, який сканує контейнери на предмет безпеки та перелічує вразливості. Nautilus працює шляхом порівняння кожного шару зображення контейнера з репозиторієм вразливостей для ідентифікації дірок у безпеці.

Для отримання більш докладної [**інформації прочитайте це**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

Команда **`docker scan`** дозволяє сканувати існуючі зображення Docker за допомогою назви або ID зображення. Наприклад, виконайте наступну команду для сканування зображення hello-world:
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Підпис Docker Image

Підпис Docker image забезпечує безпеку та цілісність зображень, що використовуються в контейнерах. Ось стисле пояснення:

* **Docker Content Trust** використовує проект Notary, заснований на The Update Framework (TUF), для управління підписом зображення. Для отримання додаткової інформації див. [Notary](https://github.com/docker/notary) та [TUF](https://theupdateframework.github.io).
* Щоб активувати довіру до вмісту Docker, встановіть `export DOCKER_CONTENT_TRUST=1`. Ця функція вимкнена за замовчуванням у версії Docker 1.10 та пізніше.
* З цією функцією активованою, можна завантажувати лише підписані зображення. Перший пуш зображення вимагає встановлення паролів для кореневого та тегового ключів, причому Docker також підтримує Yubikey для підвищеної безпеки. Додаткові деталі можна знайти [тут](https://blog.docker.com/2015/11/docker-content-trust-yubikey/).
* Спроба витягнути непідписане зображення з активованою довірою до вмісту призводить до помилки "No trust data for latest".
* Для пушів зображень після першого, Docker запитує пароль для ключа репозиторію для підпису зображення.

Для резервного копіювання ваших приватних ключів використовуйте команду:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
При переході до нових хостів Docker необхідно перемістити кореневі та ключі репозиторіїв для забезпечення нормальної роботи.

***

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів**, які працюють на найбільш розвинутих інструментах спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Функції безпеки контейнерів

<details>

<summary>Огляд функцій безпеки контейнерів</summary>

#### Основні функції ізоляції головного процесу

У контейнеризованих середовищах ізоляція проектів та їх процесів є надзвичайно важливою для забезпечення безпеки та управління ресурсами. Ось спрощене пояснення ключових концепцій:

**Простори імен (Namespaces)**

* **Мета**: Забезпечення ізоляції ресурсів, таких як процеси, мережа та файлові системи. Особливо в Docker, простори імен утримують процеси контейнера відокремлено від хоста та інших контейнерів.
* **Використання `unshare`**: Команда `unshare` (або відповідний системний виклик) використовується для створення нових просторів імен, надаючи додатковий рівень ізоляції. Однак, хоча Kubernetes не блокує це за замовчуванням, Docker робить це.
* **Обмеження**: Створення нових просторів імен не дозволяє процесу повертатися до просторів імен за замовчуванням хоста. Для проникнення в простори імен хоста зазвичай потрібен доступ до каталогу хоста `/proc`, використовуючи `nsenter` для входу.

**Групи керування (CGroups)**

* **Функція**: В основному використовується для розподілу ресурсів серед процесів.
* **Аспект безпеки**: Самі CGroups не пропонують безпекової ізоляції, за винятком функції `release_agent`, яка, якщо неправильно налаштована, може бути використана для несанкціонованого доступу.

**Скидання можливостей (Capability Drop)**

* **Важливість**: Це важлива функція безпеки для ізоляції процесів.
* **Функціональність**: Вона обмежує дії, які може виконати кореневий процес, відкидаючи певні можливості. Навіть якщо процес працює з привілеями кореневого користувача, відсутність необхідних можливостей перешкоджає виконанню привілейованих дій, оскільки системні виклики будуть невдалими через недостатні дозволи.

Ось **залишкові можливості** після скидання інших процесів:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
**Seccomp**

Включено за замовчуванням в Docker. Це допомагає **ще більше обмежити syscalls**, які може викликати процес.\
**Профіль за замовчуванням Docker Seccomp** можна знайти за посиланням [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

У Docker є шаблон, який можна активувати: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

Це дозволить обмежити можливості, syscalls, доступ до файлів та папок...

</details>

### Простори імен

**Простори імен** - це функція ядра Linux, яка **розділяє ресурси ядра** таким чином, що один набір **процесів бачить** один набір **ресурсів**, тоді як **інший** набір **процесів** бачить **інший** набір ресурсів. Ця функція працює за рахунок того, що для набору ресурсів та процесів використовується той самий простір імен, але ці простори імен посилаються на різні ресурси. Ресурси можуть існувати в кількох просторах.

Docker використовує наступні простори імен ядра Linux для досягнення ізоляції контейнерів:

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

Для **додаткової інформації про простори імен** перевірте наступну сторінку:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

Функція ядра Linux **cgroups** надає можливість **обмежувати ресурси, такі як центральний процесор, пам'ять, введення/виведення, мережева пропускна здатність** серед набору процесів. Docker дозволяє створювати контейнери з використанням функції cgroup, яка дозволяє контролювати ресурси для конкретного контейнера.\
Нижче наведено контейнер, створений з обмеженням пам'яті користувача на рівні 500м, обмеження пам'яті ядра на рівні 50м, частка центрального процесора на рівні 512, blkioweight на рівні 400. Частка центрального процесора - це відношення, яке контролює використання центрального процесора контейнером. Вона має значення за замовчуванням 1024 та діапазон від 0 до 1024. Якщо у трьох контейнерів однакова частка центрального процесора 1024, кожен контейнер може використовувати до 33% центрального процесора у випадку конфлікту ресурсів центрального процесора. blkio-weight - це відношення, яке контролює введення/виведення контейнера. Вона має значення за замовчуванням 500 та діапазон від 10 до 1000.
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
Для отримання cgroup контейнера ви можете виконати:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
Для отримання додаткової інформації перевірте:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Права

Права дозволяють **досягти більш точного контролю над правами, які можуть бути дозволені** користувачеві root. Docker використовує функцію можливостей ядра Linux для **обмеження операцій, які можуть бути виконані всередині контейнера** незалежно від типу користувача.

Під час запуску контейнера Docker **процес відмовляється від чутливих можливостей, які процес може використовувати для виходу з ізоляції**. Це спроба забезпечити, що процес не зможе виконувати чутливі дії та виходити:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Seccomp в Docker

Ця функція безпеки дозволяє Docker **обмежувати системні виклики**, які можуть бути використані всередині контейнера:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### AppArmor в Docker

**AppArmor** - це покращення ядра для обмеження **контейнерів** до **обмеженого** набору **ресурсів** з **профілями для кожної програми**:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### SELinux в Docker

* **Система міток**: SELinux присвоює унікальну мітку кожному процесу та об'єкту файлової системи.
* **Застосування політики**: Вона застосовує політику безпеки, яка визначає, які дії може виконувати мітка процесу щодо інших міток у системі.
* **Мітки процесів контейнера**: Коли двигуни контейнерів ініціюють процеси контейнерів, їм зазвичай призначається обмежена мітка SELinux, зазвичай `container_t`.
* **Мітки файлів у контейнерах**: Файли всередині контейнера зазвичай мають мітку `container_file_t`.
* **Правила політики**: Політика SELinux в основному забезпечує, що процеси з міткою `container_t` можуть взаємодіяти (читати, записувати, виконувати) лише з файлами, які мають мітку `container_file_t`.

Цей механізм забезпечує, що навіть якщо процес всередині контейнера скомпрометований, він обмежений взаємодією лише з об'єктами, які мають відповідні мітки, значно обмежуючи можливість завдання шкоди від таких компрометацій.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

У Docker авторизаційний плагін відіграє важливу роль у забезпеченні безпеки, вирішуючи, чи дозволяти чи блокувати запити до демона Docker. Це рішення приймається шляхом аналізу двох ключових контекстів:

* **Контекст аутентифікації**: Це включає вичерпну інформацію про користувача, таку як хто вони і як вони аутентифікували себе.
* **Контекст команди**: Це включає всю відповідну інформацію, пов'язану з запитом, який робиться.

Ці контексти допомагають забезпечити, що обробляються лише законні запити від аутентифікованих користувачів, підвищуючи безпеку операцій Docker.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## DoS з контейнера

Якщо ви не обмежуєте ресурси, які може використовувати контейнер, скомпрометований контейнер може запустити DoS на хості, де він працює.

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* Відмова в обслуговуванні за допомогою використання пропускної здатності
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## Цікаві прапорці Docker

### Прапорець --privileged

На наступній сторінці ви можете дізнатися, **що означає прапорець `--privileged`**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

Якщо ви запускаєте контейнер, де зловмисник здобуває доступ як користувач з низькими привілеями. Якщо у вас є **неправильно сконфігурований suid бінарник**, зловмисник може зловживати ним і **підвищувати привілеї всередині** контейнера. Це може дозволити йому втекти з нього.

Запуск контейнера з увімкненою опцією **`no-new-privileges`** **запобігне цьому виду підвищення привілеїв**.
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### Інше
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
Для отримання додаткових параметрів **`--security-opt`** перевірте: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## Інші аспекти безпеки

### Управління секретами: Найкращі практики

Важливо уникати вбудовування секретів безпосередньо в образи Docker або використання змінних середовища, оскільки ці методи викладають вашу чутливу інформацію будь-кому, хто має доступ до контейнера через команди, такі як `docker inspect` або `exec`.

**Томи Docker** є безпечнішою альтернативою, рекомендованою для доступу до чутливої інформації. Їх можна використовувати як тимчасову файлову систему в пам'яті, що зменшує ризики, пов'язані з `docker inspect` та журналюванням. Однак користувачі з правами root та ті, у яких є доступ до `exec` до контейнера, все ще можуть отримати доступ до секретів.

**Секрети Docker** пропонують ще більш безпечний метод обробки чутливої інформації. Для випадків, коли секрети потрібні під час фази побудови образу, **BuildKit** пропонує ефективне рішення з підтримкою секретів на етапі побудови, покращуючи швидкість побудови та надаючи додаткові функції.

Для використання BuildKit його можна активувати трьома способами:

1. Через змінну середовища: `export DOCKER_BUILDKIT=1`
2. Шляхом префіксування команд: `DOCKER_BUILDKIT=1 docker build .`
3. Активація за замовчуванням у конфігурації Docker: `{ "features": { "buildkit": true } }`, за якою слідує перезапуск Docker.

BuildKit дозволяє використовувати секрети на етапі побудови за допомогою параметра `--secret`, забезпечуючи, що ці секрети не включаються в кеш побудови образу або в кінцевий образ, використовуючи команду:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
Для секретів, необхідних у запущеному контейнері, **Docker Compose та Kubernetes** пропонують надійні рішення. Docker Compose використовує ключ `secrets` у визначенні сервісу для вказання секретних файлів, як показано у прикладі `docker-compose.yml`:
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
Ця конфігурація дозволяє використовувати секрети при запуску служб за допомогою Docker Compose.

У середовищах Kubernetes секрети підтримуються на рівні ядра і можуть бути додатково керовані інструментами, такими як [Helm-Secrets](https://github.com/futuresimple/helm-secrets). Контроль доступу на основі ролей (RBAC) Kubernetes підвищує безпеку управління секретами, подібно до Docker Enterprise.

### gVisor

**gVisor** - це ядро додатків, написане на Go, яке реалізує значну частину поверхні системи Linux. Воно включає рантайм [Open Container Initiative (OCI)](https://www.opencontainers.org) під назвою `runsc`, який забезпечує **ізоляційну межу між додатком та ядром хоста**. Рантайм `runsc` інтегрується з Docker та Kubernetes, що спрощує запуск контейнерів у пісочниці.

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** - це відкрита спільнота, яка працює над створенням безпечного рантайму контейнерів з легкими віртуальними машинами, які відчувають себе та працюють як контейнери, але забезпечують **сильнішу ізоляцію робочого навантаження за допомогою технології апаратної віртуалізації** як другого рівня захисту.

{% embed url="https://katacontainers.io/" %}

### Рекомендації по безпеці

* **Не використовуйте прапорець `--privileged` або монтування** [**сокету Docker всередині контейнера**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**.** Сокет Docker дозволяє створювати контейнери, тому це легкий спосіб отримати повний контроль над хостом, наприклад, запускаючи інший контейнер з прапорцем `--privileged`.
* **Не запускайте як root всередині контейнера. Використовуйте** [**іншого користувача**](https://docs.docker.com/develop/develop-images/dockerfile\_best-practices/#user) **та** [**простори імен користувачів**](https://docs.docker.com/engine/security/userns-remap/)**.** Root у контейнері такий самий, як на хості, якщо не перенаправлено за допомогою просторів імен користувачів. Він обмежується, в основному, Linux просторами імен, можливостями та cgroups.
* [**Скиньте всі можливості**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) та включіть лише ті, які потрібні** (`--cap-add=...`). Багато робочих навантажень не потребують жодних можливостей, а їх додавання збільшує обсяг можливого нападу.
* [**Використовуйте параметр безпеки “no-new-privileges”**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) для запобігання процесам отримувати більше привілеїв, наприклад через suid бінарні файли.
* [**Обмежте ресурси, доступні контейнеру**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**.** Обмеження ресурсів може захистити машину від атак на відмову в обслуговуванні.
* **Налаштуйте** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(або SELinux)** профілі для обмеження дій та системних викликів, доступних для контейнера, до мінімуму, необхідного.
* **Використовуйте** [**офіційні образи Docker**](https://docs.docker.com/docker-hub/official\_images/) **та вимагайте підписів** або створюйте власні на їх основі. Не успадковуйте або не використовуйте [заражені](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) образи. Також зберігайте кореневі ключі, пароль на безпековому місці. У Docker є плани керувати ключами за допомогою UCP.
* **Регулярно** **перебудовуйте** свої образи, щоб **застосовувати патчі безпеки до хоста та образів.**
* Розумно **керуйте своїми секретами**, щоб важко було зловмиснику отримати до них доступ.
* Якщо ви **використовуєте демон Docker, використовуйте HTTPS** з аутентифікацією клієнта та сервера.
* У своєму Dockerfile **використовуйте COPY замість ADD**. ADD автоматично розпаковує стиснуті файли та може копіювати файли з URL-адрес. COPY не має цих можливостей. Коли це можливо, уникайте використання ADD, щоб не бути вразливими до атак через віддалені URL-адреси та ZIP-файли.
* Маєте **окремі контейнери для кожного мікросервісу**
* **Не включайте ssh** всередині контейнера, можна використовувати “docker exec” для ssh до контейнера.
* Маєте **менші** образи **контейнерів**

## Втеча з Docker / Підвищення привілеїв

Якщо ви **в контейнері Docker** або у вас є доступ до користувача в **групі docker**, ви можете спробувати **втечу та підвищення привілеїв**:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Обхід аутентифікації Docker Plugin

Якщо у вас є доступ до сокету Docker або у вас є доступ до користувача в **групі docker, але ваші дії обмежуються плагіном аутентифікації Docker**, перевірте, чи можете ви **обійти його:**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Захист Docker

* Інструмент [**docker-bench-security**](https://github.com/docker/docker-bench-security) - це скрипт, який перевіряє десятки загальних найкращих практик щодо розгортання контейнерів Docker у виробництві. Тести автоматизовані та базуються на [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/).\
Вам потрібно запустити інструмент з хоста, на якому працює Docker, або з контейнера з достатніми привілеями. Дізнайтеся, **як запустити його в README:** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## Посилання

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/\_fel1x/status/1151487051986087936](https://twitter.com/\_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux\_namespaces](https://en.wikipedia.org/wiki/Linux\_namespaces)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
* [https://docs.docker.com/engine/extend/plugins\_authorization](https://docs.docker.com/engine/extend/plugins\_authorization)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/](https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/)

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш розвинутих інструментів спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>
<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.
