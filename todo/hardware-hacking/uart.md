# UART

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) je **dark-web** pretraživač koji nudi **besplatne** funkcionalnosti za proveru da li je neka kompanija ili njeni klijenti **kompromitovani** od strane **stealer malwares**.

Njihov primarni cilj je da se bore protiv preuzimanja naloga i ransomware napada koji proizlaze iz malvera za krađu informacija.

Možete proveriti njihovu veb stranicu i isprobati njihov pretraživač **besplatno** na:

{% embed url="https://whiteintel.io" %}

***

## Osnovne informacije

UART je serijski protokol, što znači da prenosi podatke između komponenti jedan po jedan bit. Nasuprot tome, paralelni komunikacioni protokoli prenose podatke simultano kroz više kanala. Uobičajeni serijski protokoli uključuju RS-232, I2C, SPI, CAN, Ethernet, HDMI, PCI Express i USB.

Generalno, linija se drži visoko (na logičkoj vrednosti 1) dok je UART u stanju mirovanja. Zatim, da signalizuje početak prenosa podataka, predajnik šalje start bit prijemniku, tokom kojeg se signal drži nisko (na logičkoj vrednosti 0). Zatim, predajnik šalje pet do osam bitova podataka koji sadrže stvarnu poruku, praćeno opcionalnim paritet bitom i jednim ili dva stop bita (sa logičkom vrednošću 1), u zavisnosti od konfiguracije. Paritet bit, koji se koristi za proveru grešaka, retko se viđa u praksi. Stop bit (ili bitovi) označavaju kraj prenosa.

Najčešća konfiguracija se naziva 8N1: osam bitova podataka, bez pariteta i jedan stop bit. Na primer, ako bismo želeli da pošaljemo karakter C, ili 0x43 u ASCII, u 8N1 UART konfiguraciji, poslali bismo sledeće bitove: 0 (start bit); 0, 1, 0, 0, 0, 0, 1, 1 (vrednost 0x43 u binarnom obliku), i 0 (stop bit).

![](<../../.gitbook/assets/image (764).png>)

Hardverski alati za komunikaciju sa UART:

* USB-to-serial adapter
* Adapteri sa CP2102 ili PL2303 čipovima
* Višenamenski alat kao što su: Bus Pirate, Adafruit FT232H, Shikra ili Attify Badge

### Identifikacija UART portova

UART ima 4 porta: **TX**(Transmit), **RX**(Receive), **Vcc**(Voltage) i **GND**(Ground). Možda ćete moći da pronađete 4 porta sa **`TX`** i **`RX`** slovima **napisanim** na PCB-u. Ali ako nema oznake, možda ćete morati da ih pronađete sami koristeći **multimetar** ili **logički analizator**.

Sa **multimetrom** i uređajem isključenim:

* Da identifikujete **GND** pin, koristite **Continuity Test** mod, stavite crni vodič u uzemljenje i testirajte sa crvenim dok ne čujete zvuk iz multimetra. Nekoliko GND pinova može se naći na PCB-u, tako da možda niste pronašli onaj koji pripada UART-u.
* Da identifikujete **VCC port**, postavite **DC voltage mode** i podesite ga na 20 V napona. Crni sondu stavite na uzemljenje, a crveni na pin. Uključite uređaj. Ako multimetar meri konstantan napon od 3.3 V ili 5 V, pronašli ste Vcc pin. Ako dobijete druge napone, pokušajte sa drugim portovima.
* Da identifikujete **TX** **port**, postavite **DC voltage mode** na 20 V napona, crni sondu na uzemljenje, a crveni na pin, i uključite uređaj. Ako primetite da napon fluktuira nekoliko sekundi, a zatim se stabilizuje na Vcc vrednosti, verovatno ste pronašli TX port. Ovo je zato što prilikom uključivanja šalje neke debug podatke.
* **RX port** biće najbliži ostalim 3, ima najmanju fluktuaciju napona i najnižu ukupnu vrednost svih UART pinova.

Možete pomešati TX i RX portove i ništa se neće desiti, ali ako pomešate GND i VCC port, mogli biste da oštetite krug.

