# ब्राउज़र आर्टिफैक्ट्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PRs सबमिट करके।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **वर्कफ़्लो** को आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ब्राउज़र आर्टिफैक्ट्स <a href="#id-3def" id="id-3def"></a>

जब हम ब्राउज़र आर्टिफैक्ट्स के बारे में बात करते हैं, तो हम नेविगेशन हिस्ट्री, बुकमार्क्स, डाउनलोड की गई फ़ाइलों की सूची, कैश डेटा आदि के बारे में बात करते हैं।

ये आर्टिफैक्ट्स ऑपरेटिंग सिस्टम में विशेष फ़ोल्डर में स्टोर की जाने वाली फ़ाइलें हैं।

प्रत्येक ब्राउज़र अपनी फ़ाइलें अन्य ब्राउज़रों से अलग स्थान पर स्टोर करता है और उनके पास अलग-अलग नाम होते हैं, लेकिन वे सभी (अधिकांश समय) समान प्रकार के डेटा (आर्टिफैक्ट्स) स्टोर करते हैं।

चलिए हम देखते हैं कि ब्राउज़र्स द्वारा स्टोर किए जाने वाले सबसे सामान्य आर्टिफैक्ट्स क्या हैं।

* **नेविगेशन हिस्ट्री:** उपयोगकर्ता की नेविगेशन हिस्ट्री के बारे में डेटा शामिल है। उदाहरण के लिए उपयोगकर्ता ने कुछ दुष्ट साइटों पर जाए होने का पता लगाने के लिए उपयोग किया जा सकता है
* **ऑटोकंप्लीट डेटा:** यह डेटा है जिसे ब्राउज़र सुझाव देता है आपके द्वारा सबसे अधिक खोजा गया आधारित। इसे नेविगेशन हिस्ट्री के साथ उपयोग करके अधिक जानकारी प्राप्त की जा सकती है।
* **बुकमार्क्स:** स्वयं समझदारी।
* **एक्सटेंशन्स और एड ऑन्स:** स्वयं समझदारी।
* **कैश:** वेबसाइटों पर नेविगेट करते समय, ब्राउज़र विभिन्न प्रकार के कैश डेटा (छवियाँ, जावास्क्रिप्ट फ़ाइलें…आदि) बनाता है बहुत से कारणों के लिए। उदाहरण के लिए वेबसाइटों के लोडिंग समय को तेज करने के लिए। ये कैश फ़ाइलें एक जांच जाँच के दौरान डेटा का एक महान स्रोत हो सकती हैं।
* **लॉगिन्स:** स्वयं समझदारी।
* **फेविकॉन्स:** वे छोटे आइकन हैं जो टैब, यूआरएल, बुकमार्क्स और इस प्रकार की जगहों पर पाए जाते हैं। इन्हें एक और स्रोत के रूप में उपयोग किया जा सकता है वेबसाइट या उपयोगकर्ता द्वारा जाए गए स्थानों के बारे में अधिक जानकारी प्राप्त करने के लिए।
* **ब्राउज़र सेशन्स:** स्वयं समझदारी।
* **डाउनलोड्स**: स्वयं समझदारी।
* **फॉर्म डेटा:** फॉर्म में टाइप किया गया कुछ भी ब्राउज़र द्वारा अक्सर स्टोर किया जाता है, ताकि अगली बार जब उपयोगकर्ता फॉर्म में कुछ भरता है तो ब्राउज़र पहले से भरे गए डेटा का सुझाव दे सके।
* **थंबनेल्स:** स्वयं समझदारी।
* **कस्टम डिक्शनरी.txt**: उपयोगकर्ता द्वारा शब्द जोड़े गए।

## फायरफॉक्स

