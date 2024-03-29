<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

**Маніпулювання аудіо та відео файлами** є стандартом у завданнях з **форензики CTF**, використовуючи **стеганографію** та аналіз метаданих для приховання або розкриття секретних повідомлень. Інструменти, такі як **[mediainfo](https://mediaarea.net/en/MediaInfo)** та **`exiftool`**, є необхідними для перевірки метаданих файлу та ідентифікації типів вмісту.

Для завдань з аудіо **[Audacity](http://www.audacityteam.org/)** виділяється як провідний інструмент для перегляду хвиль та аналізу спектрограм, що є важливим для виявлення тексту, закодованого в аудіо. **[Sonic Visualiser](http://www.sonicvisualiser.org/)** високо рекомендується для детального аналізу спектрограм. **Audacity** дозволяє маніпулювати аудіо, такі як уповільнення або реверс треків для виявлення схованих повідомлень. **[Sox](http://sox.sourceforge.net/)**, утиліта командного рядка, відмінно підходить для конвертації та редагування аудіо файлів.

Маніпулювання **найменш значущими бітами (LSB)** є поширеною технікою в аудіо та відео стеганографії, використовуючи фіксовані частини файлів медіа для приховування даних непомітно. **[Multimon-ng](http://tools.kali.org/wireless-attacks/multimon-ng)** корисний для декодування повідомлень, прихованих у вигляді **DTMF сигналів** або **коду Морзе**.

Завдання з відео часто включають контейнерні формати, які об'єднують аудіо та відео потоки. **[FFmpeg](http://ffmpeg.org/)** є основним інструментом для аналізу та маніпулювання цими форматами, здатним демультиплексувати та відтворювати вміст. Для розробників, **[ffmpy](http://ffmpy.readthedocs.io/en/latest/examples.html)** інтегрує можливості FFmpeg у Python для продвинутих сценаріїв взаємодії.

Цей набір інструментів підкреслює необхідну універсальність у завданнях CTF, де учасники повинні використовувати широкий спектр аналізу та маніпуляційних технік для виявлення схованих даних у аудіо та відео файлах.

## References
* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

</details>
