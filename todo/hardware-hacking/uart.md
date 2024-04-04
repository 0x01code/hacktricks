<details>

<summary><strong>Erlernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>


# Grundlegende Informationen

UART ist ein serielles Protokoll, das bedeutet, dass es Daten zwischen Komponenten bitweise überträgt. Im Gegensatz dazu übertragen parallele Kommunikationsprotokolle Daten gleichzeitig über mehrere Kanäle. Zu den gängigen seriellen Protokollen gehören RS-232, I2C, SPI, CAN, Ethernet, HDMI, PCI Express und USB.

Im Allgemeinen wird die Leitung während des Leerlaufs von UART auf einem hohen Niveau gehalten (bei einem logischen Wert von 1). Anschließend sendet der Sender zur Signalisierung des Beginns einer Datenübertragung ein Startbit an den Empfänger, währenddessen das Signal auf einem niedrigen Niveau gehalten wird (bei einem logischen Wert von 0). Als Nächstes sendet der Sender fünf bis acht Datenbits mit der eigentlichen Nachricht, gefolgt von einem optionalen Paritätsbit und einem oder zwei Stoppbits (mit einem logischen Wert von 1), abhängig von der Konfiguration. Das Paritätsbit, das zur Fehlerprüfung verwendet wird, wird in der Praxis selten gesehen. Das Stoppbit (oder die Stopbits) signalisieren das Ende der Übertragung.

Die häufigste Konfiguration nennen wir 8N1: acht Datenbits, keine Parität und ein Stoppbit. Wenn wir beispielsweise das Zeichen C oder 0x43 in ASCII in einer 8N1-UART-Konfiguration senden wollten, würden wir die folgenden Bits senden: 0 (das Startbit); 0, 1, 0, 0, 0, 0, 1, 1 (der Wert von 0x43 in binär) und 0 (das Stoppbit).

![](<../../.gitbook/assets/image (648) (1) (1) (1) (1).png>)

Hardware-Tools zur Kommunikation mit UART:

* USB-zu-Seriell-Adapter
* Adapter mit den Chips CP2102 oder PL2303
* Mehrzweckwerkzeug wie: Bus Pirate, den Adafruit FT232H, den Shikra oder das Attify Badge

## Identifizierung von UART-Ports

UART hat 4 Ports: **TX** (Senden), **RX** (Empfangen), **Vcc** (Spannung) und **GND** (Masse). Möglicherweise finden Sie 4 Ports mit den Buchstaben **`TX`** und **`RX`** auf der Leiterplatte. Wenn keine Kennzeichnung vorhanden ist, müssen Sie möglicherweise versuchen, sie mit einem **Multimeter** oder einem **Logikanalysator** selbst zu finden.

Mit einem **Multimeter** und dem ausgeschalteten Gerät:

* Verwenden Sie den **Durchgangstest**-Modus, um den **GND**-Pin zu identifizieren. Platzieren Sie das hintere Messgerätekabel an Masse und testen Sie mit dem roten Kabel, bis Sie einen Ton vom Multimeter hören. Auf der Leiterplatte können mehrere GND-Pins gefunden werden, sodass Sie möglicherweise denjenigen gefunden haben, der zu UART gehört oder auch nicht.
* Um den **VCC-Port** zu identifizieren, stellen Sie den **Gleichspannungsmodus** ein und stellen Sie ihn auf 20 V Spannung ein. Schwarze Sonde an Masse und rote Sonde am Pin. Schalten Sie das Gerät ein. Wenn das Multimeter eine konstante Spannung von entweder 3,3 V oder 5 V misst, haben Sie den Vcc-Pin gefunden. Wenn Sie andere Spannungen erhalten, versuchen Sie es mit anderen Ports erneut.
* Um den **TX-Port** zu identifizieren, **Gleichspannungsmodus** bis zu 20 V Spannung, schwarze Sonde an Masse und rote Sonde am Pin, und schalten Sie das Gerät ein. Wenn Sie feststellen, dass die Spannung einige Sekunden lang schwankt und dann auf den Vcc-Wert stabilisiert, haben Sie höchstwahrscheinlich den TX-Port gefunden. Dies liegt daran, dass beim Einschalten einige Debug-Daten gesendet werden.
* Der **RX-Port** wäre der nächstgelegene zu den anderen 3, er hat die geringste Spannungsschwankung und den niedrigsten Gesamtwert aller UART-Pins.

