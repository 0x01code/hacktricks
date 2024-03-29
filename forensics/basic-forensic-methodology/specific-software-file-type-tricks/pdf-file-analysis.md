# Аналіз PDF-файлу

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи телеграм**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв**.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), щоб легко створювати та **автоматизувати робочі процеси** за допомогою найбільш **продвинутих** інструментів спільноти.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

**Для отримання додаткових відомостей перегляньте:** [**https://trailofbits.github.io/ctf/forensics/**](https://trailofbits.github.io/ctf/forensics/)

Формат PDF відомий своєю складністю та потенціалом для приховування даних, що робить його центральною точкою викликів з форензики CTF. Він поєднує елементи простого тексту з бінарними об'єктами, які можуть бути стиснуті або зашифровані, і може містити скрипти на мовах, таких як JavaScript або Flash. Для розуміння структури PDF можна звертатися до вступного матеріалу Дідьє Стівенса [introductory material](https://blog.didierstevens.com/2008/04/09/quickpost-about-the-physical-and-logical-structure-of-pdf-files/), або використовувати інструменти, такі як текстовий редактор або спеціалізований редактор PDF, наприклад Origami.

Для глибокого дослідження або маніпулювання PDF-файлами доступні інструменти, такі як [qpdf](https://github.com/qpdf/qpdf) та [Origami](https://github.com/mobmewireless/origami-pdf). Приховані дані у PDF можуть бути приховані в:

* Невидимих шарах
* Форматі метаданих XMP від Adobe
* Інкрементних поколіннях
* Тексті з таким же кольором, як фон
* Тексті за зображеннями або перекриваючими зображеннями
* Невідображених коментарях

Для настроюваного аналізу PDF можна використовувати бібліотеки Python, такі як [PeepDF](https://github.com/jesparza/peepdf) для створення власних сценаріїв розбору. Крім того, потенціал PDF для зберігання прихованих даних настільки великий, що ресурси, такі як посібник NSA щодо ризиків та протиходій, хоч і більше не розміщені на своїй початковій позиції, все ще пропонують цінні уявлення. [Копія посібника](http://www.itsecure.hu/library/file/Biztons%C3%A1gi%20%C3%BAtmutat%C3%B3k/Alkalmaz%C3%A1sok/Hidden%20Data%20and%20Metadata%20in%20Adobe%20PDF%20Files.pdf) та колекція [прийомів формату PDF](https://github.com/corkami/docs/blob/master/PDF/PDF.md) від Анж Альбертіні можуть забезпечити додаткове читання з цієї теми.

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи телеграм**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв**.

</details>
