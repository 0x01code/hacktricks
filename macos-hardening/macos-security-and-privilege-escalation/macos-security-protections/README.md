# macOS सुरक्षा संरक्षण

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें**.
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## Gatekeeper

Gatekeeper आमतौर पर **Quarantine + Gatekeeper + XProtect** के संयोजन को संदर्भित करता है, 3 macOS सुरक्षा मॉड्यूल जो **डाउनलोड किए गए संभावित रूप से हानिकारक सॉफ्टवेयर को निष्पादित करने से उपयोगकर्ताओं को रोकने का प्रयास करेंगे**.

अधिक जानकारी के लिए:

{% content-ref url="macos-gatekeeper.md" %}
[macos-gatekeeper.md](macos-gatekeeper.md)
{% endcontent-ref %}

## प्रक्रियाओं की सीमाएँ

### SIP - सिस्टम इंटीग्रिटी प्रोटेक्शन

{% content-ref url="macos-sip.md" %}
[macos-sip.md](macos-sip.md)
{% endcontent-ref %}

### Sandbox

MacOS Sandbox **एप्लिकेशनों को सीमित करता है** जो सैंडबॉक्स के अंदर चल रहे होते हैं, उन्हें केवल **सैंडबॉक्स प्रोफाइल में निर्दिष्ट अनुमत क्रियाओं तक ही सीमित रखता है** जिसके साथ ऐप चल रहा है। यह सुनिश्चित करने में मदद करता है कि **एप्लिकेशन केवल अपेक्षित संसाधनों तक ही पहुँच रहा होगा**.

{% content-ref url="macos-sandbox/" %}
[macos-sandbox](macos-sandbox/)
{% endcontent-ref %}

### TCC - **पारदर्शिता, सहमति, और नियंत्रण**

**TCC (पारदर्शिता, सहमति, और नियंत्रण)** macOS में एक तंत्र है जो **कुछ विशेषताओं तक एप्लिकेशन पहुँच को सीमित और नियंत्रित करता है**, आमतौर पर गोपनीयता के दृष्टिकोण से। इसमें लोकेशन सेवाएं, संपर्क, फोटो, माइक्रोफोन, कैमरा, पहुँच, पूर्ण डिस्क पहुँच, और बहुत कुछ शामिल हो सकता है।

{% content-ref url="macos-tcc/" %}
[macos-tcc](macos-tcc/)
{% endcontent-ref %}

### लॉन्च/पर्यावरण सीमाएँ & ट्रस्ट कैश

macOS में लॉन्च सीमाएँ एक सुरक्षा विशेषता है जो **प्रक्रिया प्रारंभ करने को विनियमित करती है** यह परिभाषित करके कि **कौन** प्रक्रिया शुरू कर सकता है, **कैसे**, और **कहाँ से**। macOS Ventura में पेश की गई, यह एक **ट्रस्ट कैश** के भीतर सिस्टम बाइनरीज़ को सीमा श्रेणियों में वर्गीकृत करती है। हर निष्पादन योग्य बाइनरी के लिए **नियम** सेट होते हैं उसके **लॉन्च** के लिए, जिसमें **स्वयं**, **माता-पिता**, और **जिम्मेदार** सीमाएँ शामिल हैं। macOS Sonoma में तीसरे पक्ष के ऐप्स के लिए **पर्यावरण** सीमाएँ के रूप में विस्तारित, ये विशेषताएँ प्रक्रिया लॉन्चिंग की शर्तों को शासित करके संभावित सिस्टम शोषणों को कम करने में मदद करती हैं।

{% content-ref url="macos-launch-environment-constraints.md" %}
[macos-launch-environment-constraints.md](macos-launch-environment-constraints.md)
{% endcontent-ref %}

## MRT - मैलवेयर निकालने का उपकरण

मैलवेयर निकालने का उपकरण (MRT) macOS की सुरक्षा अवसंरचना का एक और हिस्सा है। जैसा कि नाम से पता चलता है, MRT का मुख्य कार्य **संक्रमित सिस्टमों से ज्ञात मैलवेयर को हटाना है**।

एक बार जब मैक पर मैलवेयर का पता चलता है (XProtect द्वारा या किसी अन्य माध्यम से), MRT का उपयोग स्वचालित रूप से **मैलवेयर को हटाने** के लिए किया जा सकता है। MRT पृष्ठभूमि में चुपचाप काम करता है और आमतौर पर तब चलता है जब सिस्टम अपडेट होता है या जब एक नई मैलवेयर परिभाषा डाउनलोड की जाती है (ऐसा लगता है कि MRT के पास मैलवेयर का पता लगाने के लिए नियम बाइनरी के अंदर होते हैं)।

XProtect और MRT दोनों macOS के सुरक्षा उपायों का हिस्सा हैं, लेकिन वे अलग-अलग कार्य करते हैं:

* **XProtect** एक निवारक उपकरण है। यह **फाइलों की जाँच करता है जैसे ही वे डाउनलोड की जाती हैं** (कुछ एप्लिकेशनों के माध्यम से), और यदि यह किसी भी ज्ञात प्रकार के मैलवेयर का पता लगाता है, तो यह **फाइल को खुलने से रोकता है**, इस प्रकार आपके सिस्टम को पहली जगह में संक्रमित होने से रोकता है।
* **MRT**, दूसरी ओर, एक **प्रतिक्रियाशील उपकरण** है। यह सिस्टम पर मैलवेयर का पता चलने के बाद काम करता है, अपराधी सॉफ्टवेयर को हटाने और सिस्टम को साफ करने के उद्देश्य से।

