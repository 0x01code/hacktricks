<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>


## Logstash

Logstash використовується для **збору, перетворення та розподілу журналів** через систему, відому як **конвеєри**. Ці конвеєри складаються з **вхідних**, **фільтруючих** та **вихідних** етапів. Цікавий аспект виникає, коли Logstash працює на скомпрометованій машині.

### Конфігурація конвеєра

Конвеєри налаштовуються в файлі **/etc/logstash/pipelines.yml**, який перераховує місця розташування конфігурацій конвеєрів:
```yaml
# Define your pipelines here. Multiple pipelines can be defined.
# For details on multiple pipelines, refer to the documentation:
# https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: main
path.config: "/etc/logstash/conf.d/*.conf"
- pipeline.id: example
path.config: "/usr/share/logstash/pipeline/1*.conf"
pipeline.workers: 6
```
Цей файл розкриває місцезнаходження файлів **.conf**, що містять конфігурації конвеєрів. При використанні модуля виведення **Elasticsearch**, зазвичай **конвеєри** включають **облікові дані Elasticsearch**, які часто мають широкі привілеї через необхідність Logstash писати дані в Elasticsearch. Джокери в шляхах конфігурації дозволяють Logstash виконувати всі відповідні конвеєри в призначеному каталозі.

### Підвищення привілеїв через записувані конвеєри

Для спроби підвищення привілеїв спочатку ідентифікуйте користувача, під яким працює служба Logstash, зазвичай користувач **logstash**. Переконайтеся, що ви відповідаєте **одному** з цих критеріїв:

- Маєте **право на запис** до файлу конфігурації конвеєра **.conf** **або**
- Файл **/etc/logstash/pipelines.yml** використовує джокер, і ви можете записувати в цільову теку

Додатково повинні бути виконані **одна** з цих умов:

- Можливість перезапуску служби Logstash **або**
- У файлі **/etc/logstash/logstash.yml** встановлено **config.reload.automatic: true**

З урахуванням джокера в конфігурації, створення файлу, який відповідає цьому джокеру, дозволяє виконання команд. Наприклад:
```bash
input {
exec {
command => "whoami"
interval => 120
}
}

output {
file {
path => "/tmp/output.log"
codec => rubydebug
}
}
```
Тут **інтервал** визначає частоту виконання у секундах. У даному прикладі команда **whoami** виконується кожні 120 секунд, а її вивід спрямовується в **/tmp/output.log**.

З **config.reload.automatic: true** в **/etc/logstash/logstash.yml**, Logstash автоматично виявлятиме та застосує нові або змінені конфігурації конвеєрів без необхідності перезапуску. Якщо немає метасимволів, зміни все одно можна вносити до існуючих конфігурацій, але рекомендується бути обережним, щоб уникнути перебоїв.


## Посилання

* [https://insinuator.net/2021/01/pentesting-the-elk-stack/](https://insinuator.net/2021/01/pentesting-the-elk-stack/)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
