# macOS रेड टीमिंग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

## MDM का दुरुपयोग

* JAMF Pro: `jamf checkJSSConnection`
* Kandji

यदि आप **व्यवस्थापन प्लेटफ़ॉर्म तक प्रशासनिक प्रमाणिका** को संक्रमित करने में सफल होते हैं, तो आप **संभावित रूप से सभी कंप्यूटरों को संक्रमित कर सकते हैं** अपने मैलवेयर को मशीनों में वितरित करके।

MacOS पर रेड टीमिंग के लिए MDM के काम करने की कुछ समझ होना अत्यंत अनुशंसित है:

{% content-ref url="macos-mdm/" %}
[macos-mdm](macos-mdm/)
{% endcontent-ref %}

### MDM का उपयोग C2 के रूप में

MDM को स्थापित करने की अनुमति होगी, प्रोफ़ाइल स्थापित, प्रश्न करें या हटाएं, एप्लिकेशन स्थापित करें, स्थानीय व्यवस्थापक खाते बनाएं, फर्मवेयर्ट पासवर्ड सेट करें, फ़ाइलवॉल्ट कुंजी बदलें...

अपने खुद के MDM को चलाने के लिए आपको एक विक्रेता द्वारा **आपके सीएसआर को साइन करने की आवश्यकता होगी** जिसे आप [**https://mdmcert.download/**](https://mdmcert.download/) से प्राप्त करने का प्रयास कर सकते हैं। और Apple उपकरणों के लिए अपने खुद के MDM को चलाने के लिए आप [**MicroMDM**](https://github.com/micromdm/micromdm) का उपयोग कर सकते हैं।

हालांकि, एक नामित खाते द्वारा हस्ताक्षरित एक एप्लिकेशन स्थापित करने के लिए आपको अभी भी इसकी आवश्यकता होती है... हालांकि, MDM नामांकन के बाद, **उपकरण MDM का SSL प्रमाणपत्र एक विश्वसनीय सीए रूप में जोड़ता है**, इसलिए अब आप कुछ भी हस्ताक्षर कर सकते हैं।

एक MDM में उपकरण को नामांकित करने के लिए आपको एक **`mobileconfig`** फ़ाइल को रूट के रूप में स्थापित करने की आवश्यकता होती है, जिसे एक **pkg** फ़ाइल के माध्यम से वितरित किया जा सकता है (आप इसे zip में संपीड़ित कर सकते हैं और जब आप सफारी से डाउनलोड करते हैं तो यह अनप्रेस हो जाएगा)।

**Mythic एजेंट Orthrus** इस तकनीक का उपयोग करता है।

### JAMF PRO का दुरुपयोग

JAMF कस्टम स्क्रिप्ट (सिस्टम व्यवस्थापक द्वारा विकसित स्क्रिप्ट), नेटिव पेलोड (स्थानीय खाता बनाना, EFI पासवर्ड सेट करना, फ़ाइल/प्रक्रिया मॉनिटरिंग...) और MDM (उपकरण कॉन्फ़िगरेशन, उपकरण प्रमाणपत्र...) चला सकता है।

#### JAMF स्वयं-नामांकन

यदि उनमें **स्वयं-नामांकन सक्षम है**, तो `https://<company-name>.jamfcloud.com/enroll/` जैसी पेज पर जाएं और देखें कि क्या उनमें **प्रमाणिका प्राप्त करने के लिए प्रमाणों की आवश्यकता होती है**।

आप [**JamfSniper.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfSniper.py) स्क्रिप्ट का उपयोग करके पासवर्ड स्प्रे करने की हमला कर सकते हैं।

इसके अलावा, उचित प्रमाणिका प्राप्त करने के बाद आप अन्य उपयोगकर्ता नामों को ब्रूट-फ़ोर
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

तो, एक हमलावर्धी यह कर सकता है की वह एक दुष्प्रभावी पैकेज (`pkg`) ड्रॉप करे जो इंस्टॉल होते समय इस फ़ाइल को **ओवरराइट कर देता है** और अब JAMF को C2 के रूप में उपयोग करने के लिए एक Mythic C2 सुनने वाले Typhon एजेंट के URL को सेट कर सकता है।

{% code overflow="wrap" %}
```bash
# After changing the URL you could wait for it to be reloaded or execute:
sudo jamf policy -id 0

# TODO: There is an ID, maybe it's possible to have the real jamf connection and another one to the C2
```
{% endcode %}

#### JAMF अनुकरण

एक उपकरण और JMF के बीच **संचार का अनुकरण** करने के लिए आपको निम्नलिखित चीजें चाहिए:

* उपकरण का **UUID**: `ioreg -d2 -c IOPlatformExpertDevice | awk -F" '/IOPlatformUUID/{print $(NF-1)}'`
* **JAMF keychain**: `/Library/Application\ Support/Jamf/JAMF.keychain` जिसमें उपकरण प्रमाणपत्र होता है

इस जानकारी के साथ, **चोरी की गई** हार्डवेयर **UUID** और **SIP अक्षम** के साथ एक VM **बनाएं**, **JAMF keychain** को **छोड़ें**, Jamf **एजेंट** को **हुक** करें और इसकी जानकारी चुरा लें।

#### रहस्यों की चोरी

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>a</p></figcaption></figure>

आप यह भी मॉनिटर कर सकते हैं `/Library/Application Support/Jamf/tmp/` को जहां **कस्टम स्क्रिप्ट** व्यवस्थापक जम्फ के माध्यम से निष्पादित करना चाहते हैं, क्योंकि वे यहां **रखे जाते हैं, निष्पादित किए जाते हैं और हटा दिए जाते हैं**। इन स्क्रिप्ट में **प्रमाणपत्र** हो सकते हैं।

हालांकि, **प्रमाणपत्र** इन स्क्रिप्ट के **पैरामीटर** के रूप में पास किए जा सकते हैं, इसलिए आपको `ps aux | grep -i jamf` (बिना रूट होने के भी) मॉनिटर करने की आवश्यकता होगी।

स्क्रिप्ट [**JamfExplorer.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfExplorer.py) नए फ़ाइलों के जोड़े जाने और नए प्रक्रिया तर्कों के लिए सुन सकता है।

### macOS दूरस्थ पहुँच

और भी **MacOS** "विशेष" **नेटवर्क** **प्रोटोकॉल** के बारे में:

{% content-ref url="../macos-security-and-privilege-escalation/macos-protocols.md" %}
[macos-protocols.md](../macos-security-and-privilege-escalation/macos-protocols.md)
{% endcontent-ref %}

## सक्रिय निर्देशिका

कुछ मौकों पर आपको पता चलेगा कि **MacOS कंप्यूटर एक AD से जुड़ा हुआ है**। इस स्थिति में आपको निर्देशिका को जांचने की कोशिश करनी चाहिए जैसा कि आप इसे उपयोग कर रहे हैं। निम्नलिखित पृष्ठों में कुछ **मदद** मिलेगी:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/active-directory-methodology/" %}
[active-directory-methodology](../../windows-hardening/active-directory-methodology/)
{% endcontent-ref %}

{% content-ref url="../../network-services-pentesting/pentesting-kerberos-88/" %}
[pentesting-kerberos-88](../../network-services-pentesting/pentesting-kerberos-88/)
{% endcontent-ref %}

कुछ **स्थानीय MacOS टूल** जो आपकी मदद कर सकते हैं है `dscl`:
```bash
dscl "/Active Directory/[Domain]/All Domains" ls /
```
इसके अलावा, MacOS के लिए कुछ टूल तैयार किए गए हैं जो स्वचालित रूप से AD की जांच करते हैं और kerberos के साथ खेलते हैं:

* [**Machound**](https://github.com/XMCyber/MacHound): MacHound एक Bloodhound ऑडिटिंग टूल का एक विस्तार है जो MacOS होस्ट पर Active Directory संबंधों को संग्रहीत और अवशोषित करने की अनुमति देता है।
* [**Bifrost**](https://github.com/its-a-feature/bifrost): Bifrost एक Objective-C परियोजना है जो macOS पर Heimdal krb5 APIs के साथ संवाद करने के लिए डिज़ाइन की गई है। परियोजना का उद्देश्य, लक्ष्य को पूरा करने के लिए है, जो macOS उपकरणों पर Kerberos के आसनी से सुरक्षा परीक्षण को सक्षम करता है, निर्देशिका में किसी अन्य फ्रेमवर्क या पैकेज की आवश्यकता नहीं होती है।
* [**Orchard**](https://github.com/its-a-feature/Orchard): Active Directory जाँच करने के लिए JavaScript for Automation (JXA) टूल।

### डोमेन जानकारी
```bash
echo show com.apple.opendirectoryd.ActiveDirectory | scutil
```
### उपयोगकर्ता

MacOS के तीन प्रकार के उपयोगकर्ता होते हैं:

* **स्थानीय उपयोगकर्ता** - स्थानीय ओपन डायरेक्टरी सेवा द्वारा प्रबंधित होते हैं, वे किसी भी तरीके से सक्रिय नहीं होते हैं।
* **नेटवर्क उपयोगकर्ता** - सक्रिय निरंतर नेटवर्क उपयोगकर्ता जो प्रमाणित करने के लिए DC सर्वर से कनेक्ट होने की आवश्यकता होती है।
* **मोबाइल उपयोगकर्ता** - स्थानीय बैकअप के साथ सक्रिय निरंतर उपयोगकर्ता जिनके पास क्रेडेंशियल और फ़ाइलों के लिए स्थानीय बैकअप होता है।

स्थानीय उपयोगकर्ता और समूहों के बारे में स्थानीय जानकारी _/var/db/dslocal/nodes/Default_ फ़ोल्डर में संग्रहीत होती है।
उदाहरण के लिए, _mark_ नामक उपयोगकर्ता की जानकारी _/var/db/dslocal/nodes/Default/users/mark.plist_ में संग्रहीत होती है और _admin_ समूह की जानकारी _/var/db/dslocal/nodes/Default/groups/admin.plist_ में होती है।

**MacHound** ब्लडहाउंड डेटाबेस में तीन नए एजेज जोड़ता है:

* **CanSSH** - होस्ट पर SSH करने की अनुमति देने वाला एंटिटी
* **CanVNC** - होस्ट पर VNC करने की अनुमति देने वाला एंटिटी
* **CanAE** - होस्ट पर AppleEvent स्क्रिप्ट को निष्पादित करने की अनुमति देने वाला एंटिटी
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
अधिक जानकारी के लिए [https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/](https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/) पर जाएं।

## कीचेन तक पहुंचना

कीचेन में संभावित रूप से संवेदनशील जानकारी होती है जिसे बिना किसी प्रॉम्प्ट को उत्पन्न किए तकनीकी रूप से पहुंचा जा सकता है, जो एक लाल टीम अभ्यास में आगे बढ़ने में मदद कर सकता है:

{% content-ref url="macos-keychain.md" %}
[macos-keychain.md](macos-keychain.md)
{% endcontent-ref %}

## बाहरी सेवाएं

MacOS रेड टीमिंग एक सामान्य Windows रेड टीमिंग से अलग होती है क्योंकि आमतौर पर **MacOS कई बाहरी प्लेटफॉर्मों के साथ सीधे एकीकृत होता है**। MacOS का एक सामान्य कॉन्फ़िगरेशन कंप्यूटर तक पहुंचने के लिए **OneLogin सिंक्रनाइज़ड क्रेडेंशियल का उपयोग करना है, और OneLogin के माध्यम से कई बाहरी सेवाओं** (जैसे github, aws...) तक पहुंचना है:

![](<../../.gitbook/assets/image (563).png>)

## विविध रेड टीम तकनीक

### सफारी

जब सफारी में एक फ़ाइल डाउनलोड की जाती है, तो यदि यह एक "सुरक्षित" फ़ाइल है, तो यह **स्वचालित रूप से खोल दी जाएगी**। इसलिए उदाहरण के लिए, यदि आप **एक zip फ़ाइल डाउनलोड करते हैं**, तो यह स्वचालित रूप से डीकंप्रेस हो जाएगी:

<figure><img src="../../.gitbook/assets/image (12) (3).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**https://www.youtube.com/watch?v=IiMladUbL6E**](https://www.youtube.com/watch?v=IiMladUbL6E)
* [**https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6**](https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6)
* [**https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0**](https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0)
* [**Come to the Dark Side, We Have Apples: Turning macOS Management Evil**](https://www.youtube.com/watch?v=pOQOh07eMxY)
* [**OBTS v3.0: "An Attackers Perspective on Jamf Configurations" - Luke Roberts / Calum Hall**](https://www.youtube.com/watch?v=ju1IYWUv4ZA)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना चाहते हैं? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन की।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **[💬](https://emojipedia.org/speech-balloon/) [Discord समूह](https://discord.gg/hRep4RUj7f) या [telegram समूह](https://t.me/peass) में शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs के माध्यम से** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके**।

</details>
