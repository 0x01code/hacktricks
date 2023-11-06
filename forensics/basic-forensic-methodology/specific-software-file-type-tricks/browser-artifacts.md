# ब्राउज़र आर्टिफैक्ट्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **वर्कफ़्लो** को आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ब्राउज़र आर्टिफैक्ट्स <a href="#3def" id="3def"></a>

जब हम ब्राउज़र आर्टिफैक्ट्स के बारे में बात करते हैं, तो हम नेविगेशन इतिहास, बुकमार्क्स, डाउनलोड की गई फ़ाइलों की सूची, कैश डेटा आदि के बारे में बात करते हैं।

ये आर्टिफैक्ट्स ऑपरेटिंग सिस्टम के निर्दिष्ट फ़ोल्डर में संग्रहीत फ़ाइलें होती हैं।

प्रत्येक ब्राउज़र अन्य ब्राउज़रों की तुलना में अलग स्थान पर अपनी फ़ाइलें संग्रहीत करता है और उनके पास अलग-अलग नाम होते हैं, लेकिन वे सभी (अधिकांश समय) समान प्रकार के डेटा (आर्टिफैक्ट्स) संग्रहीत करते हैं।

चलो देखते हैं कि ब्राउज़रों द्वारा संग्रहीत किए जाने वाले सबसे सामान्य आर्टिफैक्ट्स क्या हैं।

