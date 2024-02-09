# लिनक्स एक्टिव डायरेक्टरी

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करने का एक्सेस** चाहिए? [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटीज़**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>

एक लिनक्स मशीन एक एक्टिव डायरेक्टरी वातावरण में भी मौजूद हो सकती है।

एक एडी में एक लिनक्स मशीन **विभिन्न सीकैच टिकट्स को फ़ाइलों में संग्रहित कर सकता है। ये टिकट्स और किसी अन्य केरबेरोस टिकट के रूप में उपयोग और दुरुपयोग किया जा सकता है**। इन टिकट्स को पढ़ने के लिए आपको टिकट के उपयोगकर्ता मालिक या **मशीन के अंदर रूट** होना चाहिए।

## जांच

### लिनक्स से एडी जांच

यदि आपके पास लिनक्स में एडी पर पहुंच है (या विंडोज में बैश है) तो आप [https://github.com/lefayjey/linWinPwn](https://github.com/lefayjey/linWinPwn) का उपयोग करके एडी का जांच कर सकते हैं।

आप लिनक्स से एडी को जांचने के **अन्य तरीके सीखने के लिए निम्नलिखित पृष्ठ की जांच कर सकते हैं**:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

### FreeIPA

FreeIPA माइक्रोसॉफ्ट विंडोज **एक्टिव डायरेक्टरी** के लिए एक ओपन-सोर्स **वैकल्पिक** है, मुख्य रूप से **यूनिक्स** वातावरणों के लिए। यह एक पूर्ण **एलडीएपी निर्देशिका** को एक MIT **केरबेरोस** कुंजी वितरण केंद्र के साथ मिलाकर एक्टिव डायरेक्टरी के समान प्रबंधन के लिए है। CA और RA प्रमाणपत्र प्रबंधन के लिए डॉगटैग **सर्टिफिकेट सिस्टम** का उपयोग करते हुए, यह स्मार्टकार्ड सहित **मल्टी-फैक्टर** प्रमाणीकरण का समर्थन करता है। यूनिक्स प्रमाणीकरण प्रक्रियाओं के लिए SSSD एकीकृत है। इसके बारे में अधिक जानने के लिए यहाँ देखें:

{% content-ref url="../freeipa-pentesting.md" %}
[freeipa-pentesting.md](../freeipa-pentesting.md)
{% endcontent-ref %}

## टिकट्स के साथ खेलना

### पास द टिकट

इस पृष्ठ पर आपको विभिन्न स्थान मिलेंगे जहाँ आप **लिनक्स होस्ट में केरबेरोस टिकट्स पाएंगे**, आगे के पृष्ठ पर आप सीसीचेच टिकट्स के फॉर्मेट को किर्बी में बदलना सीख सकते हैं (जिसे आपको विंडोज में उपयोग करने के लिए आवश्यकता है) और यहाँ तक कि पीटीटी हमला कैसे करें:

{% content-ref url="../../windows-hardening/active-directory-methodology/pass-the-ticket.md" %}
[pass-the-ticket.md](../../windows-hardening/active-directory-methodology/pass-the-ticket.md)
{% endcontent-ref %}

### /tmp से सीसीचेच टिकट पुनः उपयोग

सीसीचेच फ़ाइलें **केरबेरोस क्रेडेंशियल्स को संग्रहित करने के लिए बाइनरी फ़ॉर्मेट** होती हैं जो आम तौर पर `/tmp` में 600 अनुमतियों के साथ संग्रहित की जाती हैं। इन फ़ाइलों को उनके **नाम प्रारूप, `krb5cc_%{uid}`,** के साथ पहचाना जा सकता है, जो उपयोक्ता के UID से संबंधित है। प्रमाणीकरण टिकट सत्यापन के लिए, **पर्यावरण चर `KRB5CCNAME`** को वांछित टिकट फ़ाइल के पथ पर सेट किया जाना चाहिए, जिससे इसका पुनः उपयोग हो सके।

वर्तमान प्रमाणीकरण के लिए उपयोग किए जा रहे टिकट की सूची `env | grep KRB5CCNAME` के साथ देखें। यह फ़ॉर्मेट पोर्टेबल है और टिकट को **पर्यावरण चर को सेट करके पुनः उपयोग किया जा सकता है** `export KRB5CCNAME=/tmp/ticket.ccache`। केरबेरोस टिकट नाम प्रारूप `krb5cc_%{uid}` है जहाँ uid उपयोक्ता UID है।
```bash
# Find tickets
ls /tmp/ | grep krb5cc
krb5cc_1000

# Prepare to use it
export KRB5CCNAME=/tmp/krb5cc_1000
```
### कीकैश टिकट कुंजीडान से पुनः उपयोग

**प्रक्रिया की स्मृति में संग्रहीत केरबेरोस टिकट निकाले जा सकते हैं**, विशेष रूप से जब मशीन का पीट्रेस संरक्षण अक्षम होता है (`/proc/sys/kernel/yama/ptrace_scope`). इस उद्देश्य के लिए एक उपयुक्त उपकरण [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey) पर पाया जा सकता है, जो सत्रों में इंजेक्शन करके टिकटों को डंप करने में सहायक है और उन्हें `/tmp` में डंप करता है।

इस उपकरण को कॉन्फ़िगर और उपयोग करने के लिए, नीचे दिए गए चरणों का पालन किया जाता है:
```bash
git clone https://github.com/TarlogicSecurity/tickey
cd tickey/tickey
make CONF=Release
/tmp/tickey -i
```
यह प्रक्रिया विभिन्न सत्रों में इंजेक्शन करने का प्रयास करेगी, उत्पन्न टिकट को `/tmp` में `__krb_UID.ccache` नामक संरचना के साथ संग्रहित करके सफलता की संकेत देगी।


### SSSD KCM से CCACHE टिकट पुनः उपयोग

SSSD एक प्रतिलिपि डेटाबेस को पथ `/var/lib/sss/secrets/secrets.ldb` पर बनाए रखता है। संबंधित कुंजी केवल रूट अनुमतियों वाले होते हैं, जो पथ `/var/lib/sss/secrets/.secrets.mkey` पर एक छिपी हुई फ़ाइल के रूप में संग्रहीत है। डिफ़ॉल्ट रूप से, यदि आपके पास **रूट** अनुमतियाँ हैं तो केवल कुंजी पठनीय है।

--डेटाबेस और --कुंजी पैरामीटर के साथ \*\*`SSSDKCMExtractor` \*\* को आह्वान करना डेटाबेस को विश्लेषित करेगा और **रहस्यों को डिक्रिप्ट** करेगा।
```bash
git clone https://github.com/fireeye/SSSDKCMExtractor
python3 SSSDKCMExtractor.py --database secrets.ldb --key secrets.mkey
```
**प्रमाणपत्र कैश कर्बेरोस ब्लॉब को एक उपयोगी कर्बेरोस सीकैश फ़ाइल में रूपांतरित किया जा सकता है** जो Mimikatz/Rubeus को पारित किया जा सकता है।

### केटैब से सीकैश टिकट पुनः उपयोग करें
```bash
git clone https://github.com/its-a-feature/KeytabParser
python KeytabParser.py /etc/krb5.keytab
klist -k /etc/krb5.keytab
```
### /etc/krb5.keytab से खाते निकालें

सेवा खाते कुंजी, जो जड़ स्तर की अनुमतियों के साथ काम करने वाली सेवाओं के लिए महत्वपूर्ण हैं, सुरक्षित रूप से **`/etc/krb5.keytab`** फ़ाइलों में संग्रहीत होती हैं। ये कुंजियाँ, सेवाओं के लिए पासवर्ड के समान, कड़ी गोपनीयता की मांग करती हैं।

कुंजीपटल फ़ाइल की सामग्री की जांच के लिए, **`klist`** का उपयोग किया जा सकता है। यह उपकरण कुंजी विवरण प्रदर्शित करने के लिए डिज़ाइन किया गया है, जिसमें **NT Hash** उपयोगकर्ता प्रमाणीकरण के लिए, विशेष रूप से जब कुंजी प्रकार को 23 के रूप में पहचाना जाता है।
```bash
klist.exe -t -K -e -k FILE:C:/Path/to/your/krb5.keytab
# Output includes service principal details and the NT Hash
```
Linux उपयोगकर्ताओं के लिए, **`KeyTabExtract`** एक कार्यक्षमता प्रदान करता है जिसका उपयोग RC4 HMAC हैश निकालने के लिए किया जा सकता है, जो NTLM हैश पुनः उपयोग के लिए उपयोग किया जा सकता है।
```bash
python3 keytabextract.py krb5.keytab
# Expected output varies based on hash availability
```
मैकओएस पर, **`bifrost`** कुंजीटैब फ़ाइल विश्लेषण के लिए एक उपकरण के रूप में काम करता है।
```bash
./bifrost -action dump -source keytab -path /path/to/your/file
```
कोड और हैश जानकारी का उपयोग करते हुए, **`crackmapexec`** जैसे उपकरण का उपयोग करके सर्वरों के साथ कनेक्शन स्थापित किया जा सकता है।
```bash
crackmapexec 10.XXX.XXX.XXX -u 'ServiceAccount$' -H "HashPlaceholder" -d "YourDOMAIN"
```
## संदर्भ
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey)
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* खोजें [**द पीएएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक पीएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
