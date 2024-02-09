<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


# मूल जानकारी

UART एक सीरियल प्रोटोकॉल है, जिसका मतलब है कि यह डेटा को एक बिट के बारे में एक कंपोनेंट से दूसरे कंपोनेंट में स्थानांतरित करता है। उपरोक्त, पैरलल संचार प्रोटोकॉल एक साथ कई चैनल के माध्यम से डेटा स्थानांतरित करते हैं। सामान्य सीरियल प्रोटोकॉल में RS-232, I2C, SPI, CAN, Ethernet, HDMI, PCI Express, और USB शामिल हैं।

सामान्यत: जब UART आईडल स्थिति में होता है, तो रेखा को उच्च रखा जाता है (लॉजिकल 1 मूल्य पर)। फिर, डेटा स्थानांतरण की शुरुआत की संकेत देने के लिए, प्रेषक ने प्राप्तकर्ता को एक स्टार्ट बिट भेजा, जिसके दौरान सिग्नल को निचे रखा गया था (लॉजिकल 0 मूल्य पर)। अगले, प्रेषक ने वास्तविक संदेश वाले पांच से आठ डेटा बिट भेजे, जिसके बाद एक वैकल्पिक पैरिटी बिट और एक या दो स्टॉप बिट (लॉजिकल 1 मूल्य पर) होते हैं, विन्यास के आधार पर। प्रैरिटी बिट, त्रुटि जांच के लिए उपयोग किया जाता है, अभ्यास में इसे बहुत कम देखा जाता है। स्टॉप बिट (या बिट) संदेश का समापन सूचित करते हैं।

हम सबसे सामान्य विन्यास 8N1 कहते हैं: आठ डेटा बिट, कोई पैरिटी, और एक स्टॉप बिट। उदाहरण के लिए, अगर हमें 8N1 UART विन्यास में वर्ण C, या 0x43 ASCII में भेजना होता है, तो हम निम्नलिखित बिट भेजेंगे: 0 (स्टार्ट बिट); 0, 1, 0, 0, 0, 0, 1, 1 (बाइनरी में 0x43 का मान), और 0 (स्टॉप बिट)।

![](<../../.gitbook/assets/image (648) (1) (1) (1) (1).png>)

UART के साथ संचार करने के लिए हार्डवेयर उपकरण:

* USB-to-serial एडाप्टर
* CP2102 या PL2303 चिप्स के साथ एडाप्टर
* Bus Pirate, Adafruit FT232H, Shikra, या Attify Badge जैसा बहुउद्देशीय उपकरण

## UART पोर्ट्स की पहचान

UART में 4 पोर्ट्स होते हैं: **TX**(ट्रांसमिट), **RX**(रिसीव), **Vcc**(वोल्टेज), और **GND**(ग्राउंड)। आपको PCB में **`TX`** और **`RX`** अक्षरों के साथ 4 पोर्ट्स मिल सकते हैं। लेकिन यदि कोई संकेत नहीं है, तो आपको एक **मल्टीमीटर** या एक **लॉजिक एनालाइजर** का उपयोग करके खुद ही उन्हें खोजने की कोशिश करनी पड़ सकती है।

एक **मल्टीमीटर** और उपकरण को बंद करके:

* **GND** पिन की पहचान के लिए **संयोगितता परीक्षण** मोड का उपयोग करें, पीछे का लीड ग्राउंड में रखें और लाल वाले से परीक्षण करें जब तक आप मल्टीमीटर से ध्वनि नहीं सुनते हैं। कई GND पिन्स PCB पर पाए जा सकते हैं, इसलिए आपने UART का वह पिन पाया हो सकता है या नहीं।
* **VCC पोर्ट** की पहचान के लिए **DC वोल्टेज मोड** सेट करें और इसे 20 V वोल्टेज तक सेट करें। ग्राउंड पर काली प्रोब और पिन पर लाल प्रोब रखें। डिवाइस को पावर ऑन करें। यदि मल्टीमीटर 3.3 V या 5 V का निरंतर वोल्टेज मापता है, तो आपने Vcc पिन पाया है। अगर आपको अन्य वोल्टेज मिलते हैं, तो अन्य पोर्ट्स के साथ पुनः प्रयास करें।
* **TX** **पोर्ट** की पहचान के लिए **DC वोल्टेज मोड** अप टू 20 V वोल्टेज, काली प्रोब ग्राउंड पर, और लाल प्रोब पिन पर रखें, और डिवाइस को पावर ऑन करें। यदि आपको वोल्टेज कुछ सेकंडों के लिए फ्लक्चुएट करते हैं और फिर Vcc मान पर स्थिर होता है, तो आपने संभावित रूप से TX पोर्ट पाया है। यह इसलिए है क्योंकि पावर ऑन करते समय, यह कुछ डीबग डेटा भेजता है।
* **RX पोर्ट** सबसे निकट होगा, इसमें सबसे कम वोल्टेज फ्लक्चुएशन और सभी UART पिन्स के सबसे कम मौलिक मान की होगी।

आप TX और RX पोर्ट्स को गलती से गलती कर सकते हैं और कुछ नहीं होगा, लेकिन अगर आप GND और VCC पोर्ट को गलती से गलती करते हैं तो आप सर्किट को जला सकते हैं।

लॉजिक एनालाइजर के साथ:

## UART बौड दर की पहचान

सही बौड दर की पहचान करने का सबसे आसान तरीका **TX पिन की आउटपुट देखना और डेटा पढ़ने का प्रयास करना** है। यदि आपको प्राप्त डेटा पढ़ने योग्य नहीं है, तो डेटा पढ़ने योग्य होने तक अगली संभावित बौड दर पर स्विच करें। आप इसे करने के लिए एक USB-to-serial एडाप्टर या Bus Pirate जैसा बहुउद्देशीय उपकरण उपयोग कर सकते हैं, जिसे [baudrate.py](https://github.com/devttys0/baudrate/) जैसे हेल्पर स्क्रिप्ट के साथ जोड़ा जा सकता है। सबसे सामान्य बौड दरें 9600, 38400, 19200, 57600, और 115200 हैं।

{% hint style="danger" %}
इस प्रोटोकॉल में एक डिवाइस के TX को दूसरे के RX से कनेक्ट करने की आवश्यकता है!
{% endhint %}

# Bus Pirate

इस स्थिति में हम Arduino की UART संचारिकता को स्निफ करने जा रहे हैं, जो सीरियल मॉनिटर में प्रोग्राम के सभी प्रिंट भेज रहा है।
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

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
