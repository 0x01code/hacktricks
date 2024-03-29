# euid, ruid, suid

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Працюєте в **кібербезпецівій компанії**? Хочете побачити вашу **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

### Змінні ідентифікації користувача

- **`ruid`**: **Реальний ідентифікатор користувача** вказує на користувача, який ініціював процес.
- **`euid`**: Відомий як **ефективний ідентифікатор користувача**, він представляє ідентичність користувача, яку використовує система для визначення привілеїв процесу. Загалом, `euid` відображає `ruid`, за винятком випадків, коли виконується бінарний файл SetUID, де `euid` припускає ідентичність власника файлу, надаючи певні операційні дозволи.
- **`suid`**: Цей **збережений ідентифікатор користувача** є ключовим, коли високопривілейований процес (зазвичай запущений як root) повинен тимчасово відмовитися від своїх привілеїв для виконання певних завдань, лише щоб пізніше повернутися до свого початкового підвищеного статусу.

#### Важлива примітка
Процес, який не працює під root, може змінювати свій `euid`, щоб відповідати поточному `ruid`, `euid` або `suid`.

### Розуміння функцій set*uid

- **`setuid`**: На відміну від початкових припущень, `setuid` в основному змінює `euid`, а не `ruid`. Зокрема, для привілейованих процесів він вирівнює `ruid`, `euid` та `suid` з вказаним користувачем, часто root, ефективно закріплюючи ці ідентифікатори через перевизначення `suid`. Детальні відомості можна знайти на сторінці [setuid man](https://man7.org/linux/man-pages/man2/setuid.2.html).
- **`setreuid`** та **`setresuid`**: Ці функції дозволяють докладне налаштування `ruid`, `euid` та `suid`. Однак їх можливості залежать від рівня привілеїв процесу. Для процесів, які не є root, модифікації обмежені поточними значеннями `ruid`, `euid` та `suid`. На відміну від цього, root-процеси або ті, які мають можливість `CAP_SETUID`, можуть призначати довільні значення цим ідентифікаторам. Додаткову інформацію можна отримати на сторінці [setresuid man](https://man7.org/linux/man-pages/man2/setresuid.2.html) та на сторінці [setreuid man](https://man7.org/linux/man-pages/man2/setreuid.2.html).

Ці функціональності призначені не як засіб безпеки, а для полегшення задуманого операційного процесу, наприклад, коли програма приймає ідентичність іншого користувача, змінюючи свій ефективний ідентифікатор користувача.

Зокрема, хоча `setuid` може бути загальним вибором для підвищення привілеїв до root (оскільки він вирівнює всі ідентифікатори до root), важливо розрізняти ці функції для розуміння та маніпулювання поведінкою ідентифікаторів користувача в різних сценаріях.

### Механізми виконання програм в Linux

#### **Системний виклик `execve`**
- **Функціональність**: `execve` запускає програму, визначену першим аргументом. Він приймає два масиви аргументів, `argv` для аргументів та `envp` для середовища.
- **Поведінка**: Він зберігає простір пам'яті викликаючого, але оновлює стек, купу та сегменти даних. Код програми замінюється новою програмою.
- **Збереження ідентифікатора користувача**:
- `ruid`, `euid` та додаткові ідентифікатори груп залишаються незмінними.
- `euid` може мати нюансові зміни, якщо у новій програмі встановлено біт SetUID.
- `suid` оновлюється з `euid` після виконання.
- **Документація**: Докладну інформацію можна знайти на сторінці [`execve` man](https://man7.org/linux/man-pages/man2/execve.2.html).

#### **Функція `system`**
- **Функціональність**: На відміну від `execve`, `system` створює дочірній процес за допомогою `fork` та виконує команду в цьому дочірньому процесі за допомогою `execl`.
- **Виконання команди**: Виконує команду через `sh` за допомогою `execl("/bin/sh", "sh", "-c", command, (char *) NULL);`.
- **Поведінка**: Оскільки `execl` є формою `execve`, він працює аналогічно, але в контексті нового дочірнього процесу.
- **Документація**: Додаткові відомості можна отримати на сторінці [`system` man](https://man7.org/linux/man-pages/man3/system.3.html).

#### **Поведінка `bash` та `sh` з SUID**
- **`bash`**:
- Має опцію `-p`, яка впливає на те, як `euid` та `ruid` обробляються в `bash`.
- Без `-p` `bash` встановлює `euid` на `ruid`, якщо вони спочатку відрізняються.
- З `-p` початковий `euid` зберігається.
- Докладнішу інформацію можна знайти на сторінці [`bash` man](https://linux.die.net/man/1/bash).
- **`sh`**:
- Не має механізму, подібного до `-p` в `bash`.
- Поведінка щодо ідентифікаторів користувача не згадується явно, за винятком опції `-i`, яка підкреслює збереження рівності `euid` та `ruid`.
- Додаткову інформацію можна знайти на сторінці [`sh` man](https://man7.org/linux/man-pages/man1/sh.1p.html).

Ці механізми, відмінні у своїй роботі, пропонують широкий спектр варіантів для виконання та переходу між програмами, з конкретними нюансами у керуванні та збереженні ідентифікаторів користувача.

### Тестування поведінки ідентифікаторів користувача під час виконання

Приклади взяті з https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail, перевірте їх для отримання додаткової інформації

#### Сценарій 1: Використання `setuid` з `system`

**Мета**: Розуміння ефекту `setuid` в поєднанні з `system` та `bash` як `sh`.

**Код на C**:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
system("id");
return 0;
}
```
**Компіляція та дозволи:**
```bash
oxdf@hacky$ gcc a.c -o /mnt/nfsshare/a;
oxdf@hacky$ chmod 4755 /mnt/nfsshare/a
```

```bash
bash-4.2$ $ ./a
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Аналіз:**

* `ruid` та `euid` спочатку рівні 99 (nobody) та 1000 (frank) відповідно.
* `setuid` вирівнює обидва на 1000.
* `system` виконує `/bin/bash -c id` через символічне посилання з sh на bash.
* `bash`, без `-p`, налаштовує `euid` для відповідності `ruid`, що призводить до того, що обидва стають 99 (nobody).

#### Випадок 2: Використання setreuid з system

**C Код**:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setreuid(1000, 1000);
system("id");
return 0;
}
```
**Компіляція та дозволи:**
```bash
oxdf@hacky$ gcc b.c -o /mnt/nfsshare/b; chmod 4755 /mnt/nfsshare/b
```
**Виконання та Результат:**
```bash
bash-4.2$ $ ./b
uid=1000(frank) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Аналіз:**

* `setreuid` встановлює як ruid, так і euid на 1000.
* `system` викликає bash, який зберігає ідентифікатори користувачів через їх рівність, ефективно працюючи як frank.

#### Сценарій 3: Використання setuid з execve
Мета: Дослідження взаємодії між setuid та execve.
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/usr/bin/id", NULL, NULL);
return 0;
}
```
**Виконання та Результат:**
```bash
bash-4.2$ $ ./c
uid=99(nobody) gid=99(nobody) euid=1000(frank) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Аналіз:**

* `ruid` залишається 99, але euid встановлено на 1000, відповідно до ефекту setuid.

**Приклад коду на мові C 2 (Виклик Bash):**
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/bin/bash", NULL, NULL);
return 0;
}
```
**Виконання та Результат:**
```bash
bash-4.2$ $ ./d
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Аналіз:**

* Хоча `euid` встановлено на 1000 за допомогою `setuid`, `bash` скидає euid на `ruid` (99) через відсутність `-p`.

**Приклад коду на мові C 3 (Використання bash -p):**
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
char *const paramList[10] = {"/bin/bash", "-p", NULL};
setuid(1000);
execve(paramList[0], paramList, NULL);
return 0;
}
```
**Виконання та Результат:**
```bash
bash-4.2$ $ ./e
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) euid=100
```
## Посилання
* [https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail](https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете, щоб ваша **компанія рекламувалася на HackTricks**? або ви хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або групи [**telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
