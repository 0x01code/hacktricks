# Викрадення розкриття чутливої інформації з веб-сторінки

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

Якщо ви коли-небудь знаходите **веб-сторінку, яка показує вам чутливу інформацію на основі вашої сесії**: Можливо, вона відображає файли cookie, або друкує або дані кредитних карт або будь-яку іншу чутливу інформацію, ви можете спробувати її викрасти.\
Ось основні способи, які ви можете спробувати використати для досягнення цієї мети:

* [**Обхід CORS**](pentesting-web/cors-bypass.md): Якщо ви можете обійти заголовки CORS, ви зможете викрасти інформацію, виконуючи запит Ajax для зловмисної сторінки.
* [**XSS**](pentesting-web/xss-cross-site-scripting/): Якщо ви знаходите уразливість XSS на сторінці, ви можете використати її для викрадення інформації.
* [**Danging Markup**](pentesting-web/dangling-markup-html-scriptless-injection/): Якщо ви не можете впровадити теги XSS, ви все ще можете викрасти інформацію, використовуючи інші звичайні теги HTML.
* [**Clickjaking**](pentesting-web/clickjacking.md): Якщо немає захисту від цього виду атаки, ви можете зманити користувача надіслати вам чутливі дані (приклад [тут](https://medium.com/bugbountywriteup/apache-example-servlet-leads-to-61a2720cac20)). 

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
