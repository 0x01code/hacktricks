# ब्राउज़र आर्टिफैक्ट्स

<details>

<summary><strong> AWS हैकिंग सीखें शून्य से लेकर हीरो तक </strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें PRs जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को बिल्ड और ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं.\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ब्राउज़र्स आर्टिफैक्ट्स <a href="#3def" id="3def"></a>

जब हम ब्राउज़र आर्टिफैक्ट्स की बात करते हैं, हम नेविगेशन हिस्ट्री, बुकमार्क्स, डाउनलोड की गई फाइलों की सूची, कैश डेटा आदि की बात करते हैं.

ये आर्टिफैक्ट्स ऑपरेटिंग सिस्टम में विशिष्ट फोल्डर्स के अंदर संग्रहीत फाइलें होती हैं.

प्रत्येक ब्राउज़र अपनी फाइलों को अन्य ब्राउज़रों से अलग स्थान पर संग्रहीत करता है और उनके नाम भी अलग होते हैं, लेकिन वे सभी (अधिकतर समय) एक ही प्रकार के डेटा (आर्टिफैक्ट्स) को संग्रहीत करते हैं.

आइए हम ब्राउज़रों द्वारा संग्रहीत सबसे सामान्य आर्टिफैक्ट्स पर एक नज़र डालते हैं.

* **नेविगेशन हिस्ट्री:** उपयोगकर्ता के नेविगेशन हिस्ट्री के बारे में डेटा होता है. उदाहरण के लिए, यह ट्रैक करने के लिए उपयोग किया जा सकता है कि क्या उपयोगकर्ता ने किसी दुर्भावनापूर्ण साइट्स का दौरा किया है.
* **ऑटोकम्प्लीट डेटा:** यह वह डेटा है जो ब्राउज़र आपके द्वारा सबसे अधिक खोजी गई चीज़ों के आधार पर सुझाव देता है. नेविगेशन हिस्ट्री के साथ मिलकर अधिक अंतर्दृष्टि प्राप्त करने के लिए उपयोग किया जा सकता है.
* **बुकमार्क्स:** स्वयं स्पष्ट.
* **एक्सटेंशन्स और ऐड ऑन्स:** स्वयं स्पष्ट.
* **कैश:** वेबसाइट्स को नेविगेट करते समय, ब्राउज़र कई कारणों से सभी प्रकार के कैश डेटा (इमेजेज, जावास्क्रिप्ट फाइलें...आदि) बनाता है. उदाहरण के लिए वेबसाइट्स के लोडिंग समय को तेज करने के लिए. ये कैश फाइलें फोरेंसिक जांच के दौरान डेटा का एक महान स्रोत हो सकती हैं.
* **लॉगिन्स:** स्वयं स्पष्ट.
* **फेविकॉन्स:** ये टैब्स, यूआरएल, बुकमार्क्स और इसी तरह की छोटी आइकन होती हैं. वेबसाइट या उपयोगकर्ता द्वारा दौरा किए गए स्थानों के बारे में अधिक जानकारी प्राप्त करने के लिए एक और स्रोत के रूप में उपयोग की जा सकती हैं.
* **ब्राउज़र सेशन्स:** स्वयं स्पष्ट.
* **डाउनलोड्स**: स्वयं स्पष्ट.
* **फॉर्म डेटा:** फॉर्म के अंदर टाइप की गई कोई भी चीज़ अक्सर ब्राउज़र द्वारा संग्रहीत की जाती है, ताकि अगली बार जब उपयोगकर्ता फॉर्म के अंदर कुछ दर्ज करता है तो ब्राउज़र पहले दर्ज किए गए डेटा का सुझाव दे सकता है.
* **थंबनेल्स:** स्वयं स्पष्ट.
* **कस्टम डिक्शनरी.txt**: उपयोगकर्ता द्वारा डिक्शनरी में जोड़े गए शब्द.

## Firefox

