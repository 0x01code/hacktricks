# macOS Red Teaming

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## MDMs का दुरुपयोग

* JAMF Pro: `jamf checkJSSConnection`
* Kandji

यदि आप प्रबंधन प्लेटफॉर्म तक पहुंचने के लिए **admin credentials को समझौता करने में सफल होते हैं**, तो आप **सभी कंप्यूटरों को संभावित रूप से समझौता कर सकते हैं** अपने मैलवेयर को मशीनों में वितरित करके।

MacOS वातावरणों में रेड टीमिंग के लिए MDMs के कामकाज की कुछ समझ होना अत्यधिक अनुशंसित है:

{% content-ref url="macos-mdm/" %}
[macos-mdm](macos-mdm/)
{% endcontent-ref %}

### MDM का C2 के रूप में उपयोग

एक MDM को प्रोफाइल्स को इंस्टॉल करने, क्वेरी करने या हटाने, एप्लिकेशन्स को इंस्टॉल करने, लोकल एडमिन अकाउंट्स बनाने, फर्मवेयर पासवर्ड सेट करने, FileVault की को बदलने की अनुमति होगी...

अपना खुद का MDM चलाने के लिए आपको एक **vendor द्वारा आपके CSR को साइन किया जाना चाहिए** जिसे आप [**https://mdmcert.download/**](https://mdmcert.download/) के साथ प्राप्त करने की कोशिश कर सकते हैं। और Apple डिवाइसेस के लिए अपना खुद का MDM चलाने के लिए आप [**MicroMDM**](https://github.com/micromdm/micromdm) का उपयोग कर सकते हैं।

हालांकि, एक enrolled डिवाइस में एक एप्लिकेशन इंस्टॉल करने के लिए, आपको अभी भी इसे एक डेवलपर अकाउंट द्वारा साइन किया जाना चाहिए... हालांकि, MDM में नामांकन के बाद **डिवाइस MDM के SSL सर्टिफिकेट को एक विश्वसनीय CA के रूप में जोड़ता है**, इसलिए अब आप कुछ भी साइन कर सकते हैं।

डिवाइस को MDM में नामांकित करने के लिए आपको एक **`mobileconfig`** फाइल को रूट के रूप में इंस्टॉल करना होगा, जिसे एक **pkg** फाइल के माध्यम से वितरित किया जा सकता है (आप इसे ज़िप में संपीड़ित कर सकते हैं और सफारी से डाउनलोड किए जाने पर यह डिकंप्रेस हो जाएगा)।

**Mythic agent Orthrus** इस तकनीक का उपयोग करता है।

### JAMF PRO का दुरुपयोग

JAMF **कस्टम स्क्रिप्ट्स** (सिस्टम एडमिन द्वारा विकसित स्क्रिप्ट्स), **नेटिव पेलोड्स** (लोकल अकाउंट क्रिएशन, EFI पासवर्ड सेट करना, फाइल/प्रोसेस मॉनिटरिंग...) और **MDM** (डिवाइस कॉन्फ़िगरेशन, डिवाइस सर्टिफिकेट्स...) चला सकता है।

#### JAMF स्व-नामांकन

ऐसे पेज पर जाएं जैसे कि `https://<company-name>.jamfcloud.com/enroll/` यह देखने के लिए कि क्या उन्होंने **स्व-नामांकन सक्षम किया है**। यदि उन्होंने इसे सक्षम किया है तो यह **पहुंच के लिए क्रेडेंशियल्स मांग सकता है**।

आप पासवर्ड स्प्रेइंग अटैक करने के लिए [**JamfSniper.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfSniper.py) स्क्रिप्ट का उपयोग कर सकते हैं।

इसके अलावा, उचित क्रेडेंशियल्स पाने के बाद आप अगले फॉर्म के साथ अन्य यूजरनेम्स को ब्रूट-फोर्स करने में सक्षम हो सकते हैं:

![](<../../.gitbook/assets/image (7) (1) (1).png>)

#### JAMF डिवाइस प्रमाणीकरण

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**`jamf`** बाइनरी में वह सीक्रेट शामिल था जो खोज के समय सभी के बीच **साझा** किया गया था और वह था: **`jk23ucnq91jfu9aj`**.\
इसके अलावा, jamf **`/Library/LaunchAgents/com.jamf.management.agent.plist`** में एक **LaunchDaemon** के रूप में **बना रहता है**।

#### JAMF डिवाइस टेकओवर

**JSS** (Jamf Software Server) **URL** जिसका उपयोग **`jamf`** करेगा वह स्थित है **`/Library/Preferences/com.jamfsoftware.jamf.plist`** में। \
यह फाइल मूल रूप से URL शामिल करती है:

{% code overflow="wrap" %}
```bash
plutil -convert xml1 -o - /Library/Preferences/com.jamfsoftware.jamf.plist

[...]
<key>is_virtual_machine</key>
<false/>
<key>jss_url</key>
<string>https://halbornasd.jamfcloud.com/</string>
<key>last_management_framework_change_id</key>
<integer>4</integer>
[...]
```
{% endcode %}

इसलिए, एक हमलावर एक हानिकारक पैकेज (`pkg`) डाल सकता है जो **इस फाइल को ओवरराइट करता है** जब इंस्टॉल होता है और **URL को एक Mythic C2 लिसनर पर सेट करता है जो Typhon एजेंट से आता है** ताकि अब JAMF का उपयोग C2 के रूप में किया जा सके।

{% code overflow="wrap" %}
```bash
# After changing the URL you could wait for it to be reloaded or execute:
sudo jamf policy -id 0

# TODO: There is an ID, maybe it's possible to have the real jamf connection and another one to the C2
```
#### JAMF अनुकरण

डिवाइस और JMF के बीच संचार का **अनुकरण करने** के लिए आपको चाहिए:

* डिवाइस का **UUID**: `ioreg -d2 -c IOPlatformExpertDevice | awk -F" '/IOPlatformUUID/{print $(NF-1)}'`
* **JAMF कीचेन** जो यहाँ से मिलेगा: `/Library/Application\ Support/Jamf/JAMF.keychain` जिसमें डिवाइस प्रमाणपत्र होता है

इस जानकारी के साथ, **एक VM बनाएं** जिसमें **चुराया गया** हार्डवेयर **UUID** हो और **SIP अक्षम** हो, **JAMF कीचेन डालें,** Jamf **एजेंट** को **हुक** करें और उसकी जानकारी चुराएं।

#### गोपनीय जानकारी चोरी

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>a</p></figcaption></figure>

आप `/Library/Application Support/Jamf/tmp/` स्थान की निगरानी भी कर सकते हैं जहाँ **कस्टम स्क्रिप्ट्स** को जमा किया जाता है, जिन्हें एडमिन्स Jamf के माध्यम से निष्पादित करना चाहते हैं क्योंकि वे **यहाँ रखे जाते हैं, निष्पादित किए जाते हैं और हटा दिए जाते हैं**। इन स्क्रिप्ट्स में **प्रमाणीकरण सामग्री हो सकती है**।

हालांकि, **प्रमाणीकरण सामग्री** इन स्क्रिप्ट्स में **पैरामीटर्स** के रूप में भी दी जा सकती है, इसलिए आपको `ps aux | grep -i jamf` की निगरानी करनी होगी (बिना रूट होने के भी)।

स्क्रिप्ट [**JamfExplorer.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfExplorer.py) नई फाइलों के जोड़े जाने और नए प्रोसेस आर्ग्युमेंट्स की निगरानी कर सकती है।

### macOS रिमोट एक्सेस

और **MacOS** के "विशेष" **नेटवर्क** **प्रोटोकॉल्स** के बारे में भी:

{% content-ref url="../macos-security-and-privilege-escalation/macos-protocols.md" %}
[macos-protocols.md](../macos-security-and-privilege-escalation/macos-protocols.md)
{% endcontent-ref %}

## एक्टिव डायरेक्टरी

कुछ मौकों पर आप पाएंगे कि **MacOS कंप्यूटर AD से जुड़ा हुआ है**। इस परिदृश्य में आपको एक्टिव डायरेक्टरी का **गणना करने** की कोशिश करनी चाहिए जैसा कि आप इस्तेमाल करते हैं। निम्नलिखित पृष्ठों में कुछ **सहायता** मिल सकती है:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/active-directory-methodology/" %}
[active-directory-methodology](../../windows-hardening/active-directory-methodology/)
{% endcontent-ref %}

{% content-ref url="../../network-services-pentesting/pentesting-kerberos-88/" %}
[pentesting-kerberos-88](../../network-services-pentesting/pentesting-kerberos-88/)
{% endcontent-ref %}

कुछ **स्थानीय MacOS टूल** जो आपकी सहायता कर सकते हैं वह है `dscl`:
```bash
dscl "/Active Directory/[Domain]/All Domains" ls /
```
मैकओएस के लिए कुछ टूल्स तैयार किए गए हैं जो स्वचालित रूप से AD का परिगणना करते हैं और केर्बरोस के साथ खेलते हैं:

* [**Machound**](https://github.com/XMCyber/MacHound): MacHound ब्लडहाउंड ऑडिटिंग टूल का एक विस्तार है जो मैकओएस होस्ट्स पर एक्टिव डायरेक्टरी संबंधों को एकत्रित करने और उन्हें इंजेस्ट करने की अनुमति देता है।
* [**Bifrost**](https://github.com/its-a-feature/bifrost): Bifrost एक Objective-C प्रोजेक्ट है जिसे मैकओएस पर Heimdal krb5 APIs के साथ इंटरैक्ट करने के लिए डिजाइन किया गया है। इस प्रोजेक्ट का लक्ष्य मैकओएस डिवाइसेज पर केर्बरोस के आसपास बेहतर सुरक्षा परीक्षण को सक्षम करना है, जिसमें नेटिव APIs का उपयोग किया जाता है बिना किसी अन्य फ्रेमवर्क या पैकेज की आवश्यकता के लक्ष्य पर।
* [**Orchard**](https://github.com/its-a-feature/Orchard): एक्टिव डायरेक्टरी परिगणना करने के लिए JavaScript for Automation (JXA) टूल।

### डोमेन जानकारी
```bash
echo show com.apple.opendirectoryd.ActiveDirectory | scutil
```
### उपयोगकर्ता

MacOS के तीन प्रकार के उपयोगकर्ता होते हैं:

* **स्थानीय उपयोगकर्ता** — स्थानीय OpenDirectory सेवा द्वारा प्रबंधित, ये Active Directory से किसी भी प्रकार से जुड़े नहीं होते हैं।
* **नेटवर्क उपयोगकर्ता** — अस्थायी Active Directory उपयोगकर्ता जिन्हें प्रमाणीकरण के लिए DC सर्वर से कनेक्शन की आवश्यकता होती है।
* **मोबाइल उपयोगकर्ता** — Active Directory उपयोगकर्ता जिनके प्रमाणपत्र और फाइलों का स्थानीय बैकअप होता है।

उपयोगकर्ताओं और समूहों के स्थानीय जानकारी _/var/db/dslocal/nodes/Default_ फोल्डर में संग्रहीत की जाती है।\
उदाहरण के लिए, _mark_ नामक उपयोगकर्ता की जानकारी _/var/db/dslocal/nodes/Default/users/mark.plist_ में और _admin_ समूह की जानकारी _/var/db/dslocal/nodes/Default/groups/admin.plist_ में संग्रहीत है।

HasSession और AdminTo edges का उपयोग करने के अलावा, **MacHound तीन नए edges को Bloodhound डेटाबेस में जोड़ता है**:

* **CanSSH** - उस होस्ट पर SSH करने की अनुमति वाला इकाई
* **CanVNC** - उस होस्ट पर VNC करने की अनुमति वाला इकाई
* **CanAE** - उस होस्ट पर AppleEvent स्क्रिप्ट्स को निष्पादित करने की अनुमति वाला इकाई
```bash
#User enumeration
dscl . ls /Users
dscl . read /Users/[username]
dscl "/Active Directory/TEST/All Domains" ls /Users
dscl "/Active Directory/TEST/All Domains" read /Users/[username]
dscacheutil -q user

#Computer enumeration
dscl "/Active Directory/TEST/All Domains" ls /Computers
dscl "/Active Directory/TEST/All Domains" read "/Computers/[compname]$"

#Group enumeration
dscl . ls /Groups
dscl . read "/Groups/[groupname]"
dscl "/Active Directory/TEST/All Domains" ls /Groups
dscl "/Active Directory/TEST/All Domains" read "/Groups/[groupname]"

#Domain Information
dsconfigad -show
```
अधिक जानकारी के लिए [https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/](https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/)

## कीचेन तक पहुँच

कीचेन में संवेदनशील जानकारी हो सकती है जिसे बिना प्रॉम्प्ट उत्पन्न किए पहुँचने से रेड टीम अभ्यास को आगे बढ़ाने में मदद मिल सकती है:

{% content-ref url="macos-keychain.md" %}
[macos-keychain.md](macos-keychain.md)
{% endcontent-ref %}

## बाहरी सेवाएँ

MacOS रेड टीमिंग एक सामान्य Windows रेड टीमिंग से अलग है क्योंकि आमतौर पर **MacOS कई बाहरी प्लेटफॉर्मों के साथ सीधे एकीकृत होता है**। MacOS का एक सामान्य कॉन्फ़िगरेशन **OneLogin सिंक्रनाइज़्ड क्रेडेंशियल्स का उपयोग करके कंप्यूटर तक पहुँचना है, और कई बाहरी सेवाओं तक पहुँचना है** (जैसे github, aws...) OneLogin के माध्यम से:

![](<../../.gitbook/assets/image (563).png>)

## विविध रेड टीम तकनीकें

### सफारी

जब सफारी में कोई फ़ाइल डाउनलोड की जाती है, अगर वह एक "सुरक्षित" फ़ाइल है, तो वह **स्वतः खुल जाएगी**। उदाहरण के लिए, अगर आप **एक ज़िप डाउनलोड करते हैं**, तो वह स्वतः डिकंप्रेस हो जाएगी:

<figure><img src="../../.gitbook/assets/image (12) (3).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**https://www.youtube.com/watch?v=IiMladUbL6E**](https://www.youtube.com/watch?v=IiMladUbL6E)
* [**https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6**](https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6)
* [**https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0**](https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0)
* [**Come to the Dark Side, We Have Apples: Turning macOS Management Evil**](https://www.youtube.com/watch?v=pOQOh07eMxY)
* [**OBTS v3.0: "An Attackers Perspective on Jamf Configurations" - Luke Roberts / Calum Hall**](https://www.youtube.com/watch?v=ju1IYWUv4ZA)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम एक्सपर्ट)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** को अपनी हैकिंग ट्रिक्स साझा करके PRs जमा करें [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
