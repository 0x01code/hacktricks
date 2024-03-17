# macOS रेड टीमिंग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## MDMs का दुरुपयोग

* JAMF Pro: `jamf checkJSSConnection`
* Kandji

यदि आप **व्यवस्थापन प्लेटफॉर्म तक पहुंचने के लिए व्यवस्थापक क्रेडेंशियल्स को कंप्रमाइज** करने में सफल होते हैं, तो आप मशीनों में अपने मैलवेयर को वितरित करके **संभावित रूप से सभी कंप्यूटर्स को कंप्रमाइज** कर सकते हैं।

MacOS वातावरण में रेड टीमिंग के लिए MDMs के काम करने की कुछ समझ होना अत्यंत अनुशंसित है:

{% content-ref url="macos-mdm/" %}
[macos-mdm](macos-mdm/)
{% endcontent-ref %}

### MDM का C2 के रूप में उपयोग

एक MDM को इंस्टॉल, क्वेरी या प्रोफाइल हटाने, एप्लिकेशन इंस्टॉल करने, स्थानीय व्यवस्थापक खाते बनाने, फर्मवेयर पासवर्ड सेट करने, फाइलवॉल्ट कुंजी बदलने की अनुमति होगी...

अपने खुद के MDM को चलाने के लिए आपको **एक विक्रेता द्वारा साइन किया गया CSR** चाहिए होगा जिसे आप [**https://mdmcert.download/**](https://mdmcert.download/) से प्राप्त करने का प्रयास कर सकते हैं। और Apple डिवाइस के लिए अपना खुद का MDM चलाने के लिए आप [**MicroMDM**](https://github.com/micromdm/micromdm) का उपयोग कर सकते हैं।

हालांकि, एक इंस्टॉल करने के लिए एप्लिकेशन को एक नामकरण खाते से साइन करने की आवश्यकता है... हालांकि, MDM नामांकन के बाद **डिवाइस MDM को SSL सर्टिफिकेट के रूप में विश्वसनीय CA के रूप में जोड़ता है**, इसलिए अब आप कुछ भी साइन कर सकते हैं।

एक MDM में डिवाइस को नामांकित करने के लिए आपको एक **`mobileconfig`** फ़ाइल को रूट के रूप में इंस्टॉल करने की आवश्यकता है, जो एक **pkg** फ़ाइल के माध्यम से पहुंचाया जा सकता है (आप इसे जब Safari से डाउनलोड करते हैं तो इसे अनज़िप किया जाएगा)।

**Mythic एजेंट Orthrus** इस तकनीक का उपयोग करता है।

### JAMF PRO का दुरुपयोग

JAMF **कस्टम स्क्रिप्ट** (सिस्टम व्यवस्थापक द्वारा विकसित स्क्रिप्ट), **नेटिव पेलोड्स** (स्थानीय खाता बनाना, EFI पासवर्ड सेट करना, फ़ाइल/प्रोसेस मॉनिटरिंग...) और **MDM** (डिवाइस कॉन्फ़िगरेशन, डिवाइस सर्टिफिकेटेस...) चला सकता है।

#### JAMF स्वयं-नामांकन

उनके पास **स्वयं-नामांकन सक्षम** है यह देखने के लिए कि क्या वे किसी पृष्ठ पर जाते हैं `https://<company-name>.jamfcloud.com/enroll/`। यदि उनके पास है तो यह **पहुंचने के लिए क्रेडेंशियल्स का अनुरोध कर सकता है**।

आप [**JamfSniper.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfSniper.py) स्क्रिप्ट का उपयोग करके पासवर्ड स्प्रेयिंग हमला कर सकते हैं।

इसके अतिरिक्त, उचित क्रेडेंशियल्स खोजने के बाद आप अन्य उपयोगकर्ता नामों को ब्रूट-फ़ोर्स कर सकते हैं अगले फ़ॉर्म के साथ:

![](<../../.gitbook/assets/image (7) (1) (1).png>)

#### JAMF डिवाइस प्रमाणीकरण

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**`jamf`** बाइनरी में चाबी शामिल थी जो की खोलने के लिए सीक्रेट को जिसे खोजने का समय था **सभी के बीच साझा** था और वह था: **`jk23ucnq91jfu9aj`**।\
इसके अतिरिक्त, jamf **`/Library/LaunchAgents/com.jamf.management.agent.plist`** में **लॉन्चडेमन** के रूप में **परिस्थित** रहता है।

#### JAMF डिवाइस ताकता

**JSS** (Jamf Software Server) **URL** जिसे **`jamf`** उपयोग करेगा वह **`/Library/Preferences/com.jamfsoftware.jamf.plist`** में स्थित है।\
यह फ़ाइल मूल रूप से URL शामिल करती है:
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

इसलिए, एक हमलावर एक दुष्ट पैकेज (`pkg`) छोड़ सकता है जो **इस फ़ाइल को अधिलिखित करता है** जब इंस्टॉल किया जाता है, जिससे **URL को एक Mythic C2 सुनने वाले से Typhon एजेंट के रूप में सेट किया जाता है** अब JAMF का दुरुपयोग करने के लिए।

{% code overflow="wrap" %}
```bash
# After changing the URL you could wait for it to be reloaded or execute:
sudo jamf policy -id 0

# TODO: There is an ID, maybe it's possible to have the real jamf connection and another one to the C2
```
{% endcode %}

#### JAMF अनुरूपण

एक उपकरण और JMF के बीच **संचार का अनुकरण** करने के लिए निम्नलिखित आवश्यक है:

* उपकरण का **UUID**: `ioreg -d2 -c IOPlatformExpertDevice | awk -F" '/IOPlatformUUID/{print $(NF-1)}'`
* **JAMF keychain** से: `/Library/Application\ Support/Jamf/JAMF.keychain` जिसमें उपकरण प्रमाणपत्र है

इस जानकारी के साथ, **चोरी** हुए हार्डवेयर **UUID** और **SIP अक्षम** के साथ एक VM **बनाएं**, **JAMF keychain** को ड्रॉप करें, Jamf **एजेंट** को **हुक** करें और उसकी जानकारी चुरा लें।

#### रहस्य चुराना

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>a</p></figcaption></figure>

आप भी स्थान `/Library/Application Support/Jamf/tmp/` का मॉनिटर कर सकते हैं जहां **कस्टम स्क्रिप्ट** एडमिनिस्ट्रेटर्स जम्फ के माध्यम से निष्पादित करना चाहेंगे क्योंकि ये यहां **रखे जाते हैं, निष्पादित किए जाते हैं और हटा दिए जाते हैं**। ये स्क्रिप्ट **प्रमाणपत्र** भी हो सकते हैं।

हालांकि, **प्रमाणपत्र** इन स्क्रिप्ट्स के माध्यम से **पारित** किए जा सकते हैं, इसलिए आपको `ps aux | grep -i jamf` (बिना रूट होने के भी) का मॉनिटरिंग करने की आवश्यकता होगी।

स्क्रिप्ट [**JamfExplorer.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfExplorer.py) नए फ़ाइलों और नए प्रक्रिया तर्कों के लिए सुन सकता है।

### macOS रिमोट एक्सेस

और भी **MacOS** "विशेष" **नेटवर्क** **प्रोटोकॉल** के बारे में:

{% content-ref url="../macos-security-and-privilege-escalation/macos-protocols.md" %}
[macos-protocols.md](../macos-security-and-privilege-escalation/macos-protocols.md)
{% endcontent-ref %}

## सक्रिय निर्देशिका

कुछ मौकों पर आपको पता चलेगा कि **MacOS कंप्यूटर एक AD से कनेक्टेड है**। इस स्थिति में आपको इसे जैसा आप उसे उपयोग करते हैं **एक्टिव डायरेक्टरी** को **जांचने** की कोशिश करनी चाहिए। निम्नलिखित पृष्ठों में कुछ **मदद** मिल सकती है:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/active-directory-methodology/" %}
[active-directory-methodology](../../windows-hardening/active-directory-methodology/)
{% endcontent-ref %}

{% content-ref url="../../network-services-pentesting/pentesting-kerberos-88/" %}
[pentesting-kerberos-88](../../network-services-pentesting/pentesting-kerberos-88/)
{% endcontent-ref %}

कुछ **स्थानीय MacOS उपकरण** जो आपकी मदद कर सकते हैं है `dscl`:
```bash
dscl "/Active Directory/[Domain]/All Domains" ls /
```
इसके अलावा, कुछ टूल MacOS के लिए तैयार किए गए हैं जो स्वचालित रूप से AD की जांच करते हैं और केरबेरोस के साथ खेलते हैं:

* [**Machound**](https://github.com/XMCyber/MacHound): MacHound एक एक्सटेंशन है जो Bloodhound ऑडिटिंग टूल को एक्टिव डायरेक्टरी संबंधों को MacOS होस्ट पर संकलित और इंजेस्ट करने की अनुमति देता है।
* [**Bifrost**](https://github.com/its-a-feature/bifrost): Bifrost एक Objective-C परियोजना है जो macOS पर Heimdal krb5 APIs के साथ बातचीत करने के लिए डिज़ाइन किया गया है। परियोजना का उद्देश्य यह है कि लक्ष्य को बेहतर सुरक्षा परीक्षण को सक्षम करना है जो macOS उपकरणों पर केरबेरोस के आसपास नेटिव APIs का उपयोग करते हैं बिना लक्ष्य पर किसी अन्य फ्रेमवर्क या पैकेज की आवश्यकता हो।
* [**Orchard**](https://github.com/its-a-feature/Orchard): Active Directory जांच करने के लिए JavaScript for Automation (JXA) टूल।

### डोमेन सूचना
```bash
echo show com.apple.opendirectoryd.ActiveDirectory | scutil
```
### उपयोगकर्ता

मैकओएस उपयोगकर्ताओं के तीन प्रकार हैं:

* **स्थानीय उपयोगकर्ता** — स्थानीय ओपन डायरेक्टरी सेवा द्वारा प्रबंधित, वे किसी भी तरह से सक्रिय नहीं हैं एक्टिव डायरेक्टरी से कनेक्टेड।
* **नेटवर्क उपयोगकर्ता** — सक्रिय एक्टिव डायरेक्टरी उपयोगकर्ता जिन्हें प्रमाणित करने के लिए डीसी सर्वर से कनेक्शन की आवश्यकता होती है।
* **मोबाइल उपयोगकर्ता** — स्थानीय बैकअप के साथ एक्टिव डायरेक्टरी उपयोगकर्ता जिनके पास उनके क्रेडेंशियल्स और फ़ाइलों के लिए स्थानीय बैकअप है।

उपयोगकर्ताओं और समूहों के बारे में स्थानीय जानकारी _/var/db/dslocal/nodes/Default_ फ़ोल्डर में संग्रहित है।\
उदाहरण के लिए, _mark_ नाम के उपयोगकर्ता के बारे में जानकारी _/var/db/dslocal/nodes/Default/users/mark.plist_ में संग्रहित है और _admin_ समूह के बारे में जानकारी _/var/db/dslocal/nodes/Default/groups/admin.plist_ में है।

हैसेशन और एडमिन टू एजेज का उपयोग करने के अतिरिक्त, **MacHound तीन नए एजेज** ब्लडहाउंड डेटाबेस में जोड़ता है:

* **CanSSH** - एंटिटी जिसे होस्ट पर SSH करने की अनुमति है
* **CanVNC** - एंटिटी जिसे होस्ट पर VNC करने की अनुमति है
* **CanAE** - एंटिटी जिसे होस्ट पर AppleEvent स्क्रिप्ट को निष्पादित करने की अनुमति है
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
अधिक जानकारी [https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/](https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/) में मिलेगी।

## कीचेन तक पहुंचना

कीचेन में संवेदनशील जानकारी हो सकती है जिसे एक लाल टीम अभ्यास में आगे बढ़ने में मदद मिल सकती है अगर बिना प्रॉम्प्ट उत्पन्न किए एक्सेस किया जाए:

{% content-ref url="macos-keychain.md" %}
[macos-keychain.md](macos-keychain.md)
{% endcontent-ref %}

## बाह्य सेवाएं

MacOS रेड टीमिंग एक सामान्य Windows रेड टीमिंग से अलग है क्योंकि आम तौर पर **MacOS कई बाह्य प्लेटफॉर्मों के साथ सीधे एकीकृत होता है**। MacOS का एक सामान्य कॉन्फ़िगरेशन कंप्यूटर तक पहुंचने की अनुमति देना है **OneLogin समकालीन प्रमाणों का उपयोग करके, और OneLogin के माध्यम से कई बाह्य सेवाओं तक पहुंचना** (जैसे github, aws...)।

## विविध रेड टीम तकनीकें

### सफारी

जब सफारी में कोई फ़ाइल डाउनलोड होती है, और यह एक "सुरक्षित" फ़ाइल है, तो यह **स्वचालित रूप से खोल दी जाएगी**। इसलिए उदाहरण के लिए, अगर आप **एक zip फ़ाइल डाउनलोड** करते हैं, तो यह स्वचालित रूप से डीकंप्रेस हो जाएगी:

<figure><img src="../../.gitbook/assets/image (12) (3).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**https://www.youtube.com/watch?v=IiMladUbL6E**](https://www.youtube.com/watch?v=IiMladUbL6E)
* [**https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6**](https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6)
* [**https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0**](https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0)
* [**Come to the Dark Side, We Have Apples: Turning macOS Management Evil**](https://www.youtube.com/watch?v=pOQOh07eMxY)
* [**OBTS v3.0: "An Attackers Perspective on Jamf Configurations" - Luke Roberts / Calum Hall**](https://www.youtube.com/watch?v=ju1IYWUv4ZA)