फायरफॉक्स \~/_**.mozilla/firefox/**_ (Linux) में प्रोफाइल फ़ोल्डर बनाता है, **/Users/$USER/Library/Application Support/Firefox/Profiles/** (MacOS) में, _**%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\\**_ (Windows)_**.**_\
इस फ़ोल्डर के अंदर, फ़ाइल _**profiles.ini**_ उपयोगकर्ता प्रोफ़ाइल(स) के नाम(स) के साथ दिखना चाहिए।\
प्रत्येक प्रोफ़ाइल में "**पथ**" चरण होता है जिसमें उस फ़ोल्डर का नाम होता है जिसमें उसका डेटा स्टोर होगा। फ़ोल्डर को **\_\_profiles.ini**\_\*\* के साथ उपस्थित होना चाहिए। अगर यह नहीं है, तो संभावना है कि यह हटा दिया गया था।

प्रत्येक प्रोफ़ाइल के फ़ोल्डर के अंदर (_\~/.mozilla/firefox/\<ProfileName>/_) पढ़ने योग्य निम्नलिखित फ़ाइलें मिलनी चाहिए:

* _**places.sqlite**_ : इतिहास (moz\_\_places), बुकमार्क्स (moz\_bookmarks), और डाउनलोड्स (moz\_\_annos)। Windows में उपकरण [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) का उपयोग _**places.sqlite**_ में इतिहास पढ़ने के लिए किया जा सकता है।
* इतिहास डंप करने के लिए क्वेरी: `select datetime(lastvisitdate/1000000,'unixepoch') as visit_date, url, title, visit_count, visit_type FROM moz_places,moz_historyvisits WHERE moz_places.id = moz_historyvisits.place_id;`
* ध्यान दें कि लिंक प्रकार एक संख्या है जो इंडिकेट करती है:
* 1: उपयोगकर्ता ने लिंक का पालन किया
* 2: उपयोगकर्ता ने यूआरएल लिखा
* 3: उपयोगकर्ता ने पसंदीदा का उपयोग किया
* 4: आईफ्रेम से लोड किया गया
* 5: HTTP रीडायरेक्ट 301 के माध्यम से पहुंचा गया
* 6: HTTP रीडायरेक्ट 302 के माध्यम से पहुंचा गया
* 7: फ़ाइल डाउनलोड किया गया
* 8: उपयोगकर्ता ने आईफ्रेम के अंदर लिंक का पालन किया
* डाउनलोड्स डंप करने के लिए क्वेरी: `SELECT datetime(lastModified/1000000,'unixepoch') AS down_date, content as File, url as URL FROM moz_places, moz_annos WHERE moz_places.id = moz_annos.place_id;`
*
* _**bookmarkbackups/**_ : बुकमार्क्स बैकअप्स
* _**formhistory.sqlite**_ : **वेब फॉर्म डेटा** (जैसे ईमेल)
* _**handlers.json**_ : प्रोटोकॉल हैंडलर्स (जैसे, कौन _mailto://_ प्रोटोकॉल को हैंडल करेगा)
* _**persdict.dat**_ : उपयोगकर्ता द्वारा शब्द जोड
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

Google Chrome उपयोगकर्ता के घर के अंदर प्रोफ़ाइल बनाता है _**\~/.config/google-chrome/**_ (Linux), में _**C:\Users\XXX\AppData\Local\Google\Chrome\User Data\\**_ (Windows), या _**/Users/$USER/Library/Application Support/Google/Chrome/**_ (MacOS)।
अधिकांश जानकारी _**Default/**_ या _**ChromeDefaultData/**_ फ़ोल्डर में सहेजी जाएगी जो पहले निर्दिष्ट पथों के अंदर हैं। यहाँ आप निम्नलिखित दिलचस्प फ़ाइलें पा सकते हैं:

* _**History**_: URLs, डाउनलोड और खोजे गए शब्द। Windows में, आप इतिहास पढ़ने के लिए उपकरण [ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html) का उपयोग कर सकते हैं। "Transition Type" स्तंभ का मतलब है:
* Link: उपयोगकर्ता ने लिंक पर क्लिक किया
* Typed: URL लिखा गया था
* Auto Bookmark
* Auto Subframe: जोड़ें
* Start page: होम पेज
* Form Submit: एक फ़ॉर्म भरा और भेजा गया था
* Reloaded
* _**Cookies**_: कुकीज़। [ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html) का उपयोग कुकीज़ की जांच करने के लिए किया जा सकता है।
* _**Cache**_: कैश। Windows में, आप उपकरण [ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html) का उपयोग कैश की जांच करने के लिए कर सकते हैं।
* _**Bookmarks**_: बुकमार्क्स
* _**Web Data**_: फ़ॉर्म इतिहास
* _**Favicons**_: फ़ेविकॉन्स
* _**Login Data**_: लॉगिन जानकारी (उपयोगकर्ता नाम, पासवर्ड...)
* _**Current Session**_ और _**Current Tabs**_: वर्तमान सत्र डेटा और वर्तमान टैब्स
* _**Last Session**_ और _**Last Tabs**_: इन फ़ाइलों में वह साइटें हैं जो ब्राउज़र में सक्रिय थीं जब Chrome को आखिरी बार बंद किया गया था।
* _**Extensions**_: एक्सटेंशन्स और एडऑन्स फ़ोल्डर
* **Thumbnails** : थंबनेल्स
* **Preferences**: इस फ़ाइल में प्लगइन्स, एक्सटेंशन्स, जियोलोकेशन का उपयोग करने वाली साइटें, पॉपअप्स, सूचनाएँ, DNS prefetching, प्रमाणपत्र अपवाद और बहुत कुछ जैसी अच्छी जानकारी की एक बहुत सारी जानकारी होती है। यदि आप जांच कर रहे हैं कि क्या कोई विशिष्ट Chrome सेटिंग सक्षम थी या नहीं, तो आप शायद इसमें उस सेटिंग को पाएंगे।
* **Browser’s built-in anti-phishing:** `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`
* आप बस “**safebrowsing**” के लिए grep कर सकते हैं और परिणाम में `{"enabled: true,"}` खोज सकते हैं जो दिखाता है कि एंटी-फिशिंग और मैलवेयर सुरक्षा सक्षम है।

## **SQLite DB Data Recovery**

जैसा कि आप पिछले खंडों में देख सकते हैं, च्रोम और फ़ायरफ़ॉक्स दोनों डेटा सहेजने के लिए **SQLite** डेटाबेस का उपयोग करते हैं। इस उपकरण का उपयोग करके **हटाए गए एंट्रीज़ को पुनः प्राप्त किया जा सकता है** [**sqlparse**](https://github.com/padfoot999/sqlparse) **या** [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases)।

## **Internet Explorer 11**

इंटरनेट एक्सप्लोरर **डेटा** और **मेटाडेटा** विभिन्न स्थानों में संग्रहित करता है। मेटाडेटा डेटा खोजने में मदद करेगा।

मेटाडेटा फ़ोल्डर में मिल सकता है `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` जहाँ VX V01, V16, या V24 हो सकता है।\
पिछले फ़ोल्डर में, आप फ़ाइल V01.log भी पा सकते हैं। यदि इस फ़ाइल की **संशोधित समय** और WebcacheVX.data फ़ाइल का **अलग है** तो आपको संभावित **असंगतताओं** को ठीक करने के लिए आदेश `esentutl /r V01 /d` चलाने की आवश्यकता हो सकती है।

एक बार इस आर्टिफैक्ट को पुनः प्राप्त किया गया है (यह एक ESE डेटाबेस है, photorec इसे विनिमय डेटाबेस या EDB विकल्प के साथ पुनर्प्राप्त कर सकता है) तो आप इसे खोलने के लिए प्रोग्राम [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) का उपयोग कर सकते हैं। एक बार **खोलने** के बाद, जाएं तालिका नामित "**Containers**".

![](<../../../.gitbook/assets/image (446).png>)

इस तालिका के अंदर, आप प्राप्त सूचना के प्रत्येक हिस्से को किस अन्य तालिकाओं या कंटेनर में सहेजा गया है, वहाँ से आप ब्राउज़र्स द्वारा संग्रहित डेटा की **स्थान** और उसमें मौजूद **मेटाडेटा** पा सकते हैं।

**ध्यान दें कि यह तालिका अन्य माइक्रोसॉफ्ट उपकरणों के लिए भी कैश की मेटाडेटा को दर्शाता है (जैसे skype)**

### कैश

आप टूल [IECacheView](https://www.nirsoft.net/utils/ie\_cache\_viewer.html) का उपयोग कैश की जांच करने के लिए कर सकते हैं। आपको उस फ़ोल्डर की स्थान दर्शाना होगा जहाँ आपने कैश डेटा निकाला है।

#### मेटाडेटा

कैश के बारे में मेटाडेटा जानकारी सहेजता है:

* डिस्क में फ़ाइल का नाम
* SecureDIrectory: कैश निर्देशिकाओं के अंदर फ़ाइल का स्थान
* AccessCount: कितनी बार यह कैश में सहेजा गया था
* URL: यूआरएल मूल
* CreationTime: पहली बार जब यह कैश किया गया था
* AccessedTime: कैश का उपयोग किया गया समय
* ModifiedTime: अंतिम वेबपेज संस्करण
* ExpiryTime: कैश का समय जब समाप्त होगा

#### फ़ाइलें

कैश जानकारी _**%userprofile%\Appdata\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5**_ और _**%userprofile%\Appdata\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5\low**_ में पाई जा सकती है।

इन फ़ोल्डरों के अंदर की जानकारी उपयोगकर्ता द्वारा देखा जा रहा था। कैश का आकार **250 MB** होता है और टाइमस्टैम्प दिखाता है कि पृष्ठ कब देखा गया था (पहली बार, NTFS के निर्माण तिथि, अंतिम समय, NTFS के संशोधन समय)।

### कुकीज़

आप टूल [IECookiesView](https://www.nirsoft.net/utils/iecookies.html) का उपयोग कुकीज़ की जांच करने के लिए कर सकते हैं। आपको उस फ़ोल्डर की स्थान दर्शाना होगा जहाँ आपने कुकीज़ निकाली हैं।

#### **मेटाडेटा**

कुकीज़ के बारे में मेटाडेटा जानकारी सहेजता है:

* फ़ाइल सिस्टम में कुकीज़ का नाम
* URL
* AccessCount: कितनी बार कुकीज़ सर्वर को भेजी गई है
* CreationTime: पहली बार कुकीज़ बनाई गई थी
* ModifiedTime: अंतिम बार कुकीज़ को संशोधित किया गया था
* AccessedTime: कुकीज़ को अंतिम बार एक्सेस किया गया था
* ExpiryTime: कुकीज़ की समाप्ति का समय

#### फ़ाइलें

कुकीज़ डेटा _**%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies**_ और _**%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies\low**_ में पाई जा सकती है।

सत्र कुकीज़ मेमोरी में रहेंगी और स्थायी कुकी डिस्क में।

### डाउनलोड

#### **मेटाडेटा**

उपकरण [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) की ज
