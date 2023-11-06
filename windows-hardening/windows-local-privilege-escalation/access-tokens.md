# पहुंच टोकन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें**.

</details>

## पहुंच टोकन

प्रत्येक **उपयोगकर्ता जिसने सिस्टम पर लॉग इन किया है**, उसके पास उस लॉगइन सत्र के लिए सुरक्षा सूचना के साथ एक पहुंच टोकन होता है। सिस्टम उपयोगकर्ता लॉग इन करते समय एक पहुंच टोकन बनाता है। **प्रत्येक प्रक्रिया** जो उपयोगकर्ता के नाम पर **चलाई जाती है**, उसके पास एक पहुंच टोकन की प्रतिलिपि होती है। टोकन उपयोगकर्ता को पहचानता है, उपयोगकर्ता के समूहों को पहचानता है, और उपयोगकर्ता की विशेषाधिकारों को पहचानता है। टोकन में एक लॉगइन SID (सुरक्षा पहचानक) भी होता है जो वर्तमान लॉगइन सत्र की पहचान करता है।

आप इस सूचना को `whoami /all` को निष्पादित करके देख सकते हैं।
```
whoami /all

USER INFORMATION
----------------

User Name             SID
===================== ============================================
desktop-rgfrdxl\cpolo S-1-5-21-3359511372-53430657-2078432294-1001


GROUP INFORMATION
-----------------

Group Name                                                    Type             SID                                                                                                           Attributes
============================================================= ================ ============================================================================================================= ==================================================
Mandatory Label\Medium Mandatory Level                        Label            S-1-16-8192
Everyone                                                      Well-known group S-1-1-0                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account and member of Administrators group Well-known group S-1-5-114                                                                                                     Group used for deny only
BUILTIN\Administrators                                        Alias            S-1-5-32-544                                                                                                  Group used for deny only
BUILTIN\Users                                                 Alias            S-1-5-32-545                                                                                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Performance Log Users                                 Alias            S-1-5-32-559                                                                                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE                                      Well-known group S-1-5-4                                                                                                       Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                                                 Well-known group S-1-2-1                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users                              Well-known group S-1-5-11                                                                                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization                                Well-known group S-1-5-15                                                                                                      Mandatory group, Enabled by default, Enabled group
MicrosoftAccount\cpolop@outlook.com                           User             S-1-11-96-3623454863-58364-18864-2661722203-1597581903-3158937479-2778085403-3651782251-2842230462-2314292098 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account                                    Well-known group S-1-5-113                                                                                                     Mandatory group, Enabled by default, Enabled group
LOCAL                                                         Well-known group S-1-2-0                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Cloud Account Authentication                     Well-known group S-1-5-64-36                                                                                                   Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```
या तो _प्रक्रिया एक्सप्लोरर_ का उपयोग करके साइंटर्नल्स से (प्रक्रिया का चयन करें और "सुरक्षा" टैब तक पहुंचें):

![](<../../.gitbook/assets/image (321).png>)

### स्थानीय प्रशासक

