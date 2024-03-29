# Збройовий Дистролес

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди HackTricks AWS)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

## Що таке Дистролес

Контейнер Distroless - це тип контейнера, який **містить лише необхідні залежності для запуску конкретного додатка**, без будь-якого додаткового програмного забезпечення або інструментів, які не є обов'язковими. Ці контейнери призначені бути **легкими** та **безпечними** наскільки це можливо, і вони спрямовані на **мінімізацію поверхні атаки**, видаляючи будь-які зайві компоненти.

Контейнери Distroless часто використовуються в **виробничих середовищах, де безпека та надійність є найважливішими**.

Деякі **приклади** **контейнерів Distroless**:

* Надано **Google**: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* Надано **Chainguard**: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## Збройовий Дистролес

Метою збройованого контейнера Distroless є можливість **виконання довільних бінарних файлів та навантажень навіть за обмежень**, що випливають з **Distroless** (відсутність загальних бінарних файлів у системі) та також захистів, які часто зустрічаються в контейнерах, таких як **тільки для читання** або **без виконання** в `/dev/shm`.

### Через пам'ять

З'явиться приблизно в 2023 році...

### Через існуючі бінарні файли

#### openssl

****[**У цьому пості**](https://www.form3.tech/engineering/content/exploiting-distroless-images) пояснюється, що бінарний файл **`openssl`** часто зустрічається в цих контейнерах, ймовірно, тому що він **необхідний** для програмного забезпечення, яке буде запущено всередині контейнера.

Зловживанням бінарним файлом **`openssl`** можливо **виконати довільні дії**.

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди HackTricks AWS)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
