<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।

- **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में।**

</details>


# मूलभूत जानकारी

UART एक सीरियल प्रोटोकॉल है, जिसका अर्थ है कि यह डेटा को एक बिट के साथ ही घटित करता है। इसके विपरीत, पैरलल संचार प्रोटोकॉल एक साथ एकाधिक चैनल के माध्यम से डेटा प्रसारित करते हैं। सामान्य सीरियल प्रोटोकॉल में RS-232, I2C, SPI, CAN, Ethernet, HDMI, PCI Express और USB शामिल हैं।

आमतौर पर, UART आईडल स्थिति में रेखा को ऊचा रखा जाता है (तार्किक 1 मान में)। फिर, डेटा संक्रमण की शुरुआत संकेत करने के लिए, प्रेषक द्वारा प्राप्तकर्ता को एक स्टार्ट बिट भेजा जाता है, जिसके दौरान संकेत को नीचे रखा जाता है (तार्किक 0 मान में)। अगले, प्रेषक द्वारा वास्तविक संदेश समेत पांच से आठ डेटा बिट भेजे जाते हैं, जिसमें वैकल्पिक त्रुटि जांच के लिए एक पैरिटी बिट और एक या दो स्टॉप बिट (तार्किक 1 मान में) शामिल होते हैं, यहां तक कि विन्यास पर निर्भर करता है। त्रुटि जांच के लिए उपयोग किए जाने वाले पैरिटी बिट को अमल में बहुत कम देखा जाता है। स्टॉप बिट (या बिट) प्रेषण के अंत की ओर संकेत करते हैं।

हम 8N1 को सबसे सामान्य विन्यास कहते हैं: आठ डेटा बिट, कोई पैरिटी और एक स्टॉप बिट। उदाहरण के लिए, यदि हमें वर्ण C, यानी ASCII में 0x43, को 8N1 UART विन्यास में भेजना होता है, तो हम निम्नलिखित बिट भेजेंगे: 0 (स्टार्ट बिट); 0, 1, 0, 0, 0, 0, 1, 1 (बाइनरी में 0x43 का मान), और 0 (स्टॉप बिट)।

![](<../../.gitbook/assets/image (648) (1) (1) (1) (1).png>)

UART के साथ संचार करने के लिए हार्डवेयर उपकरण:

* USB-to-serial एडाप्टर
* CP2102 या PL2303 चिप्स के साथ एडाप्टर
* Bus Pirate, Adafruit FT232H, Shikra या Attify Badge जैसा बहुउद्देशीय उपकरण

## UART पोर्ट्स की पहचान करना

UART में 4 पोर्ट होते हैं: **TX**(ट्रांसमिट), **RX**(रिसीव), **Vcc**(वोल्टेज) और **GND**(ग्राउंड)। आप PCB में **`TX`** और **`RX`** अक्षरों के साथ 4 पोर्ट्स ढूंढ़ सकते हैं। लेकिन यदि कोई संकेत नहीं है, तो आपको एक **मल्टीमीटर** या एक **लॉजिक एनालाइजर** का उपयोग करके खुद ही उन्हें ढूंढ़ने की कोशिश करनी पड़ सकती है।

एक **मल्टीमीटर** और उपकरण को बंद करके:

* **GND** पिन की पहचान करने के लिए **सततता परीक्षण** मोड का उपयोग करें, पीछे की लीड को ग्राउंड में रखें और लाल वाले के साथ परीक्षण करें
# बस पायरेट

इस परिदृश्य में हम आर्डुइनो के UART संचार को स्निफ करेंगे, जो सीरियल मॉनिटर में प्रोग्राम के सभी प्रिंट को भेज रहा है।
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