जब स्थानीय प्रशासक लॉगिन करता है, **दो पहुंच टोकन बनाए जाते हैं**: एक वाणिज्यिक अधिकारों वाला और दूसरा सामान्य अधिकारों वाला। **डिफ़ॉल्ट रूप से**, जब यह उपयोगकर्ता कोई प्रक्रिया चलाता है, तो **सामान्य** (गैर-प्रशासक) **अधिकार वाला उपयोग होता है**। जब यह उपयोगकर्ता किसी भी प्रशासक के रूप में कुछ भी **चलाने** की कोशिश करता है ("एडमिनिस्ट्रेटर के रूप में चलाएं" उदाहरण के लिए), तो **UAC** अनुमति के लिए पूछने के लिए उपयोग होगा।\
यदि आप [**UAC के बारे में और अधिक जानना चाहते हैं तो इस पृष्ठ को पढ़ें**](../authentication-credentials-uac-and-efs.md#uac)**.**

### प्रमाणपत्र उपयोगकर्ता अनुकरण

यदि आपके पास किसी अन्य उपयोगकर्ता के **वैध प्रमाणपत्र** हैं, तो आप उन प्रमाणपत्रों के साथ एक **नई लॉगऑन सत्र** बना सकते हैं:
```
runas /user:domain\username cmd.exe
```
एक्सेस टोकन में **LSASS** के अंदर लॉगऑन सत्र का **संदर्भ** भी होता है, यह उपयोगी होता है अगर प्रक्रिया को नेटवर्क के कुछ ऑब्जेक्ट्स तक पहुंच की आवश्यकता हो।\
आप नेटवर्क सेवाओं तक पहुंच के लिए **विभिन्न क्रेडेंशियल का उपयोग करने वाली प्रक्रिया** चला सकते हैं:
```
runas /user:domain\username /netonly cmd.exe
```
यह उपयोगी होता है यदि आपके पास नेटवर्क में ऑब्जेक्ट तक पहुंच के लिए उपयोगी क्रेडेंशियल हैं, लेकिन वे क्रेडेंशियल मौजूदा होस्ट के अंदर मान्य नहीं हैं क्योंकि वे केवल नेटवर्क में उपयोग होंगे (मौजूदा होस्ट में आपकी मौजूदा उपयोगकर्ता विशेषाधिकार का उपयोग किया जाएगा)।

### टोकन के प्रकार

दो प्रकार के टोकन उपलब्ध होते हैं:

* **प्राथमिक टोकन**: प्राथमिक टोकन केवल **प्रक्रियाओं से संबद्ध किए जा सकते हैं**, और वे प्रक्रिया की सुरक्षा विषय को प्रतिष्ठित करते हैं। प्राथमिक टोकन के निर्माण और उनके प्रक्रियाओं से संबद्ध करने दोनों विशेषाधिकारिता की आवश्यकता होती है - प्राथमिक टोकन का निर्माण आमतौर पर प्रमाणीकरण सेवा द्वारा किया जाता है, और एक लॉगऑन सेवा उपयोगकर्ता के ऑपरेटिंग सिस्टम शैली से इसे संबद्ध करती है। प्रक्रियाएं प्राथमिक टोकन की मूल प्रक्रिया की प्रतिलिपि को आधिकारिक रूप से अनुग्रहित करती हैं।
* **अनुकरण टोकन**: अनुकरण एक सुरक्षा अवधारणा है जो Windows NT में लागू होती है और इसके माध्यम से एक सर्वर एप्लिकेशन को सुरक्षित ऑब्जेक्ट्स तक पहुंच के मामले में अस्थायी रूप से "ग्राहक" के रूप में कार्य करने की अनुमति देती है। अनुकरण के चार संभावित स्तर होते हैं:

* **अनामक**, जो सर्वर को एक अनामक / अज्ञात उपयोगकर्ता की पहुंच देता है
* **पहचान**, जो सर्वर को ग्राहक की पहचान जांचने देता है लेकिन उस पहचान का उपयोग ऑब्जेक्ट्स तक पहुंच करने के लिए नहीं कर सकता
* **अनुकरण**, जो सर्वर को ग्राहक के पक्ष में कार्य करने देता है
* **प्रतिनिधित्व**, अनुकरण के समान है लेकिन इसे सर्वर कनेक्ट करने के लिए दूरस्थ प्रणालियों तक विस्तारित किया जाता है (प्रमाणों के संरक्षण के माध्यम से)।

ग्राहक सर्वर के रूप में उपलब्ध अधिकतम अनुकरण स्तर (यदि कोई हो) को एक कनेक्शन पैरामीटर के रूप में चुन सकता है। अनुकरण और अनुकरण विशेषाधिकारिता की आवश्यकता होती है (अनुकरण पहले नहीं था, लेकिन ग्राहक API के अनुमति सीमित करने के लिए डिफ़ॉल्ट स्तर को "पहचान" तक सीमित न करने के इतिहासिक लापरवाही के कारण, जिससे एक अनुचित ग्राहक को अनुकरण करने की अनुमति दी जाती है)। **अनुकरण टोकन केवल थ्रेड्स से संबद्ध किए जा सकते हैं**, और वे ग्राहक प्रक्रिया की सुरक्षा विषय को प्रतिष्ठित करते हैं। अनुकरण टोकन आमतौर पर ब्यापार के माध्यम से वर्तमान थ्रेड के लिए स्वतः ही निर्मित और संबद्ध किए जाते हैं, जैसे DCE RPC, DDE और नेम्ड पाइप्स जैसे IPC मेकेनिज़मों द्वारा।

#### अनुकरण टोकन

मेटास्प्लोइट के _**incognito**_\*\* मॉड्यूल\*\* का उपयोग करके यदि आपके पास पर्याप्त विशेषाधिकार हैं, तो आप आसानी से अन्य **टोकनों** की **सूची** और **अनुकरण** कर सकते हैं। यह दूसरे उपयोगकर्ता के रूप में कार्यवाही करने के लिए उपयोगी हो सकता है। इस तकनीक के साथ आप विशेषाधिकारों को भी बढ़ा सकते हैं।

### टोकन विशेषाधिकार

यह जानें कि कौन से **टोकन विशेषाधिकार** विशेषाधिकारों को बढ़ाने के लिए उपयोग किए जा सकते हैं:

{% content-ref url="privilege-escalation-abusing-tokens/" %}
[privilege-escalation-abusing-tokens](privilege-escalation-abusing-tokens/)
{% endcontent-ref %}

[**इस बाहरी पृष्ठ पर सभी संभावित टोकन विशेषाधिकारों और कुछ परिभाषाएं देखें**](https://github.com/gtworek/Priv2Admin)।

## संदर्भ

इस ट्यूटोरियल में टोकन के बारे में और अधिक जानें: [https://medium.com/@seemant.bisht24/understanding-and-abusing-process-tokens-part-i-ee51671f2cfa](https://medium.com/@seemant.bisht24/understanding-and-abusing-process-tokens-part-i-ee51671f2cfa) और [https://medium.com/@seemant.bisht24/understanding-and-abusing-access-tokens-part-ii-b9069f432962](https://medium.com/@seemant.bisht24/understanding-and-abusing-access-tokens-part-ii-b9069f432962)

<details>