U nekim ciljnim uređajima, UART port je onemogućen od strane proizvođača onemogućavanjem RX ili TX ili čak oba. U tom slučaju, može biti korisno pratiti veze na štampanoj ploči i pronaći neki breakout point. Jak znak koji potvrđuje da UART nije otkriven i da je krug prekinut je provera garancije uređaja. Ako je uređaj isporučen sa nekom garancijom, proizvođač ostavlja neke debug interfejse (u ovom slučaju, UART) i stoga, mora da je isključio UART i ponovo ga povezao tokom debagovanja. Ovi breakout pinovi mogu se povezati lemljenjem ili jumper žicama.

### Identifikacija UART Baud Rate-a

Najlakši način da identifikujete ispravnu baud rate je da pogledate **izlaz TX pina i pokušate da pročitate podatke**. Ako podaci koje primate nisu čitljivi, prebacite se na sledeću moguću baud rate dok podaci ne postanu čitljivi. Možete koristiti USB-to-serial adapter ili višenamenski uređaj poput Bus Pirate za to, uparen sa pomoćnim skriptom, kao što je [baudrate.py](https://github.com/devttys0/baudrate/). Najčešće baud rate su 9600, 38400, 19200, 57600 i 115200.

{% hint style="danger" %}
Važno je napomenuti da u ovom protokolu treba povezati TX jednog uređaja sa RX drugog!
{% endhint %}

## CP210X UART to TTY Adapter

CP210X čip se koristi u mnogim prototipnim pločama kao što je NodeMCU (sa esp8266) za serijsku komunikaciju. Ovi adapteri su relativno jeftini i mogu se koristiti za povezivanje sa UART interfejsom cilja. Uređaj ima 5 pinova: 5V, GND, RXD, TXD, 3.3V. Uverite se da povežete napon koji podržava cilj kako biste izbegli bilo kakvu štetu. Na kraju povežite RXD pin adaptera sa TXD cilja i TXD pin adaptera sa RXD cilja.

U slučaju da adapter nije otkriven, uverite se da su CP210X drajveri instalirani u host sistemu. Kada se adapter otkrije i poveže, alati poput picocom, minicom ili screen mogu se koristiti.

Da biste naveli uređaje povezane na Linux/MacOS sistemima:
```
ls /dev/
```
Za osnovnu interakciju sa UART interfejsom, koristite sledeću komandu:
```
picocom /dev/<adapter> --baud <baudrate>
```
Za minicom, koristite sledeću komandu za konfiguraciju:
```
minicom -s
```
Konfigurišite postavke kao što su baudrate i ime uređaja u opciji `Serial port setup`.

Nakon konfiguracije, koristite komandu `minicom` da pokrenete UART konzolu.

## UART putem Arduino UNO R3 (uklonljive Atmel 328p čip ploče)

U slučaju da UART Serial to USB adapteri nisu dostupni, Arduino UNO R3 se može koristiti uz brzi hak. Pošto je Arduino UNO R3 obično dostupan svuda, ovo može uštedeti mnogo vremena.

Arduino UNO R3 ima USB to Serial adapter ugrađen na samoj ploči. Da biste dobili UART vezu, jednostavno izvadite Atmel 328p mikrokontroler čip sa ploče. Ovaj hak funkcioniše na varijantama Arduino UNO R3 koje imaju Atmel 328p koji nije lemljen na ploči (SMD verzija se koristi u njemu). Povežite RX pin Arduina (Digital Pin 0) sa TX pinom UART interfejsa i TX pin Arduina (Digital Pin 1) sa RX pinom UART interfejsa.

Na kraju, preporučuje se korišćenje Arduino IDE za dobijanje Serial Console. U `tools` sekciji u meniju, izaberite opciju `Serial Console` i postavite baud rate prema UART interfejsu.

## Bus Pirate

U ovom scenariju ćemo prisluškivati UART komunikaciju Arduina koji šalje sve ispise programa na Serial Monitor.
```bash
# Check the modes
UART>m
1. HiZ
2. 1-WIRE
3. UART
4. I2C
5. SPI
6. 2WIRE
7. 3WIRE
8. KEYB
9. LCD
10. PIC
11. DIO
x. exit(without change)

# Select UART
(1)>3
Set serial port speed: (bps)
1. 300
2. 1200
3. 2400
4. 4800
5. 9600
6. 19200
7. 38400
8. 57600
9. 115200
10. BRG raw value

# Select the speed the communication is occurring on (you BF all this until you find readable things)
# Or you could later use the macro (4) to try to find the speed
(1)>5
Data bits and parity:
1. 8, NONE *default
2. 8, EVEN
3. 8, ODD
4. 9, NONE

# From now on pulse enter for default
(1)>
Stop bits:
1. 1 *default
2. 2
(1)>
Receive polarity:
1. Idle 1 *default
2. Idle 0
(1)>
Select output type:
1. Open drain (H=Hi-Z, L=GND)
2. Normal (H=3.3V, L=GND)

(1)>
Clutch disengaged!!!
To finish setup, start up the power supplies with command 'W'
Ready

# Start
UART>W
POWER SUPPLIES ON
Clutch engaged!!!

# Use macro (2) to read the data of the bus (live monitor)
UART>(2)
Raw UART input
Any key to exit
Escritura inicial completada:
AAA Hi Dreg! AAA
waiting a few secs to repeat....
```
## Dumping Firmware with UART Console

UART Console pruža odličan način za rad sa osnovnim firmverom u runtime okruženju. Ali kada je pristup UART Console samo za čitanje, to može uvesti mnogo ograničenja. U mnogim ugrađenim uređajima, firmver se čuva u EEPROM-ima i izvršava u procesorima koji imaju prolaznu memoriju. Stoga, firmver ostaje samo za čitanje jer je originalni firmver tokom proizvodnje unutar EEPROM-a i svi novi fajlovi bi se izgubili zbog prolazne memorije. Stoga, dumpovanje firmvera je dragocen napor dok radite sa ugrađenim firmverima.

Postoji mnogo načina da se to uradi, a SPI sekcija pokriva metode za ekstrakciju firmvera direktno iz EEPROM-a sa raznim uređajima. Iako, preporučuje se prvo pokušati dumpovanje firmvera sa UART-om, jer dumpovanje firmvera sa fizičkim uređajima i spoljnim interakcijama može biti rizično.

Dumpovanje firmvera iz UART Console zahteva prvo dobijanje pristupa bootloader-ima. Mnogi popularni proizvođači koriste uboot (Universal Bootloader) kao svoj bootloader za učitavanje Linux-a. Stoga, dobijanje pristupa uboot-u je neophodno.

Da biste dobili pristup bootloader-u, povežite UART port sa računarom i koristite bilo koji od alata za Serijsku Konzolu i držite napajanje uređaja isključeno. Kada je postavka spremna, pritisnite taster Enter i držite ga. Na kraju, povežite napajanje uređaja i pustite ga da se pokrene.

Raditi ovo će prekinuti učitavanje uboot-a i pružiti meni. Preporučuje se da razumete uboot komande i koristite meni pomoći da ih navedete. Ovo može biti komanda `help`. Pošto različiti proizvođači koriste različite konfiguracije, neophodno je razumeti svaku od njih posebno.

Obično, komanda za dumpovanje firmvera je:
```
md
```
koji označava "memory dump". Ovo će prikazati memoriju (EEPROM sadržaj) na ekranu. Preporučuje se da se zabeleži izlaz Serial Console pre nego što započnete proceduru za hvatanje memory dump-a.

Na kraju, jednostavno uklonite sve nepotrebne podatke iz log fajla i sačuvajte fajl kao `filename.rom` i koristite binwalk za ekstrakciju sadržaja:
```
binwalk -e <filename.rom>
```
Ovo će navesti moguće sadržaje iz EEPROM-a prema potpisima pronađenim u hex datoteci.

Iako, potrebno je napomenuti da nije uvek slučaj da je uboot otključan čak i ako se koristi. Ako taster Enter ne radi ništa, proverite druge tastere kao što je taster Space, itd. Ako je bootloader zaključan i ne prekida se, ova metoda neće raditi. Da biste proverili da li je uboot bootloader za uređaj, proverite izlaz na UART konzoli tokom pokretanja uređaja. Možda će spomenuti uboot tokom pokretanja.

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) je **dark-web** pretraživač koji nudi **besplatne** funkcionalnosti za proveru da li je neka kompanija ili njeni klijenti **kompromitovani** od strane **stealer malvera**.

Njihov primarni cilj WhiteIntel-a je da se bori protiv preuzimanja naloga i ransomware napada koji proizlaze iz malvera koji krade informacije.

Možete proveriti njihovu veb stranicu i isprobati njihov pretraživač **besplatno** na:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podrška HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