* **नेविगेशन इतिहास:** उपयोगकर्ता के नेविगेशन इतिहास के बारे में डेटा संग्रहीत करता है। उदाहरण के लिए, उपयोगकर्ता ने किसी खतरनाक साइट का दौरा किया है या नहीं, इसे ट्रैक करने के लिए इस्तेमाल किया जा सकता है।
* **ऑटोकंप्लीट डेटा:** यह डेटा है जिसे ब्राउज़र सुझाव देता है आपके द्वारा सबसे अधिक खोज करने पर आधारित। इसे नेविगेशन इतिहास के साथ मिलाकर अधिक जानकारी प्राप्त करने के लिए इस्तेमाल किया जा सकता है।
* **बुकमार्क्स:** स्वयं समझदारी।
* **एक्सटेंशन और ऐड-ऑन्स:** स्वयं समझदारी।
* **कैश:** वेबसाइट पर नेविगेशन करते समय, ब्राउज़
* _**downloads.sqlite**_ : पुराना डाउनलोड डेटाबेस (अब यह places.sqlite में है)
* _**thumbnails/**_ : थंबनेल्स
* _**logins.json**_ : एन्क्रिप्टेड उपयोगकर्ता नाम और पासवर्ड
* **ब्राउज़र के अंतर्जालीय एंटी-फिशिंग:** `grep 'browser.safebrowsing' ~/Library/Application Support/Firefox/Profiles/*/prefs.js`
* यदि सुरक्षित खोज सेटिंग्स अक्षम कर दी गई हैं, तो "safebrowsing.malware.enabled" और "phishing.enabled" को false लौटाएगा
* _**key4.db**_ या _**key3.db**_ : मास्टर की?

मास्टर पासवर्ड को डिक्रिप्ट करने के लिए, आप [https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt) का उपयोग कर सकते हैं।
निम्नलिखित स्क्रिप्ट और कॉल के साथ, आप ब्रूट फोर्स करने के लिए एक पासवर्ड फ़ाइल निर्दिष्ट कर सकते हैं:

{% code title="brute.sh" %}
```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```
{% endcode %}

![](<../../../.gitbook/assets/image (417).png>)

## Google Chrome

Google Chrome उपयोगकर्ता के घर के अंदर प्रोफ़ाइल बनाता है _**\~/.config/google-chrome/**_ (Linux), _**C:\Users\XXX\AppData\Local\Google\Chrome\User Data\\**_ (Windows), या \_**/Users/$USER/Library/Application Support/Google/Chrome/** \_ (MacOS) में।\
ज्यादातर जानकारी _**Default/**_ या _**ChromeDefaultData/**_ फ़ोल्डर में सहेजी जाएगी जो पहले दिए गए पथों के अंदर होते हैं। यहां आप निम्नलिखित दिलचस्प फ़ाइलें पा सकते हैं:

* _**History**_: URLs, downloads और खोजी गई कीवर्ड। Windows में, आप इस्तेमाल कर सकते हैं उपकरण [ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html) इतिहास पढ़ने के लिए। "Transition Type" स्तंभ का अर्थ है:
* Link: उपयोगकर्ता ने लिंक पर क्लिक किया
* Typed: URL लिखा गया था
* Auto Bookmark
* Auto Subframe: जोड़ें
* Start page: होम पेज
* Form Submit: एक फ़ॉर्म भरा और भेजा गया
* Reloaded
* _**Cookies**_: कुकीज़। [ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html) का उपयोग करके कुकीज़ की जांच की जा सकती है।
* _**Cache**_: कैश। Windows में, आप इस्तेमाल कर सकते हैं उपकरण [ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html) कैश की जांच करने के लिए।
* _**Bookmarks**_: बुकमार्क्स
* _**Web Data**_: फ़ॉर्म इतिहास
* _**Favicons**_: फ़ेविकॉन्स
* _**Login Data**_: लॉगिन जानकारी (उपयोगकर्ता नाम, पासवर्ड...)
* _**Current Session**_ और _**Current Tabs**_: वर्तमान सत्र डेटा और वर्तमान टैब्स
* _**Last Session**_ और _**Last Tabs**_: ये फ़ाइलें उस समय चल रही साइट्स को रखती हैं जब Chrome बंद हो गया था।
* _**Extensions**_: एक्सटेंशन और एडऑन फ़ोल्डर
* **Thumbnails** : थंबनेल्स
* **Preferences**: इस फ़ाइल में प्लगइन, एक्सटेंशन, जियोलोकेशन का उपयोग करने वाली साइटें, पॉपअप, सूचनाएँ, DNS prefetching, प्रमाणपत्र अपवाद, और बहुत कुछ जैसी अच्छी जानकारी होती है। यदि आप जांचने की कोशिश कर रहे हैं कि क्या कोई विशेष Chrome सेटिंग सक्षम थी या नहीं, तो आप शायद इसमें उस सेटिंग को पाएंगे।
* **Browser’s built-in anti-phishing:** `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`
* आप आसानी से “**safebrowsing**” के लिए grep कर सकते हैं और परिणाम में `{"enabled: true,"}` ढूंढ़ सकते हैं जो एंटी-फिशिंग और मैलवेयर सुरक्षा को सक्षम करता है।

## **SQLite DB डेटा रिकवरी**

जैसा कि आप पिछले खंडों में देख सकते हैं, च्रोम और फ़ायरफ़ॉक्स दोनों डेटा सहेजने के लिए **SQLite** डेटाबेस का उपयोग करते हैं। इस उपकरण [**sqlparse**](https://github.com/padfoot999/sqlparse) **या** [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases) का उपयोग करके **हटाए गए एंट्रीज़ को रिकवर किया जा सकता है**।

## **Internet Explorer 11**

इंटरनेट एक्सप्लोरर विभिन्न स्थानों में **डेटा** और **मेटाडेटा** संग्रहीत करता है। मेटाडेटा डेटा ढूंढ़ने की अनुमति देगा।

**मेटाडेटा** फ़ोल्डर `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` में पाया जा सकता है जहां VX V01, V16, या V24 हो सकता है।\
पिछले फ़ोल्डर में, आप V01.log फ़ाइल भी पा सकते हैं। यदि इस फ़ाइल का **संशोधित समय** और WebcacheVX.data फ़ाइल का **अलग हैं**, तो आपको शायद यह आदेश चलाने की आवश्यकता होगी `esentutl /r V01 /d` कोई संगतता संबंधित समस्याओं को **ठीक करने** के लिए।

एक बार जब यह आर्टिफैक्ट **रिकवर** हो जाए (यह एक ESE डेटाबेस है, फ़ोटोरेक इसे विकल्प एक्सचेंज डेटाबेस या EDB के साथ रिकवर कर सकता है) आप इसे खोलने के लिए प्रोग्राम [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) का उपयोग कर सकते हैं। एक बार **खोलने** के बाद, जाएं टेबल में "**Containers**" नामक।

![](<../../../.gitbook/assets/image (446).png>)

इस टेबल के अंदर, आप देख सकते हैं कि संग्रहीत जानकारी के हर हिस्से को किस अन्य टेबल या कंटेनर में सहेजा जाता है। इसके बाद, आप ब्राउज़र्स द
### डाउनलोड

#### **मेटाडेटा**

[ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) टूल की जांच करके आप डाउनलोड के मेटाडेटा कंटेनर को ढूंढ सकते हैं:

![](<../../../.gitbook/assets/image (445).png>)

"ResponseHeaders" स्तंभ की जानकारी प्राप्त करके आप उस जानकारी को हेक्स से ट्रांसफ़ॉर्म कर सकते हैं और डाउनलोड की गई फ़ाइल का URL, फ़ाइल प्रकार और स्थान प्राप्त कर सकते हैं।

#### फ़ाइलें

_**%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory**_ पथ में देखें

### **इतिहास**

[BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) टूल का उपयोग इतिहास पढ़ने के लिए किया जा सकता है। लेकिन पहले, आपको उन्नत विकल्पों में ब्राउज़र और निकाले गए इतिहास फ़ाइलों के स्थान को दर्ज करना होगा।

#### **मेटाडेटा**

* ModifiedTime: URL पहली बार मिला
* AccessedTime: अंतिम बार
* AccessCount: बार तक पहुँचे गए

#### **फ़ाइलें**

_**userprofile%\Appdata\Local\Microsoft\Windows\History\History.IE5**_ और _**userprofile%\Appdata\Local\Microsoft\Windows\History\Low\History.IE5**_ में खोजें

### **टाइप किए गए URL**

यह जानकारी रजिस्ट्री NTDUSER.DAT के अंदर खोजी जा सकती है पथ में:

* _**Software\Microsoft\InternetExplorer\TypedURLs**_
* उपयोगकर्ता द्वारा टाइप किए गए अंतिम 50 URL संग्रहीत करता है
* _**Software\Microsoft\InternetExplorer\TypedURLsTime**_
* URL को अंतिम बार टाइप किया गया था

## Microsoft Edge

Microsoft Edge आर्टिफैक्ट का विश्लेषण करने के लिए पिछले खंड (IE 11) के कैश और स्थानों के बारे में सभी **व्याख्यान** वैध रहते हैं, बस इसका अंतर है कि इस मामले में बेस लोकेशन _**%userprofile%\Appdata\Local\Packages**_ है (निम्नलिखित पथों में देखा जा सकता है):

* प्रोफ़ाइल पथ: _**C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge\_XXX\AC**_
* इतिहास, कुकीज़ और डाउनलोड: _**C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat**_
* सेटिंग्स, बुकमार्क्स और रीडिंग सूची: _**C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge\_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb**_
* कैश: _**C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge\_XXX\AC#!XXX\MicrosoftEdge\Cache**_
* अंतिम सक्रिय सत्र: _**C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge\_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active**_

## **Safari**

डेटाबेस `/Users/$User/Library/Safari` में पाए जा सकते हैं

* **History.db**: टेबल `history_visits` और `history_items` में इतिहास और टाइमस्टैम्प के बारे में जानकारी होती है।
* `sqlite3 ~/Library/Safari/History.db "SELECT h.visit_time, i.url FROM history_visits h INNER JOIN history_items i ON h.history_item = i.id"`
* **Downloads.plist**: डाउनलोड की गई फ़ाइलों के बारे में जानकारी होती है।
* **Book-marks.plist**: बुकमार्क किए गए URL।
* **TopSites.plist**: उपयोगकर्ता द्वारा सबसे अधिक देखी जाने वाली वेबसाइटों की सूची।
* **Extensions.plist**: पुरानी शैली की सूची प्राप्त करने के लिए Safari ब्राउज़र एक्सटेंशन।
* `plutil -p ~/Library/Safari/Extensions/Extensions.plist| grep "Bundle Directory Name" | sort --ignore-case`
* `pluginkit -mDvvv -p com.apple.Safari.extension`
* **UserNotificationPermissions.plist**: पुष सूचनाएं भेजने की अनुमति देने वाले डोमेन।
* `plutil -p ~/Library/Safari/UserNotificationPermissions.plist | grep -a3 '"Permission" => 1'`
* **LastSession.plist**: टैब जो उपयोगकर्ता ने Safari से बाहर निकलते समय खोले थे।
* `plutil -p ~/Library/Safari/LastSession.plist | grep -iv sessionstate`
* **ब्राउज़र के अंति-फिशिंग:** `defaults read com.apple.Safari WarnAboutFraudulentWebsites`
* उत्तर 1 होना चाहिए जो सेटिंग सक्रिय दिखाने के लिए है

## Opera

डेटाबेस `/Users/$USER/Library/Application Support/com.operasoftware.Opera` में पाए जा सकते हैं

Opera **ब्राउज़र इतिहास और डाउनलोड डेटा को Google Chrome के समान फ़ॉर्मेट में संग्रहीत करता है**। इसका फ़ाइल नाम और टेबल नाम दोनों पर लागू होता है।

* **ब्राउज़र के अंति-फिशिंग:** `grep --color 'fraud_protection_enabled' ~/Library/Application Support/com.operasoftware.Opera/Preferences`
* **fraud\_protection\_enabled** को **true** होना चाहिए

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उप
