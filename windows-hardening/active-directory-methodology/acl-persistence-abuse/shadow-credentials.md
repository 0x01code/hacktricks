# Тіньові облікові дані

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити вашу **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## Вступ <a href="#3f17" id="3f17"></a>

**Перевірте оригінальний пост для [всієї інформації про цей метод](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab).**

Як **короткий огляд**: якщо ви можете записувати в властивість **msDS-KeyCredentialLink** користувача/комп'ютера, ви можете отримати **NT-хеш цього об'єкта**.

У пості описано метод налаштування **публічно-приватних аутентифікаційних облікових даних** для отримання унікального **Сервісного квитка**, який містить NTLM-хеш цілі. Цей процес включає зашифрований NTLM_SUPPLEMENTAL_CREDENTIAL у Свідоцтві про привілеї (PAC), який можна розшифрувати.

### Вимоги

Для застосування цього методу необхідно виконати певні умови:
- Потрібен принаймні один контролер домену Windows Server 2016.
- На контролері домену повинен бути встановлений цифровий сертифікат аутентифікації сервера.
- Активний каталог повинен бути на рівні функціональності Windows Server 2016.
- Потрібен обліковий запис з делегованими правами на зміну властивості msDS-KeyCredentialLink цільового об'єкта.

## Зловживання

Зловживання довіри до ключа для об'єктів комп'ютера охоплює кроки, які виходять за межі отримання Квитка для надання прав (TGT) та NTLM-хешу. Опції включають:
1. Створення **срібного квитка RC4** для дії як привілейовані користувачі на цільовому хості.
2. Використання TGT з **S4U2Self** для імітації **привілейованих користувачів**, що потребує змін у Сервісному квитку для додавання класу служби до назви служби.

Значним перевагою зловживання довіри до ключа є обмеження на приватний ключ, створений зловмисником, уникнення делегування потенційно вразливим обліковим записам та відсутність необхідності створення облікового запису комп'ютера, що може бути складним для видалення.

## Інструменти

### [**Whisker**](https://github.com/eladshamir/Whisker)

Це базується на DSInternals, надаючи інтерфейс C# для цього атаки. Whisker та його Python аналог, **pyWhisker**, дозволяють маніпулювати властивістю `msDS-KeyCredentialLink`, щоб отримати контроль над обліковими записами Active Directory. Ці інструменти підтримують різні операції, такі як додавання, перелік, видалення та очищення ключових облікових даних з цільового об'єкта.

Функції **Whisker** включають:
- **Додати**: Генерує пару ключів та додає ключові облікові дані.
- **Список**: Показує всі записи ключових облікових даних.
- **Видалити**: Видаляє вказані ключові облікові дані.
- **Очистити**: Стирає всі ключові облікові дані, що може порушити законне використання WHfB.
```shell
Whisker.exe add /target:computername$ /domain:constoso.local /dc:dc1.contoso.local /path:C:\path\to\file.pfx /password:P@ssword1
```
### [pyWhisker](https://github.com/ShutdownRepo/pywhisker)

Він розширює функціональність Whisker на **системи на базі UNIX**, використовуючи Impacket та PyDSInternals для комплексних можливостей експлуатації, включаючи перелік, додавання та видалення KeyCredentials, а також їх імпорт та експорт у форматі JSON.
```shell
python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "list"
```
### [ShadowSpray](https://github.com/Dec0ne/ShadowSpray/)

ShadowSpray має на меті **експлуатувати права GenericWrite/GenericAll, які можуть мати широкі групи користувачів над об'єктами домену**, щоб широко застосовувати ShadowCredentials. Це передбачає вхід в домен, перевірку функціонального рівня домену, перелік об'єктів домену та спробу додати KeyCredentials для отримання TGT та розкриття NT-хешу. Опції очищення та тактики рекурсивної експлуатації підвищують його корисність.


## References

* [https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
* [https://github.com/eladshamir/Whisker](https://github.com/eladshamir/Whisker)
* [https://github.com/Dec0ne/ShadowSpray/](https://github.com/Dec0ne/ShadowSpray/)
* [https://github.com/ShutdownRepo/pywhisker](https://github.com/ShutdownRepo/pywhisker)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпецівій компанії**? Хочете побачити **рекламу вашої компанії на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або групи [**telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
