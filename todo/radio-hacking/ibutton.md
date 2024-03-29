# iButton

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

## Вступ

iButton - це загальна назва для електронного ідентифікаційного ключа, упакованого в **металевий контейнер у формі монети**. Його також називають **Dallas Touch** Memory або контактна пам'ять. Незважаючи на те, що його часто неправильно називають "магнітним" ключем, в ньому **немає нічого магнітного**. Насправді всередині прихований повноцінний **мікросхема**, яка працює за цифровим протоколом.

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

### Що таке iButton? <a href="#what-is-ibutton" id="what-is-ibutton"></a>

Зазвичай, під iButton розуміється фізична форма ключа та читача - кругла монета з двома контактами. Щодо рамки, яка його оточує, існує багато варіацій, від найпоширенішого пластикового тримача з отвором до кілець, кулонів тощо.

<figure><img src="../../.gitbook/assets/image (23) (2).png" alt=""><figcaption></figcaption></figure>

Коли ключ доходить до читача, **контакти доторкаються** і ключ живиться для **передачі** свого ідентифікатора. Іноді ключ **не зчитується** одразу через те, що **контактна PSD інтеркому** більша, ніж повинна бути. Таким чином, зовнішні контури ключа та читача не можуть доторкнутися. У цьому випадку вам доведеться натиснути ключ на одну зі стін читача.

<figure><img src="../../.gitbook/assets/image (21) (2).png" alt=""><figcaption></figcaption></figure>

### **Протокол 1-Wire** <a href="#1-wire-protocol" id="1-wire-protocol"></a>

Ключі Dallas обмінюються даними за допомогою протоколу 1-Wire. З одним контактом для передачі даних (!!) в обидва напрямки, від майстра до раба і навпаки. Протокол 1-Wire працює відповідно до моделі Майстер-Раб. У цій топології Майстер завжди ініціює комунікацію, а Раб слідує його інструкціям.

Коли ключ (Раб) контактує з інтеркомом (Майстром), чіп всередині ключа увімкнеться, живиться інтеркомом, і ключ ініціалізується. Після цього інтерком запитує ідентифікатор ключа. Далі ми розглянемо цей процес більш детально.

Flipper може працювати як в режимі Майстра, так і в режимі Раба. У режимі читання ключа Flipper діє як читач, тобто він працює як Майстер. А в режимі емуляції ключа Flipper претендує на ключ, він перебуває в режимі Раба.

### Ключі Dallas, Cyfral та Metakom

Для отримання інформації про те, як працюють ці ключі, перегляньте сторінку [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

### Атаки

Ключі iButton можуть бути атаковані за допомогою Flipper Zero:

{% content-ref url="flipper-zero/fz-ibutton.md" %}
[fz-ibutton.md](flipper-zero/fz-ibutton.md)
{% endcontent-ref %}

## Посилання

* [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
