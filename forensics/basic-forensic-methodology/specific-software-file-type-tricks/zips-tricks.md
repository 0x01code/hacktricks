# Підступи до ZIP-файлів

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

**Командні інструменти** для управління **zip-файлами** є важливими для діагностики, відновлення та взлому zip-файлів. Ось деякі ключові утиліти:

- **`unzip`**: Розкриває причину, чому zip-файл може не розпакуватися.
- **`zipdetails -v`**: Надає детальний аналіз полів формату zip-файлу.
- **`zipinfo`**: Показує вміст zip-файлу без їх видобутку.
- **`zip -F input.zip --out output.zip`** та **`zip -FF input.zip --out output.zip`**: Спробуйте відновити пошкоджені zip-файли.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Інструмент для грубої сили підбору паролів для zip, ефективний для паролів до близько 7 символів.

Специфікація [формату zip-файлу](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) надає вичерпні відомості про структуру та стандарти zip-файлів.

Важливо зауважити, що захищені паролем zip-файли **не шифрують імена файлів або розміри файлів** всередині, уразливість, яку не поділяють файли RAR або 7z, які шифрують цю інформацію. Крім того, zip-файли, зашифровані за допомогою старішого методу ZipCrypto, вразливі до **атаки на текст**, якщо доступний незашифрований примірник стиснутого файлу. Ця атака використовує відомий вміст для взлому пароля zip, уразливість, детально описана в [статті HackThis](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) та подальше пояснення в [цій науковій статті](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). Однак zip-файли, захищені шифруванням **AES-256**, є невразливими до цієї атаки на текст, демонструючи важливість вибору безпечних методів шифрування для конфіденційних даних.

## Посилання
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)
