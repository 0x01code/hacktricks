# macOS सुरक्षा सुरक्षा

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert) के साथ</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **PR जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## गेटकीपर

गेटकीपर आम तौर पर **क्वारंटाइन + गेटकीपर + एक्सप्रोटेक्ट** का संयोजन करने के लिए उपयोग किया जाता है, जो macOS सुरक्षा मॉड्यूल हैं जो **उपयोगकर्ताओं को निर्दिष्ट किए गए संभावित हानिकारक सॉफ्टवेयर को निषेधित करने की कोशिश करेंगे**।

अधिक जानकारी:

{% content-ref url="macos-gatekeeper.md" %}
[macos-gatekeeper.md](macos-gatekeeper.md)
{% endcontent-ref %}

## प्रक्रिया सीमाएं

### SIP - सिस्टम अखंडता सुरक्षा

{% content-ref url="macos-sip.md" %}
[macos-sip.md](macos-sip.md)
{% endcontent-ref %}

### सैंडबॉक्स

MacOS सैंडबॉक्स **ऐप्लिकेशनों की सीमा लगाता है** जो सैंडबॉक्स प्रोफ़ाइल के साथ चल रहा है। इससे यह सुनिश्चित होता है कि **ऐप्लिकेशन केवल अपेक्षित संसाधनों तक ही पहुंचेगा**।

{% content-ref url="macos-sandbox/" %}
[macos-sandbox](macos-sandbox/)
{% endcontent-ref %}

### TCC - **पारदर्शिता, सहमति और नियंत्रण**

**TCC (पारदर्शिता, सहमति और नियंत्रण)** एक सुरक्षा ढांचा है। यह **ऐप्लिकेशनों की अनुमतियों का प्रबंधन** करने के लिए डिज़ाइन किया गया है, विशेष रूप से उनके पहुंच को नियंत्रित करके। इसमें लोकेशन सेवाएं, संपर्क, फोटो, माइक्रोफोन, कैमरा, पहुंचनीयता और पूर्ण डिस्क पहुंच जैसे तत्व शामिल हैं। TCC सुनिश्चित करता है कि ऐप्स केवल इन सुविधाओं तक पहुंच सकते हैं जब उन्होंने स्पष्ट उपयोगकर्ता सहमति प्राप्त की हो, इससे निजता और व्यक्तिगत डेटा पर नियंत्रण मजबूत होता है।

{% content-ref url="macos-tcc/" %}
[macos-tcc](macos-tcc/)
{% endcontent-ref %}

### लॉन्च/पर्यावरण सीमाएं और विश्वास कैश

macOS में लॉन्च सीमाएं एक सुरक्षा सुविधा है जो प्रक्रिया प्रारंभ करने का नियमित करने के लिए है, जिसमें **कौन प्रक्रिया प्रारंभ कर सकता है**, **कैसे**, और **कहाँ से** को परिभाषित करके। macOS Ventura में पेश किए गए, ये सिस्टम बाइनरी को एक **विश्वास कैश** के भीतर रोगण श्रेणियों में वर्गीकृत करते हैं। प्रत्येक निष्पादनीय बाइनरी के लिए उसके **लॉन्च** के लिए सेट **नियम** होते हैं, जिसमें इसके **स्वयं**, **माता-पिता**, और **जिम्मेदार** सीमाएं शामिल हैं। macOS Sonoma में तीसरे पक्ष के ऐप्स के लिए **पर्यावरण** सीमाएं के रूप में विस्तारित किए गए, ये सुविधाएं प्रक्रिया प्रारंभ करने की स्थितियों को नियंत्रित करके संभावित सिस्टम शोषणों को कम करने में मदद करती हैं।

{% content-ref url="macos-launch-environment-constraints.md" %}
[macos-launch-environment-constraints.md](macos-launch-environment-constraints.md)
{% endcontent-ref %}

## MRT - मैलवेयर हटाने वाला टूल

मैलवेयर हटाने वाला टूल (MRT) macOS की सुरक्षा ढांचा का एक और हिस्सा है। जैसा कि नाम सुझाता है, MRT का मुख्य कार्य है **प्रश्नित सिस्टम से ज्ञात मैलवेयर को हटाना**।

जब किसी मैक पर मैलवेयर का पता लगाया जाता है (या तो XProtect द्वारा या किसी अन्य तरीके से), तो MRT का उपयोग करके मैलवेयर को स्वचालित रूप से **हटाया जा सकता है**। MRT पिछले प्लान में चुपचाप काम करता है और सामान्यत: जब सिस्टम अपडेट किया जाता है या जब एक नया मैलवेयर परिभाषा डाउनलोड की जाती है (ऐसा लगता है कि MRT के नियम जिनका उपयोग मैलवेयर को पहचानने के लिए हैं, वे बाइनरी के भीतर हैं)।

जबकि XProtect और MRT दोनों macOS की सुरक्षा उपायों का हिस्सा हैं, वे अलग-अलग कार्य करते हैं:

* **XProtect** एक निवारक उपकरण है। यह **फ़ाइलों की जांच करता है जैसे ही वे डाउनलोड होती हैं** (कुछ ऐप्लिकेशन के माध्यम से), और यदि यह किसी भी ज्ञात प्रकार के मैलवेयर को पता लगाता है, तो यह **फ़ाइल को खोलने से रोकता है**, इससे पहले ही मैलवेयर को आपके सिस्टम में प्रवेश करने से रोकता है।
* **MRT**, दूसरी ओर, एक **प्रतिक्रियात्मक उपकरण** है। यह सिस्टम पर मैलवेयर का पता लगाने के बाद काम करता है, जिसका उद्देश्य दोषी सॉफ़्टवेयर को हटाकर सिस्टम को साफ करना है।

MRT एप्लिकेशन **`/Library/Apple/System/Library/CoreServices/MRT.app`** में स्थित है।

## पृष्ठभूमि कार्य प्रबंधन

**macOS** अब हर बार **चेतावनी देता है** जब भी कोई टूल एक प्रसिद्ध **तकनीक का उपयोग करता है कोड निष्ठांतरण के लिए** (जैसे लॉगिन आइटम, डेमन...), ताकि उपयोगकर्ता को बेहतर पता चले **कौन सॉफ़्टवेयर स्थिर रख रहा है**।

<figure><img src="../../../.gitbook/assets/image (711).png" alt=""><figcaption></figcaption></figure>

यह एक **डेमन** में चलता है जो `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/Resources/backgroundtaskmanagementd` में स्थित है और **एजेंट** `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Support/BackgroundTaskManagementAgent.app` में है।

**`backgroundtaskmanagementd`** को पता चलता है कि कुछ स्थायी फ़ोल्डर में कुछ स्थापित है कैसे **FSEvents प्राप्त करके** और उनके लिए कुछ **हैंडलर** बनाकर।

इसके अतिरिक्त, एक plist फ़ाइल है जिसमें एप्पल द्वारा बनाए गए **प्रसिद्ध ऐप्लिकेशन** हैं जो निरंतर बने रहते हैं जो स्थित है: `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/Resources/attributions.plist`
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
### जांच

आप Apple cli उपकरण चला कर सभी कॉन्फ़िगर किए गए पृष्ठभूमि आइटम को **जांचना** संभव है:
```bash
# The tool will always ask for the users password
sfltool dumpbtm
```
इस जानकारी को [**DumpBTM**](https://github.com/objective-see/DumpBTM) के साथ सूचीबद्ध करना भी संभव है।
```bash
# You need to grant the Terminal Full Disk Access for this to work
chmod +x dumpBTM
xattr -rc dumpBTM # Remove quarantine attr
./dumpBTM
```
यह जानकारी **`/private/var/db/com.apple.backgroundtaskmanagement/BackgroundItems-v4.btm`** में स्टोर की जा रही है और टर्मिनल को FDA की आवश्यकता है।

### BTM के साथ खिलवाड़

जब एक नई स्थायित्व मिलता है, तो एक प्रकार की घटना होती है **`ES_EVENT_TYPE_NOTIFY_BTM_LAUNCH_ITEM_ADD`**। इसलिए, इस **घटना** को भेजने से रोकने या उपयोगकर्ता को सूचित करने वाले **एजेंट** को रोकने का कोई तरीका एटैकर को BTM को _**बायपास**_ करने में मदद करेगा।

* **डेटाबेस को रीसेट करना**: निम्नलिखित कमांड चलाने से डेटाबेस रीसेट हो जाएगा (इसे मूल से पुनः निर्माण करना चाहिए), हालांकि, किसी कारण से, इसके बाद इसे चलाने के बाद **कोई नई स्थायित्व सूचित नहीं किया जाएगा जब तक सिस्टम पुनरारंभ नहीं होता है**।
* **रूट** की आवश्यकता है।
```bash
# Reset the database
sfltool resettbtm
```
* **एजेंट को रोकें**: एजेंट को रोकना संभव है ताकि जब नई पहचानें मिलती हैं तो उपयोगकर्ता को सूचित न करें।
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
* **बग**: अगर **परिस्थिति को बनाने वाली प्रक्रिया** तुरंत उसके बाद में मौजूद होती है, तो डेमन उसके बारे में **जानकारी प्राप्त करने** की कोशिश करेगा, **विफल** हो जाएगा, और **नयी चीज को स्थायी रूप से बना रहे हैं** इस घटना को भेजने में **सक्षम नहीं होगा**।

BTM के बारे में संदर्भ और **अधिक जानकारी**:

* [https://youtu.be/9hjUmT031tc?t=26481](https://youtu.be/9hjUmT031tc?t=26481)
* [https://www.patreon.com/posts/new-developer-77420730?l=fr](https://www.patreon.com/posts/new-developer-77420730?l=fr)
* [https://support.apple.com/en-gb/guide/deployment/depdca572563/web](https://support.apple.com/en-gb/guide/deployment/depdca572563/web)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **पीआर जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