Firefox \~/_**.mozilla/firefox/**_ (Linux) में प्रोफाइल्स फोल्डर बनाता है, **/Users/$USER/Library/Application Support/Firefox/Profiles/** (MacOS) में, _**%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\\**_ (Windows) में_**.**_\
इस फोल्डर के अंदर, _**profiles.ini**_ फाइल उपयोगकर्ता प्रोफाइल(स) के नाम के साथ दिखाई देनी चाहिए.\
प्रत्येक प्रोफाइल में एक "**Path**" वेरिएबल होता है जिसमें उसके डेटा को संग्रहीत किए जाने वाले फोल्डर का नाम होता है. फोल्डर को **उसी डायरेक्टरी में मौजूद होना चाहिए जहां \_profiles.ini**\_\*\* मौजूद है\*\*. यदि ऐसा नहीं है, तो संभवतः इसे हटा दिया गया है.

प्रत्येक प्रोफाइल के फोल्डर (_\~/.mozilla/firefox/\<ProfileName>/_) पथ के अंदर आपको निम्नलिखित दिलचस्प फाइलें मिलनी चाहिए:

* _**places.sqlite**_ : हिस्ट्री (moz\_\_places), बुकमार्क्स (moz\_bookmarks), और डाउनलोड्स (moz\_\_annos). Windows में _**places.sqlite**_ के अंदर हिस्ट्री पढ़ने के लिए [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing_history_view.html) टूल का उपयोग किया जा सकता है.
* हिस्ट्री डंप करने के लिए क्वेरी: `select datetime(lastvisitdate/1000000,'unixepoch') as visit_date, url, title, visit_count, visit_type FROM moz_places,moz_historyvisits WHERE moz_places.id = moz_historyvisits.place_id;`
* ध्यान दें कि लिंक प्रकार एक संख्या है जो इंगित करती है:
* 1: उपयोगकर्ता ने एक लिंक का अनुसरण किया
* 2: उपयोगकर्ता ने URL लिखा
* 3: उपयोगकर्ता ने एक पसंदीदा का उपयोग किया
* 4: Iframe से लोड किया गया
* 5: HTTP रीडायरेक्ट 301 के माध्यम से पहुँचा
* 6: HTTP रीडायरेक्ट 302 के माध्यम से पहुँचा
* 7: फाइल डाउनलोड की गई
* 8: उपयोगकर्ता ने Iframe के अंदर एक लिंक का अनुसरण किया
* डाउनलोड्स डंप करने के लिए क्वेरी: `SELECT datetime(lastModified/1000000,'unixepoch') AS down_date, content as File, url as URL FROM moz_places, moz_annos WHERE moz_places.id = moz_annos.place_id;`
*
* _**bookmarkbackups/**_ : बुकमार्क्स बैकअप
* _**formhistory.sqlite**_ : **वेब फॉर्म डेटा** (जैसे ईमेल)
* _**handlers.json**_ : प्रोटोकॉल हैंडलर्स (जैसे, कौन सा ऐप _mailto://_ प्रोटोकॉल को हैंडल करेगा)
* _**persdict.dat**_ : डिक्शनरी में जोड़े गए शब्द
* _**addons.json**_ और \_**extensions.sqlite** \_ : इंस्टॉल किए गए
```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```
## Google Chrome

Google Chrome उपयोगकर्ता के होम में प्रोफाइल बनाता है _**\~/.config/google-chrome/**_ (Linux), _**C:\Users\XXX\AppData\Local\Google\Chrome\User Data\\**_ (Windows), या _**/Users/$USER/Library/Application Support/Google/Chrome/**_ (MacOS)।\
अधिकांश जानकारी _**Default/**_ या _**ChromeDefaultData/**_ फोल्डर्स में सहेजी जाएगी जो पहले बताए गए पथों के अंदर होते हैं। यहां आपको निम्नलिखित दिलचस्प फाइलें मिल सकती हैं:

* _**History**_: URLs, डाउनलोड्स और यहां तक कि खोजे गए कीवर्ड्स। Windows में, आप [ChromeHistoryView](https://www.nirsoft.net/utils/chrome_history_view.html) टूल का उपयोग करके इतिहास पढ़ सकते हैं। "Transition Type" कॉलम का अर्थ है:
* Link: उपयोगकर्ता ने एक लिंक पर क्लिक किया
* Typed: URL लिखा गया था
* Auto Bookmark
* Auto Subframe: जोड़ें
* Start page: होम पेज
* Form Submit: एक फॉर्म भरा गया और भेजा गया
* Reloaded
* _**Cookies**_: Cookies। [ChromeCookiesView](https://www.nirsoft.net/utils/chrome_cookies_view.html) का उपयोग करके कुकीज़ की जांच की जा सकती है।
* _**Cache**_: Cache। Windows में, आप [ChromeCacheView](https://www.nirsoft.net/utils/chrome_cache_view.html) टूल का उपयोग करके कैश की जांच कर सकते हैं।
* _**Bookmarks**_: Bookmarks
* _**Web Data**_: फॉर्म इतिहास
* _**Favicons**_: Favicons
* _**Login Data**_: लॉगिन जानकारी (उपयोगकर्ता नाम, पासवर्ड...)
* _**Current Session**_ और _**Current Tabs**_: वर्तमान सत्र डेटा और वर्तमान टैब्स
* _**Last Session**_ और _**Last Tabs**_: ये फाइलें उन साइटों को रखती हैं जो ब्राउज़र में सक्रिय थीं जब Chrome अंतिम बार बंद हुआ था।
* _**Extensions**_: एक्सटेंशन्स और ऐडऑन्स फोल्डर
* **Thumbnails** : Thumbnails
* **Preferences**: इस फाइल में प्लगइन्स, एक्सटेंशन्स, जियोलोकेशन का उपयोग करने वाली साइट्स, पॉपअप्स, नोटिफिकेशन्स, DNS प्रीफेचिंग, सर्टिफिकेट अपवाद, और बहुत कुछ जैसी अच्छी जानकारी होती है। यदि आप यह शोध करने की कोशिश कर रहे हैं कि क्या कोई विशिष्ट Chrome सेटिंग सक्षम थी, तो आपको यह सेटिंग यहां मिल सकती है।
* **Browser’s built-in anti-phishing:** `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`
* आप सिर्फ “**safebrowsing**” के लिए grep कर सकते हैं और परिणाम में `{"enabled: true,"}` की तलाश कर सकते हैं जो एंटी-फ़िशिंग और मैलवेयर सुरक्षा सक्षम होने का संकेत देता है।

## **SQLite DB Data Recovery**

जैसा कि आप पिछले खंडों में देख सकते हैं, Chrome और Firefox दोनों डेटा स्टोर करने के लिए **SQLite** डेटाबेस का उपयोग करते हैं। [**sqlparse**](https://github.com/padfoot999/sqlparse) **या** [**sqlparse_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases) टूल का उपयोग करके हटाए गए प्रविष्टियों को **पुनर्प्राप्त** करना संभव है।

## **Internet Explorer 11**

Internet Explorer **डेटा** और **मेटाडेटा** को विभिन्न स्थानों पर संग्रहीत करता है। मेटाडेटा डेटा को खोजने में मदद करेगा।

**मेटाडेटा** को `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` फोल्डर में पाया जा सकता है जहां VX V01, V16, या V24 हो सकता है।\
पिछले फोल्डर में, आप V01.log फाइल भी पा सकते हैं। यदि इस फाइल और WebcacheVX.data फाइल के **संशोधित समय** **अलग-अलग हैं** तो आपको संभावित **असंगतियों** को **ठीक** करने के लिए `esentutl /r V01 /d` कमांड चलाने की आवश्यकता हो सकती है।

एक बार **पुनर्प्राप्त** कर लेने के बाद यह आर्टिफैक्ट (यह एक ESE डेटाबेस है, photorec इसे Exchange Database या EDB विकल्पों के साथ पुनर्प्राप्त कर सकता है) आप [ESEDatabaseView](https://www.nirsoft.net/utils/ese_database_view.html) प्रोग्राम का उपयोग करके इसे खोल सकते हैं। एक बार **खोलने** के बाद, "**Containers**" नामक टेबल पर जाएं।

![](<../../../.gitbook/assets/image (446).png>)

इस टेबल के अंदर, आप पा सकते हैं कि किस अन्य टेबल्स या कंटेनर्स में प्रत्येक भाग की संग्रहीत जानकारी सहेजी गई है। उसके बाद, आप ब्राउज़र्स द्वारा संग्रहीत **डेटा के स्थानों** और उसके अंदर के **मेटाडेटा** को पा सकते हैं।

**ध्यान दें कि यह टेबल Microsoft के अन्य टूल्स के लिए कैश के मेटाडेटा का भी संकेत देती है (जैसे कि skype)**

### Cache

आप [IECacheView](https://www.nirsoft.net/utils/ie_cache_viewer.html) टूल का उपयोग करके कैश की जांच कर सकते हैं। आपको उस फोल्डर को इंगित करना होगा जहां आपने कैश डेटा निकाला है।

#### Metadata

कैश के बारे में मेटाडेटा जानकारी में शामिल है:

* डिस्क में फाइलनाम
* SecureDIrectory: कैश निर्देशिकाओं के अंदर फाइल का स्थान
* AccessCount: कैश में सहेजे जाने की संख्या
* URL: उत्पत्ति का URL
* CreationTime: पहली बार जब इसे कैश किया गया था
* AccessedTime: समय जब कैश का उपयोग किया गया था
* ModifiedTime: वेबपेज का अंतिम संस्करण
* ExpiryTime: समय जब कैश समाप्त हो जाएगा

#### Files

कैश जानकारी _**%userprofile%\Appdata\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5**_ और _**%userprofile%\Appdata\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5\low**_ में पाई जा सकती है।

इन फोल्डरों के अंदर की जानकारी वह **स्नैपशॉट है जो उपयोगकर्ता देख रहा था**। कैश का आकार **250 MB** होता है और टाइमस्टैम्प्स बताते हैं कि पेज कब देखा गया था (पहली बार, NTFS की निर्माण तिथि, अंतिम बार, NTFS की संशोधन तिथि)।

### Cookies

आप [IECookiesView](https://www.nirsoft.net/utils/iecookies.html) टूल का उपयोग करके कुकीज़ की जांच कर सकते हैं। आपको उस फोल्डर को इंगित करना होगा जहां आपने कुकीज़ निकाली हैं।

#### **Metadata**

कुकीज़ के बारे में मेटाडेटा जानकारी में शामिल है:

* फाइलसिस्टम में कुकी का नाम
* URL
* AccessCount: कुकीज़ को सर्वर को भेजे जाने की संख्या
* CreationTime: पहली बार जब कुकी बनाई गई थी
* ModifiedTime: अंतिम बार जब कुकी संशोधित की गई थी
* AccessedTime: अंतिम बार जब कुकी तक पहुंचा गया था
* ExpiryTime: कुकी की समाप्ति का समय

#### Files

कुकीज़ का डेटा _**%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies**_ और _**%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies\low**_ में पाया जा सकता है।

सत्र कुकीज़ मेमोरी में रहेंगी और स्थायी कुकी डिस्क में।

### Downloads

#### **Metadata**

[ESEDatabaseView](https://www.nirsoft.net/utils/ese_database_view.html) टूल की जांच करने पर आप डाउनलोड्स के मेटाडेटा के साथ कंटेनर पा सकते हैं:

![](<../../../.gitbook/assets/image (445).png>)

"ResponseHeaders" कॉलम की जानकारी प्राप्त करके आप उस जानकारी को हेक्स से बदल सकते हैं और डाउनलोड की गई फाइल का URL, फाइल प्रकार और स्थान प्राप्त कर सकते हैं।

#### Files

पथ _**%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory**_ म
