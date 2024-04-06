<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв GitHub**.

</details>


# Перевірка BSSIDs

Коли ви отримуєте захоплення, головний трафік якого є Wifi, використовуючи WireShark, ви можете почати досліджувати всі SSID захоплення за допомогою _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## Brute Force

Один з стовпців на екрані показує, чи **була знайдена будь-яка аутентифікація всередині pcap**. Якщо це так, ви можете спробувати використати Brute force за допомогою `aircrack-ng`:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
Наприклад, він витягне пароль WPA, що захищає PSK (попередньо встановлений ключ), який буде потрібно розшифрувати трафік пізніше.

# Дані в маяках / Бічний канал

Якщо ви підозрюєте, що **дані витікають всередині маяків мережі Wifi**, ви можете перевірити маяки мережі, використовуючи фільтр, подібний до наступного: `wlan містить <ІМЯмережі>`, або `wlan.ssid == "ІМЯмережі"` шукати підозрілі рядки всередині відфільтрованих пакетів.

# Знайдіть невідомі MAC-адреси в мережі Wifi

Наступне посилання буде корисним для пошуку **машин, що відправляють дані всередині мережі Wifi**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

Якщо ви вже знаєте **MAC-адреси, ви можете видалити їх з виводу**, додавши перевірки, подібні до цієї: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Після виявлення **невідомих MAC-адрес, що спілкуються всередині мережі**, ви можете використовувати **фільтри**, подібні до наступного: `wlan.addr==<MAC-адрес> && (ftp || http || ssh || telnet)` для фільтрації його трафіку. Зверніть увагу, що фільтри ftp/http/ssh/telnet корисні, якщо ви розшифрували трафік.

# Розшифрувати трафік

Редагувати --> Налаштування --> Протоколи --> IEEE 802.11--> Редагувати

![](<../../../.gitbook/assets/image (426).png>)
