# BloodHound & Інші Інструменти Пошуку В AD

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити вашу **компанію рекламовану на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## Провідник AD

[Провідник AD](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) є частиною набору Sysinternal:

> Продвинутий переглядач та редактор Active Directory (AD). Ви можете використовувати Провідник AD для легкого навігування по базі даних AD, визначення улюблених місць, перегляду властивостей об'єктів та атрибутів без відкриття діалогових вікон, редагування дозволів, перегляду схеми об'єкта та виконання складних пошуків, які можна зберегти та повторно виконати.

### Знімки

Провідник AD може створювати знімки AD, щоб ви могли перевірити їх офлайн.\
Це може бути використано для виявлення уразливостей офлайн або для порівняння різних станів бази даних AD в різний час.

Для створення знімка AD перейдіть до `Файл` --> `Створити знімок` та введіть назву для знімка.

## ADRecon

[**ADRecon**](https://github.com/adrecon/ADRecon) - це інструмент, який витягує та поєднує різні артефакти з середовища AD. Інформацію можна представити у **спеціально форматованому** звіті Microsoft Excel, який містить узагальнені перегляди з метриками для полегшення аналізу та надання цілісної картини поточного стану цільового середовища AD.
```bash
# Run it
.\ADRecon.ps1
```
## BloodHound

З [https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)

> BloodHound - це односторінкова веб-додаток на Javascript, побудований на основі [Linkurious](http://linkurio.us/), скомпільований за допомогою [Electron](http://electron.atom.io/), з базою даних [Neo4j](https://neo4j.com/), яку живить збирач даних на C#.

BloodHound використовує теорію графів для виявлення прихованих та часто непередбачуваних відносин всередині середовища Active Directory або Azure. Атакувальники можуть використовувати BloodHound для легкого виявлення високо складних шляхів атак, які інакше було б неможливо швидко виявити. Захисники можуть використовувати BloodHound для виявлення та усунення тих самих шляхів атаки. Як сині, так і червоні команди можуть використовувати BloodHound для легкого отримання глибшого розуміння привілейованих відносин в середовищі Active Directory або Azure.

Отже, [Bloodhound](https://github.com/BloodHoundAD/BloodHound) - це дивовижний інструмент, який може автоматично перелічити домен, зберегти всю інформацію, знайти можливі шляхи підвищення привілеїв та показати всю інформацію за допомогою графіків.

Booldhound складається з 2 основних частин: **інгесторів** та **візуалізаційного додатку**.

**Інгестори** використовуються для **переліку домену та вилучення всієї інформації** у форматі, який зрозуміє візуалізаційний додаток.

**Візуалізаційний додаток використовує neo4j** для показу того, як усі дані пов'язані між собою та для показу різних способів підвищення привілеїв в домені.

### Встановлення
Після створення BloodHound CE, весь проект був оновлений для зручності використання з Docker. Найпростіший спосіб почати - використовувати його попередньо налаштовану конфігурацію Docker Compose.

1. Встановіть Docker Compose. Це повинно бути включено в [встановлення Docker Desktop](https://www.docker.com/products/docker-desktop/).
2. Виконайте:
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```
3. Знайдіть випадково згенерований пароль у виводі терміналу Docker Compose.
4. У браузері перейдіть за посиланням http://localhost:8080/ui/login. Увійдіть під ім'ям користувача admin та використовуйте випадково згенерований пароль з журналів.

Після цього вам потрібно буде змінити випадково згенерований пароль, і у вас буде готовий новий інтерфейс, з якого ви зможете безпосередньо завантажити інгестори.

### SharpHound

У них є кілька варіантів, але якщо ви хочете запустити SharpHound з ПК, приєднаного до домену, використовуючи ваш поточний користувач та витягнути всю інформацію, ви можете зробити:
```
./SharpHound.exe --CollectionMethods All
Invoke-BloodHound -CollectionMethod All
```
> Ви можете дізнатися більше про **CollectionMethod** та цикл сесії [тут](https://support.bloodhoundenterprise.io/hc/en-us/articles/17481375424795-All-SharpHound-Community-Edition-Flags-Explained)

Якщо ви хочете виконати SharpHound, використовуючи інші облікові дані, ви можете створити сеанс CMD netonly та запустити SharpHound звідти:
```
runas /netonly /user:domain\user "powershell.exe -exec bypass"
```
[**Дізнайтеся більше про Bloodhound на ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)


## Group3r

[**Group3r**](https://github.com/Group3r/Group3r) - це інструмент для пошуку **вразливостей** в **Group Policy**, пов'язаних з Active Directory. \
Вам потрібно **запустити group3r** з хоста всередині домену, використовуючи **будь-якого користувача домену**.
```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```
## PingCastle

[**PingCastle**](https://www.pingcastle.com/documentation/) **оцінює безпеку середовища AD** та надає зручний **звіт** з графіками.

Для запуску, можна виконати виконуваний файл `PingCastle.exe`, і він розпочне **інтерактивну сесію**, презентуючи меню опцій. Опція за замовчуванням для використання - **`healthcheck`**, яка встановить базовий **огляд** **домену**, та знайде **неправильні налаштування** та **вразливості**.&#x20;
