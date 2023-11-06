# macOS सुरक्षा सुरक्षा

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो**? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की पहुंच** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा संग्रह विशेष [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## गेटकीपर

गेटकीपर आमतौर पर **क्वारंटीन + गेटकीपर + XProtect** के संयोजन को संदर्भित करने के लिए उपयोग किया जाता है, जो 3 macOS सुरक्षा मॉड्यूल हैं जो **उपयोगकर्ताओं को रोकने का प्रयास करेंगे** पोटेंशियली हानिकारक सॉफ़्टवेयर को डाउनलोड करने से।

अधिक जानकारी के लिए:

{% content-ref url="macos-gatekeeper.md" %}
[macos-gatekeeper.md](macos-gatekeeper.md)
{% endcontent-ref %}

## प्रक्रिया सीमाएं

### SIP - सिस्टम अखंडता संरक्षण

{% content-ref url="macos-sip.md" %}
[macos-sip.md](macos-sip.md)
{% endcontent-ref %}

### सैंडबॉक्स

MacOS सैंडबॉक्स **सैंडबॉक्स प्रोफ़ाइल में निर्दिष्ट अनुमतियों के साथ चल रहे ऐप्लिकेशन्स को सीमित करता है**। इससे सुनिश्चित होता है कि **ऐप्लिकेशन केवल उम्मीदित संसाधनों तक ही पहुंचेगा**।

{% content-ref url="macos-sandbox/" %}
[macos-sandbox](macos-sandbox/)
{% endcontent-ref %}

### TCC - **पारदर्शिता, सहमति और नियंत्रण**

**TCC (पारदर्शिता, सहमति और नियंत्रण)** macOS में एक तंत्र है जो **निश्चित सुविधाओं तक ऐप्लिकेशन पहुंच को सीमित और नियंत्रित करने** के लिए है, आमतौर पर एक गोपनीयता के दृष्टिकोण से। इसमें स्थान सेवाएं, संपर्क, फ़ोटो, माइक्रोफ़ोन, कैमरा, पहुंचियों, पूर्ण डिस्क पहुंच और बहुत कुछ जैसी चीजें शामिल हो सकती हैं।

{% content-ref url="macos-tcc/" %}
[macos-tcc](macos-tcc/)
{% endcontent-ref %}

### लॉन्च/पर्यावरण सीमाएं और विश्वास कैश

macOS में लॉन्च सीमाएं प्रक्रिया प्रारंभ करने के लिए एक सुरक्षा सुविधा है जो **कौन प्रक्रिया प्रारंभ कर सकती है**, **कैसे** और **कहां से** परिभाषित करके प्रक्रिया को नियंत्रित करती है। macOS Ventura में प्रस्तुत किए गए, यह सिस्टम बाइनरी को एक **विश्वास कैश** के भीतर नियम श्रेणियों में वर्गीकृत करता है। प्रत्येक निष्पादनीय बाइनरी के लिए इसके **नियम** होते हैं, जिनमें इसके **प्रारंभ**, **माता-पिता**, और **जिम्मेदार** सीमाएं शामिल होती हैं। macOS Sonoma में तृतीय-पक्ष ऐप्स के लिए **पर्यावरण** सीमाएं के रूप में विस्तारित किए गए, ये सुविधाएं प्रक्रिया प्रारंभ की स्थितियों को नियंत्रित करके संभावित सिस्टम उत्पीड़नों को कम करने में मदद करती हैं।

{% content-ref url="macos-launch-environment-constraints.md" %}
[macos-launch-environment-constraints.md](macos-launch-environment-constraints.md)
{% endcontent-ref %}

## MRT - मैलवेयर हटाने वाला उपकरण

मैलवेयर हटाने वाला उपकरण (MRT) macOS की सुरक्षा बुनियाद का एक और हिस्सा है। जैसा कि नाम से पता चलता है, MRT का मुख
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
### गणना

यह संभव है कि आप Apple cli उपकरण के द्वारा कॉन्फ़िगर किए गए सभी पृष्ठभूमि आइटमों की गणना कर सकते हैं:
```bash
# The tool will always ask for the users password
sfltool dumpbtm
```
इसके अलावा, [**DumpBTM**](https://github.com/objective-see/DumpBTM) के साथ भी यह जानकारी सूचीबद्ध करना संभव है।
```bash
# You need to grant the Terminal Full Disk Access for this to work
chmod +x dumpBTM
xattr -rc dumpBTM # Remove quarantine attr
./dumpBTM
```
यह जानकारी **`/private/var/db/com.apple.backgroundtaskmanagement/BackgroundItems-v4.btm`** में संग्रहीत की जा रही है और टर्मिनल को FDA की आवश्यकता होती है।

### BTM के साथ खिलवाड़

जब एक नई स्थायित्व मिलती है, तो एक प्रकार की घटना **`ES_EVENT_TYPE_NOTIFY_BTM_LAUNCH_ITEM_ADD`** को भेजी जाती है। इसलिए, इस घटना को भेजने से रोकने या एजेंट को उपयोगकर्ता को सतर्क करने से किसी भी तरीके से एक हमलावर को BTM को _**छलना**_ में मदद मिलेगी।

* **डेटाबेस को रीसेट करना**: निम्नलिखित कमांड चलाने से डेटाबेस रीसेट हो जाएगा (इसे मूल से पुनः निर्माण करना चाहिए), हालांकि, किसी कारण से, इसके बाद, **सिस्टम को रिबूट किए बिना कोई नई स्थायित्व सतर्क नहीं की जाएगी**।
* **रूट** की आवश्यकता होती है।
```bash
# Reset the database
sfltool resettbtm
```
* **एजेंट को रोकें**: एजेंट को रोकना संभव है ताकि जब नई पहचानें मिलती हैं, वह उपयोगकर्ता को **चेतावनी न दे**।
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
* **बग**: यदि **परिस्थिति जो परिस्थिति बनाई है तुरंत उपस्थित होती है**, तो डीमन इसके बारे में **जानकारी प्राप्त करने** का प्रयास करेगा, **विफल हो जाएगा**, और नई चीज परिस्थित हो रही है इसकी घटना भेजने में **सक्षम नहीं होगा**।

BTM के बारे में **अधिक जानकारी और संदर्भ**:

* [https://youtu.be/9hjUmT031tc?t=26481](https://youtu.be/9hjUmT031tc?t=26481)
* [https://www.patreon.com/posts/new-developer-77420730?l=fr](https://www.patreon.com/posts/new-developer-77420730?l=fr)
* [https://support.apple.com/en-gb/guide/deployment/depdca572563/web](https://support.apple.com/en-gb/guide/deployment/depdca572563/web)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