MRT एप्लिकेशन **`/Library/Apple/System/Library/CoreServices/MRT.app`** में स्थित है

## पृष्ठभूमि कार्य प्रबंधन

**macOS** अब हर बार **चेतावनी देता है** जब कोई टूल एक ज्ञात **तकनीक का उपयोग करता है कोड निष्पादन को बनाए रखने के लिए** (जैसे कि लॉगिन आइटम, डेमॉन्स...), ताकि उपयोगकर्ता बेहतर जान सके **कौन सा सॉफ्टवेयर बना रहा है**।

<figure><img src="../../../.gitbook/assets/image (711).png" alt=""><figcaption></figcaption></figure>

यह `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/Resources/backgroundtaskmanagementd` में स्थित **डेमॉन** के साथ और `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Support/BackgroundTaskManagementAgent.app` में स्थित **एजेंट** के साथ चलता है।

**`backgroundtaskmanagementd`** को कैसे पता चलता है कि कुछ स्थायी फोल्डर में स्थापित है, यह **FSEvents प्राप्त करके** और उनके लिए कुछ **हैंडलर्स** बनाकर होता है।

इसके अलावा, एक plist फाइल है जिसमें **ज्ञात एप्लिकेशन** होते हैं जो अक्सर एप्पल द्वारा बनाए रखे जाते हैं, स्थित है: `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/Resources/attributions.plist`
```json
[...]
"us.zoom.ZoomDaemon" => {
"AssociatedBundleIdentifiers" => [
0 => "us.zoom.xos"
]
"Attribution" => "Zoom"
"Program" => "/Library/PrivilegedHelperTools/us.zoom.ZoomDaemon"
"ProgramArguments" => [
0 => "/Library/PrivilegedHelperTools/us.zoom.ZoomDaemon"
]
"TeamIdentifier" => "BJ4HAAB9B3"
}
[...]
```
### एन्युमरेशन

Apple के cli टूल का उपयोग करके सभी कॉन्फ़िगर किए गए बैकग्राउंड आइटम्स को **सूचीबद्ध करना** संभव है:
```bash
# The tool will always ask for the users password
sfltool dumpbtm
```
इसके अलावा, इस जानकारी को [**DumpBTM**](https://github.com/objective-see/DumpBTM) के साथ सूचीबद्ध करना भी संभव है।
```bash
# You need to grant the Terminal Full Disk Access for this to work
chmod +x dumpBTM
xattr -rc dumpBTM # Remove quarantine attr
./dumpBTM
```
### BTM के साथ छेड़छाड़

जब एक नया पर्सिस्टेंस पाया जाता है तो **`ES_EVENT_TYPE_NOTIFY_BTM_LAUNCH_ITEM_ADD`** प्रकार की एक घटना होती है। इसलिए, इस **घटना** को भेजने से **रोकने** का कोई भी तरीका या **एजेंट को उपयोगकर्ता को सतर्क करने से रोकना** एक हमलावर को BTM को _**बायपास**_ करने में मदद करेगा।

* **डेटाबेस को रीसेट करना**: निम्नलिखित कमांड चलाने से डेटाबेस रीसेट हो जाएगा (इसे जमीन से फिर से बनाना चाहिए), हालांकि, किसी कारण से, इसे चलाने के बाद, **सिस्टम को रिबूट किए जाने तक कोई नया पर्सिस्टेंस सतर्क नहीं किया जाएगा**।
* **root** की आवश्यकता है।
```bash
# Reset the database
sfltool resettbtm
```
* **एजेंट को रोकें**: एजेंट को एक रुकने का सिग्नल भेजना संभव है ताकि जब नए डिटेक्शन पाए जाएं तो वह **उपयोगकर्ता को सतर्क न करे**।
```bash
# Get PID
pgrep BackgroundTaskManagementAgent
1011

# Stop it
kill -SIGSTOP 1011

# Check it's stopped (a T means it's stopped)
ps -o state 1011
T
```
* **बग**: यदि **प्रक्रिया जिसने पर्सिस्टेंस बनाया है वह तुरंत बाद में तेजी से समाप्त हो जाती है**, तो डेमन प्रक्रिया के बारे में **जानकारी प्राप्त करने** की कोशिश करेगा, **असफल हो जाएगा**, और **नई चीज के पर्सिस्ट होने की घटना भेजने में सक्षम नहीं होगा**।

संदर्भ और **BTM के बारे में अधिक जानकारी**:

* [https://youtu.be/9hjUmT031tc?t=26481](https://youtu.be/9hjUmT031tc?t=26481)
* [https://www.patreon.com/posts/new-developer-77420730?l=fr](https://www.patreon.com/posts/new-developer-77420730?l=fr)
* [https://support.apple.com/en-gb/guide/deployment/depdca572563/web](https://support.apple.com/en-gb/guide/deployment/depdca572563/web)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
