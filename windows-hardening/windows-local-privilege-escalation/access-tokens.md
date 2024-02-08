# एक्सेस टोकन्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>

## एक्सेस टोकन्स

**प्रत्येक उपयोगकर्ता** जो सिस्टम पर **लॉग इन** होता है, **उसके पास सुरक्षा सूचना के साथ एक्सेस टोकन होता है**। सिस्टम एक एक्सेस टोकन बनाता है जब उपयोगकर्ता लॉग इन करता है। **प्रत्येक प्रक्रिया** जो उपयोगकर्ता के लिए **चलाई जाती है, उसके पास एक्सेस टोकन की एक प्रति होती है**। टोकन उपयोगकर्ता, उपयोगकर्ता के समूह और उपयोगकर्ता की विशेषाधिकारों की पहचान करता है। एक टोकन में एक लॉग इन एसआईडी (सुरक्षा पहचानकर्ता) भी होता है जो वर्तमान लॉग इन सत्र की पहचान करता है।

आप इस सूचना को देख सकते हैं `whoami /all` का निष्पादन करके।
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
या _प्रक्रिया एक्सप्लोरर_ से Sysinternals (प्रक्रिया का चयन करें और "सुरक्षा" टैब तक पहुंचें):

![](<../../.gitbook/assets/image (321).png>)

### स्थानीय प्रशासक

जब एक स्थानीय प्रशासक लॉगिन करता है, **दो एक्सेस टोकन बनाए जाते हैं**: एक विशेषाधिकार वाला और दूसरा सामान्य अधिकार वाला। **डिफ़ॉल्ट** रूप से, जब यह उपयोगकर्ता प्रक्रिया को निष्क्रिय करता है तो **सामान्य** (प्रशासक नहीं) **अधिकार वाला उपयोग किया जाता है**। जब यह उपयोगकर्ता किसी भी चीज़ को **प्रशासक के रूप में** चलाने की कोशिश करता है ("प्रशासक के रूप में चलाएं" उदाहरण के लिए) तो **UAC** अनुमति के लिए पूछने के लिए उपयोग किया जाएगा।\
यदि आप [**UAC के बारे में अधिक जानना चाहते हैं तो इस पृष्ठ को पढ़ें**](../authentication-credentials-uac-and-efs.md#uac)**.**

### प्रमाणों उपयोगकर्ता प्रतिनिधित्व

यदि आपके पास **किसी अन्य उपयोगकर्ता के मान्य प्रमाण हैं**, तो आप उन प्रमाणों के साथ एक **नया लॉगऑन सत्र बना सकते हैं** :
```
runas /user:domain\username cmd.exe
```
**एक्सेस टोकन** में **LSASS** के अंदर लॉगऑन सत्र का **संदर्भ** भी होता है, यह उपयोगी है अगर प्रक्रिया को नेटवर्क के कुछ ऑब्जेक्ट्स तक पहुंचने की आवश्यकता हो।\
आप नेटवर्क सेवाओं तक पहुंचने के लिए **विभिन्न क्रेडेंशियल का उपयोग करने वाली प्रक्रिया लॉन्च** कर सकते हैं:
```
runas /user:domain\username /netonly cmd.exe
```
### पहुंच टोकन्स के प्रकार

दो प्रकार के टोकन्स उपलब्ध हैं:

* **प्राथमिक टोकन**: यह प्रक्रिया की सुरक्षा पहचान का प्रतिनिधित्व करता है। प्राथमिक टोकन का निर्माण और प्रक्रियाओं के साथ संबंधित क्रियाएं उच्च विशेषाधिकारों की आवश्यकता है, विशेषाधिकार के सिद्धांत को जोर देते हैं। सामान्यत: प्रमाणीकरण सेवा टोकन का निर्माण करने के लिए जिम्मेदार होती है, जबकि लॉगऑन सेवा इसे उपयोगकर्ता के ऑपरेटिंग सिस्टम शैली के साथ संबंधित करती है। यह ध्यान देने योग्य है कि प्रक्रियाएं अपने माता प्रक्रिया का प्राथमिक टोकन विरासत में प्राप्त करती हैं।

* **अनुकरण टोकन**: एक सर्वर एप्लिकेशन को सुरक्षित ऑब्जेक्ट तक पहुंच के लिए अस्थायी रूप से ग्राहक की पहचान अपनाने की अधिकारी बनाता है। यह तंत्र चार स्तरों में विभाजित है:
- **अनामित**: सर्वर को एक अनपहचानित उपयोगकर्ता के समान पहुंच प्रदान करता है।
- **पहचान**: सर्वर को ऑब्जेक्ट पहुंच के लिए उसका उपयोग किए बिना ग्राहक की पहचान सत्यापित करने की अनुमति देता है।
- **अनुकरण**: सर्वर को ग्राहक की पहचान के तहत काम करने की सक्षमता प्रदान करता है।
- **अधिकार**: अनुकरण के समान है, लेकिन इस व्यक्ति को दूरस्थ सिस्टमों के साथ इस पहचान को विस्तारित करने की क्षमता भी शामिल है, प्रमाण प्रेजर्वेशन सुनिश्चित करते हुए।