Sie können die TX- und RX-Ports verwechseln und es würde nichts passieren, aber wenn Sie den GND- und den VCC-Port verwechseln, könnten Sie die Schaltung zerstören.

In einigen Zielgeräten ist der UART-Port vom Hersteller deaktiviert, indem RX oder TX oder sogar beides deaktiviert werden. In diesem Fall kann es hilfreich sein, die Verbindungen auf der Leiterplatte nachzuverfolgen und einen Ausbruchpunkt zu finden. Ein deutlicher Hinweis darauf, dass keine Erkennung von UART und Unterbrechung des Stromkreises vorliegt, besteht darin, die Gerätegarantie zu überprüfen. Wenn das Gerät mit einer Garantie geliefert wurde, hat der Hersteller einige Debug-Schnittstellen (in diesem Fall UART) hinterlassen und daher den UART getrennt und würde ihn wieder anschließen, während er debuggt. Diese Ausbruchspins können durch Löten oder Jumperdrähte verbunden werden.

## Identifizierung der UART-Baudrate

Der einfachste Weg, die richtige Baudrate zu identifizieren, besteht darin, sich die **Ausgabe des TX-Pins anzusehen und zu versuchen, die Daten zu lesen**. Wenn die empfangenen Daten nicht lesbar sind, wechseln Sie zur nächsten möglichen Baudrate, bis die Daten lesbar werden. Sie können hierfür einen USB-zu-Seriell-Adapter oder ein Mehrzweckgerät wie Bus Pirate verwenden, gepaart mit einem Hilfsskript wie [baudrate.py](https://github.com/devttys0/baudrate/). Die häufigsten Baudraten sind 9600, 38400, 19200, 57600 und 115200.

{% hint style="danger" %}
Es ist wichtig zu beachten, dass in diesem Protokoll der TX eines Geräts mit dem RX des anderen verbunden werden muss!
{% endhint %}

# CP210X UART zu TTY-Adapter

Der CP210X-Chip wird in vielen Prototyping-Boards wie NodeMCU (mit esp8266) für die serielle Kommunikation verwendet. Diese Adapter sind relativ kostengünstig und können verwendet werden, um eine Verbindung zur UART-Schnittstelle des Ziels herzustellen. Das Gerät hat 5 Pins: 5V, GND, RXD, TXD, 3.3V. Stellen Sie sicher, dass die Spannung entsprechend dem Ziel angeschlossen ist, um Schäden zu vermeiden. Verbinden Sie schließlich den RXD-Pin des Adapters mit dem TXD des Ziels und den TXD-Pin des Adapters mit dem RXD des Ziels.

Falls der Adapter nicht erkannt wird, stellen Sie sicher, dass die CP210X-Treiber im Hostsystem installiert sind. Sobald der Adapter erkannt und verbunden ist, können Tools wie picocom, minicom oder screen verwendet werden.

Um die an Linux/MacOS-Systeme angeschlossenen Geräte aufzulisten:
```
ls /dev/
```
Für die grundlegende Interaktion mit der UART-Schnittstelle verwenden Sie den folgenden Befehl:
```
picocom /dev/<adapter> --baud <baudrate>
```
Für minicom verwenden Sie den folgenden Befehl, um es zu konfigurieren:
```
minicom -s
```
Konfigurieren Sie die Einstellungen wie Baudrate und Gerätename in der Option `Serielles Anschluss-Setup`.

Nach der Konfiguration verwenden Sie den Befehl `minicom`, um die UART-Konsole zu starten.

# Bus Pirate

In diesem Szenario werden wir die UART-Kommunikation des Arduino abhören, der alle Ausgaben des Programms an den Seriellen Monitor sendet.
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
<details>

<summary><strong>Erlernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories einreichen.

</details>
