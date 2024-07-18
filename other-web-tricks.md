# Інші веб-трюки

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

### Заголовок хоста

Декілька разів back-end довіряє **заголовку Host** для виконання деяких дій. Наприклад, він може використовувати його значення як **домен для відправлення скидання пароля**. Таким чином, коли ви отримуєте електронного листа з посиланням на скидання пароля, домен, який використовується, - це той, який ви вказали в заголовку Host. Тоді ви можете запитати скидання пароля інших користувачів та змінити домен на той, яким ви керуєте, щоб вкрасти їх коди скидання пароля. [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
Зверніть увагу, що можливо вам навіть не потрібно чекати, поки користувач натисне на посилання для скидання пароля, щоб отримати токен, оскільки можливо навіть **спам-фільтри або інші проміжні пристрої/боти натиснуть на нього для аналізу**.
{% endhint %}

### Булеві значення сеансу

Іноді, коли ви успішно завершуєте деяку перевірку, back-end **просто додасть булеве значення "True" до атрибуту безпеки вашого сеансу**. Потім інша точка доступу буде знати, чи успішно ви пройшли цю перевірку.\
Однак, якщо ви **пройшли перевірку** і ваш сеанс отримав це значення "True" в атрибуті безпеки, ви можете спробувати **отримати доступ до інших ресурсів**, які **залежать від того самого атрибуту**, але до яких ви **не повинні мати дозволу** на доступ. [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### Функціонал реєстрації

Спробуйте зареєструватися як вже існуючий користувач. Спробуйте також використовувати еквівалентні символи (крапки, багато пробілів та Юнікод).

### Захоплення електронних листів

Зареєструйте електронну пошту, перед підтвердженням змініть пошту, а потім, якщо новий лист підтвердження відправлено на першу зареєстровану пошту, ви можете захопити будь-яку пошту. Або якщо ви можете активувати другу пошту, підтверджуючи першу, ви також можете захопити будь-який обліковий запис.

### Доступ до внутрішньої служби підтримки компаній, які використовують atlassian

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### Метод TRACE

Розробники можуть забути вимкнути різні параметри налагодження в середовищі виробництва. Наприклад, HTTP-метод `TRACE` призначений для діагностичних цілей. Якщо він увімкнений, веб-сервер буде відповідати на запити, які використовують метод `TRACE`, повертаючи у відповіді точний запит, який було отримано. Ця поведінка часто безпечна, але іноді призводить до розкриття інформації, такої як назва внутрішніх заголовків аутентифікації, які можуть додаватися до запитів реверс-проксі.![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)


{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
