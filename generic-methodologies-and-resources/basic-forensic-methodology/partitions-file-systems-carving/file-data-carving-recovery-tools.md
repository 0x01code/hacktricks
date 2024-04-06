# Інструменти для відновлення файлів та даних

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Інструменти для відновлення та карвінгу

Більше інструментів за посиланням [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery)

### Autopsy

Найпоширеніший інструмент, який використовується в судовій експертизі для вилучення файлів з образів - це [**Autopsy**](https://www.autopsy.com/download/). Завантажте його, встановіть та використовуйте для пошуку "прихованих" файлів у файлі. Зверніть увагу, що Autopsy призначений для підтримки образів диска та інших видів образів, але не простих файлів.

### Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** - це інструмент для аналізу бінарних файлів для пошуку вбудованого вмісту. Він може бути встановлений через `apt`, а його вихідний код знаходиться на [GitHub](https://github.com/ReFirmLabs/binwalk).

**Корисні команди**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
### Foremost

Ще один поширений інструмент для пошуку прихованих файлів - **foremost**. Ви можете знайти файл конфігурації foremost у `/etc/foremost.conf`. Якщо ви хочете лише знайти певні файли, розкоментуйте їх. Якщо ви нічого не розкоментуєте, foremost буде шукати файли типів, налаштованих за замовчуванням.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
### **Scalpel**

**Scalpel** - це ще один інструмент, який можна використовувати для пошуку та вилучення **файлів, вбудованих у файл**. У цьому випадку вам потрібно розкоментувати з конфігураційного файлу (_/etc/scalpel/scalpel.conf_) типи файлів, які ви хочете вилучити.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
### Bulk Extractor

Цей інструмент входить до складу Kali, але ви можете знайти його тут: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

Цей інструмент може сканувати зображення та **витягувати pcaps** всередині нього, **мережеву інформацію (URL-адреси, домени, IP-адреси, MAC-адреси, пошту)** та більше **файлів**. Вам потрібно лише виконати:
```
bulk_extractor memory.img -o out_folder
```
### PhotoRec

Ви можете знайти його за посиланням [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download)

Він поставляється з версіями GUI та CLI. Ви можете вибрати **типи файлів**, які ви хочете, щоб PhotoRec шукав.

![](<../../../.gitbook/assets/image (524).png>)

### binvis

Перевірте [код](https://code.google.com/archive/p/binvis/) та [веб-сторінку інструменту](https://binvis.io/#/).

#### Особливості BinVis

* Візуальний та активний **переглядач структури**
* Кілька графіків для різних точок фокусу
* Фокусування на частинах вибірки
* **Побачення рядків та ресурсів**, у виконуваних файлах PE або ELF, наприклад
* Отримання **шаблонів** для криптоаналізу файлів
* **Виявлення** алгоритмів упаковки або кодування
* **Визначення** стеганографії за шаблонами
* **Візуальне** порівняння бінарних файлів

BinVis - це чудова **стартова точка для ознайомлення з невідомою ціллю** в сценарії чорної скриньки.

## Специфічні Інструменти Для Відновлення Даних

### FindAES

Шукає ключі AES, шукаючи їх розклади ключів. Здатний знаходити ключі на 128, 192 та 256 біт, такі, як ті, що використовуються TrueCrypt та BitLocker.

Завантажте [тут](https://sourceforge.net/projects/findaes/).

## Доповнюючі інструменти

Ви можете використовувати [**viu** ](https://github.com/atanunq/viu), щоб переглядати зображення з терміналу.\
Ви можете використовувати інструмент командного рядка linux **pdftotext**, щоб перетворити pdf у текст і прочитати його.
