<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>


# Інструменти для відновлення даних

## Autopsy

Найпоширеніший інструмент, який використовується в судовій експертизі для вилучення файлів з образів - [**Autopsy**](https://www.autopsy.com/download/). Завантажте його, встановіть та дайте йому обробити файл для пошуку "прихованих" файлів. Зверніть увагу, що Autopsy призначений для підтримки образів дисків та інших видів образів, але не простих файлів.

## Binwalk <a id="binwalk"></a>

**Binwalk** - це інструмент для пошуку вбудованих файлів та даних у бінарних файлах, таких як зображення та аудіофайли.
Він може бути встановлений за допомогою `apt`, однак [вихідний код](https://github.com/ReFirmLabs/binwalk) можна знайти на GitHub.
**Корисні команди**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

Ще один поширений інструмент для пошуку прихованих файлів - **foremost**. Ви можете знайти файл конфігурації foremost у `/etc/foremost.conf`. Якщо ви хочете лише знайти певні файли, розкоментуйте їх. Якщо ви нічого не розкоментуєте, foremost буде шукати файли, налаштовані за замовчуванням.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** - ще один інструмент, який можна використовувати для пошуку та вилучення **файлів, вбудованих у файл**. У цьому випадку вам потрібно розкоментувати з конфігураційного файлу \(_/etc/scalpel/scalpel.conf_\) типи файлів, які ви хочете видобути.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

Цей інструмент входить до складу Kali, але ви можете знайти його тут: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

Цей інструмент може сканувати зображення та **видобувати pcaps** всередині нього, **мережеву інформацію \(URL-адреси, домени, IP-адреси, MAC-адреси, пошту\)** та інші **файли**. Вам потрібно лише виконати:
```text
bulk_extractor memory.img -o out_folder
```
Пройдіть через **всю інформацію**, яку зібрав інструмент \(паролі?\), **проаналізуйте** **пакети** \(читайте [**аналіз Pcaps**](../pcap-inspection/)\), шукайте **дивні домени** \(домени, пов'язані з **шкідливим ПЗ** або **несправжніми**\).

## PhotoRec

Ви можете знайти його за посиланням [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download)

Він постачається з версією GUI та CLI. Ви можете вибрати **типи файлів**, які ви хочете, щоб PhotoRec їх шукав.

![](../../../.gitbook/assets/image%20%28524%29.png)

# Конкретні інструменти для витягування даних

## FindAES

Шукає ключі AES, шукаючи їх розклади ключів. Здатний знаходити ключі на 128, 192 та 256 біт, такі, як ті, що використовуються TrueCrypt та BitLocker.

Завантажте [тут](https://sourceforge.net/projects/findaes/).

# Додаткові інструменти

Ви можете використовувати [**viu** ](https://github.com/atanunq/viu), щоб переглядати зображення з терміналу.
Ви можете використовувати інструмент командного рядка linux **pdftotext**, щоб перетворити pdf у текст і прочитати його.



<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
