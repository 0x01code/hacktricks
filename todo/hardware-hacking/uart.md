<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


# मूल जानकारी

UART एक सीरियल प्रोटोकॉल है, जिसका मतलब है कि यह डेटा को एक समय में एक बिट के रूप में कंपोनेंट्स के बीच स्थानांतरित करता है। इसके विपरीत, पैरलल कम्युनिकेशन प्रोटोकॉल्स एक साथ कई चैनलों के माध्यम से डेटा ट्रांसमिट करते हैं। आम सीरियल प्रोटोकॉल्स में RS-232, I2C, SPI, CAN, Ethernet, HDMI, PCI Express, और USB शामिल हैं।

आमतौर पर, जब UART निष्क्रिय अवस्था में होता है, तो लाइन को उच्च (लॉजिकल 1 मान) पर रखा जाता है। फिर, डेटा स्थानांतरण की शुरुआत को संकेत देने के लिए, ट्रांसमीटर एक स्टार्ट बिट को रिसीवर को भेजता है, जिस दौरान सिग्नल को निम्न (लॉजिकल 0 मान) पर रखा जाता है। इसके बाद, ट्रांसमीटर पांच से आठ डेटा बिट्स भेजता है जिसमें वास्तविक संदेश होता है, इसके बाद एक वैकल्पिक पैरिटी बिट और एक या दो स्टॉप बिट्स (लॉजिकल 1 मान के साथ), कॉन्फ़िगरेशन के आधार पर। पैरिटी बिट, जिसका उपयोग त्रुटि जांच के लिए किया जाता है, व्यवहार में शायद ही देखा जाता है। स्टॉप बिट (या बिट्स) संचार के अंत को सूचित करते हैं।

सबसे आम कॉन्फ़िगरेशन को हम 8N1 कहते हैं: आठ डेटा बिट्स, कोई पैरिटी नहीं, और एक स्टॉप बिट। उदाहरण के लिए, यदि हम C अक्षर, या ASCII में 0x43, को 8N1 UART कॉन्फ़िगरेशन में भेजना चाहते हैं, तो हम निम्नलिखित बिट्स भेजेंगे: 0 (स्टार्ट बिट); 0, 1, 0, 0, 0, 0, 1, 1 (0x43 का बाइनरी में मान), और 0 (स्टॉप बिट)।

![](<../../.gitbook/assets/image (648) (1) (1) (1) (1).png>)

UART के साथ संवाद करने के लिए हार्डवेयर टूल्स:

* USB-to-serial एडाप्टर
* CP2102 या PL2303 चिप्स वाले एडाप्टर्स
* मल्टीपर्पस टूल जैसे: Bus Pirate, Adafruit FT232H, Shikra, या Attify Badge

## UART पोर्ट्स की पहचान करना

UART में 4 पोर्ट्स होते हैं: **TX**(Transmit), **RX**(Receive), **Vcc**(Voltage), और **GND**(Ground)। आपको PCB पर **`TX`** और **`RX`** अक्षर **लिखे हुए** 4 पोर्ट्स मिल सकते हैं। लेकिन यदि कोई संकेत नहीं है, तो आपको खुद से उन्हें ढूंढने की कोशिश करनी पड़ सकती है, इसके लिए आप **मल्टीमीटर** या **लॉजिक एनालाइज़र** का उपयोग कर सकते हैं।

**मल्टीमीटर** के साथ और डिवाइस को बंद करके:

* **GND** पिन की पहचान करने के लिए **Continuity Test** मोड का उपयोग करें, ब्लैक लीड को ग्राउंड में डालें और लाल लीड के साथ टेस्ट करें जब तक कि आपको मल्टीमीटर से आवाज़ न सुनाई दे। PCB पर कई GND पिन्स मिल सकते हैं, इसलिए आपने UART के संबंधित GND पिन को पाया हो सकता है या नहीं।
* **VCC पोर्ट** की पहचान करने के लिए, **DC वोल्टेज मोड** सेट करें और इसे 20 V तक के वोल्टेज पर सेट करें। ब्लैक प्रोब को ग्राउंड पर और रेड प्रोब को पिन पर रखें। डिवाइस को चालू करें। यदि मल्टीमीटर लगातार 3.3 V या 5 V का वोल्टेज मापता है, तो आपने Vcc पिन पाया है। यदि आपको अन्य वोल्टेज मिलते हैं, तो अन्य पोर्ट्स के साथ पुनः प्रयास करें।
* **TX पोर्ट** की पहचान करने के लिए, **DC वोल्टेज मोड** 20 V तक के वोल्टेज पर, ब्लैक प्रोब को ग्राउंड पर और रेड प्रोब को पिन पर रखें, और डिवाइस को चालू करें। यदि आप पाते हैं कि वोल्टेज कुछ सेकंड के लिए उतार-चढ़ाव करता है और फिर Vcc मान पर स्थिर हो जाता है, तो आपने संभवतः TX पोर्ट पाया है। यह इसलिए है क्योंकि चालू होने पर, यह कुछ डीबग डेटा भेजता है।
* **RX पोर्ट** अन्य तीनों के सबसे करीब होगा, इसमें सबसे कम वोल्टेज उतार-चढ़ाव और सभी UART पिन्स का सबसे कम समग्र मान होता है।

आप TX और RX पोर्ट्स को भ्रमित कर सकते हैं और कुछ नहीं होगा, लेकिन यदि आप GND और VCC पोर्ट को भ्रमित करते हैं तो आप सर्किट को जला सकते हैं।

लॉजिक एनालाइज़र के साथ:

## UART बॉड रेट की पहचान करना

सही बॉड रेट की पहचान करने का सबसे आसान तरीका है **TX पिन के आउटपुट को देखना और डेटा को पढ़ने की कोशिश करना**। यदि आपको प्राप्त होने वाला डेटा पढ़ने योग्य नहीं है, तो अगले संभावित बॉड रेट पर स्विच करें जब तक कि डेटा पढ़ने योग्य न हो जाए। इसके लिए आप USB-to-serial एडाप्टर या Bus Pirate जैसे मल्टीपर्पस डिवाइस का उपयोग कर सकते हैं, जिसे [baudrate.py](https://github.com/devttys0/baudrate/) जैसी हेल्पर स्क्रिप्ट के साथ जोड़ा जा सकता है। सबसे आम बॉड रेट्स 9600, 38400, 19200, 57600, और 115200 हैं।

{% hint style="danger" %}
इस प्रोटोकॉल में यह ध्यान रखना महत्वपूर्ण है कि आपको एक डिवाइस के TX को दूसरे के RX से कनेक्ट करना होगा!
{% endhint %}

# Bus Pirate

इस परिदृश्य में हम Arduino के UART संचार को स्निफ करने जा रहे हैं जो प्रोग्राम के सभी प्रिंट्स को Serial Monitor पर भेज रहा है।
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

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>
