<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>


# Опис атаки

Уявіть сервер, який **підписує** деякі **дані**, додаючи **секрет** до деяких відомих чітких даних і потім хешуючи ці дані. Якщо ви знаєте:

* **Довжину секрету** (це також можна перебрати з вказаного діапазону довжини)
* **Чіткі дані**
* **Алгоритм (і він вразливий до цієї атаки)**
* **Відоме заповнення**
* Зазвичай використовується типове, тому якщо виконані інші 3 вимоги, це також
* Заповнення змінюється в залежності від довжини секрету+даних, тому потрібна довжина секрету

Тоді для **зловмисника** стає можливим **додавати** **дані** і **генерувати** дійсний **підпис** для **попередніх даних + доданих даних**.

## Як?

Основні вразливі алгоритми генерують хеші, спочатку **хешуючи блок даних**, а потім, **з** **раніше** створеного **хешу** (стану), вони **додають наступний блок даних** і **хешують його**.

Тоді уявіть, що секрет - "секрет" і дані - "дані", MD5 "секретдані" - 6036708eba0d11f6ef52ad44e8b74d5b.\
Якщо зловмисник хоче додати рядок "додати", він може:

* Згенерувати MD5 з 64 "A"
* Змінити стан раніше ініціалізованого хешу на 6036708eba0d11f6ef52ad44e8b74d5b
* Додати рядок "додати"
* Завершити хеш і отриманий хеш буде **дійсним для "секрет" + "дані" + "заповнення" + "додати"**

## **Інструмент**

{% embed url="https://github.com/iagox86/hash_extender" %}

## Посилання

Ви можете знайти цю атаку добре поясненою за посиланням [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
