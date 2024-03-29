<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>


# Основна інформація

SPI (Serial Peripheral Interface) - це синхронний протокол послідовної комунікації, який використовується вбудованими системами для короткодистанційного зв'язку між ІС (інтегральними схемами). Протокол комунікації SPI використовує архітектуру майстер-раб, яку організовують сигнали годинника та вибору чіпа. Архітектура майстер-раб складається з майстра (зазвичай мікропроцесора), який керує зовнішніми пристроями, такими як EEPROM, датчики, пристрої керування тощо, які вважаються рабами.

До майстра може бути підключено кілька рабів, але раби не можуть спілкуватися між собою. Раби керуються двома контактами: годинником та вибором чіпа. Оскільки SPI - це синхронний протокол комунікації, вхідні та вихідні контакти слідують за сигналами годинника. Вибір чіпа використовується майстром для вибору раба та взаємодії з ним. Коли вибір чіпа високий, пристрій раба не вибраний, тоді як коли він низький, чіп був вибраний, і майстер буде взаємодіяти з рабом.

MOSI (Master Out, Slave In) та MISO (Master In, Slave Out) відповідальні за відправку та отримання даних. Дані надсилаються на пристрій раба через контакт MOSI, коли вибір чіпа утримується на низькому рівні. Вхідні дані містять інструкції, адреси пам'яті або дані відповідно до технічного опису виробника пристрою раба. Після прийняття дійсного вводу контакт MISO відповідає за передачу даних майстру. Вихідні дані надсилаються точно на наступному тактовому циклі після завершення вводу. Контакти MISO передають дані до тих пір, поки дані повністю не передані або майстер не встановить вибір чіпа на високий рівень (у цьому випадку раб припинить передачу, а майстер не буде слухати після цього тактового циклу).

# Вивантаження Flash

## Bus Pirate + flashrom

![](<../../.gitbook/assets/image (201).png>)

Зверніть увагу, що навіть якщо PINOUT Pirate Bus вказує контакти для **MOSI** та **MISO** для підключення до SPI, деякі SPI можуть вказувати контакти як DI та DO. **MOSI -> DI, MISO -> DO**

![](<../../.gitbook/assets/image (648) (1) (1).png>)

У Windows або Linux ви можете використовувати програму [**`flashrom`**](https://www.flashrom.org/Flashrom), щоб вивантажити вміст флеш-пам'яті, запустивши щось на зразок:
```bash
# In this command we are indicating:
# -VV Verbose
# -c <chip> The chip (if you know it better, if not, don'tindicate it and the program might be able to find it)
# -p <programmer> In this case how to contact th chip via the Bus Pirate
# -r <file> Image to save in the filesystem
flashrom -VV -c "W25Q64.V" -p buspirate_spi:dev=COM3 -r flash_content.img
```
<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
