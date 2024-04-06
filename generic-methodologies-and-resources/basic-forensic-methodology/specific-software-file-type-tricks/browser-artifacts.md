# ब्राउज़र आर्टिफैक्ट्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **वर्कफ़्लो** को आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ब्राउज़र आर्टिफैक्ट्स <a href="#id-3def" id="id-3def"></a>

ब्राउज़र आर्टिफैक्ट्स में वेब ब्राउज़र्स द्वारा संग्रहित विभिन्न प्रकार के डेटा शामिल हैं, जैसे नेविगेशन हिस्ट्री, बुकमार्क्स, और कैश डेटा। ये आर्टिफैक्ट्स ऑपरेटिंग सिस्टम के भीतर विशेष फोल्डर में रखे जाते हैं, जो ब्राउज़र के अलग-अलग स्थानों में और नामों में भिन्न होते हैं, लेकिन सामान्य रूप से समान डेटा प्रकार संग्रहित करते हैं।

यहाँ एक सारांश है सबसे सामान्य ब्राउज़र आर्टिफैक्ट्स का:

* **नेविगेशन हिस्ट्री**: उपयोगकर्ता की वेबसाइटों पर यात्राओं का ट्रैक, जो खतरनाक साइटों पर यात्राओं की पहचान के लिए उपयोगी है।
* **ऑटोकंप्लीट डेटा**: आधारित सुझाव जो नियमित खोजों पर आधारित हैं, जब नेविगेशन हिस्ट्री के साथ मिलाकर उपयोगकर्ता को अनुभव प्रदान करते हैं।
* **बुकमार्क्स**: उपयोगकर्ता द्वारा त्वरित पहुंच के लिए सहेजी गई साइटें।
* **एक्सटेंशन और एड-ऑन्स**: उपयोगकर्ता द्वारा स्थापित ब्राउज़र एक्सटेंशन या एड-ऑन्स।
* **कैश**: वेब सामग्री (जैसे छवियाँ, जावास्क्रिप्ट फ़ाइलें) को वेबसाइट लोडिंग समय सुधारने के लिए संग्रहित करता है, जिसे फोरेंसिक विश्लेषण के लिए मूल्यवान माना जाता है।
* **लॉगिन्स**: संग्रहित लॉगिन क्रेडेंशियल्स।
* **फेविकॉन्स**: वेबसाइटों से संबंधित आइकन, जो टैब्स और बुकमार्क्स में दिखाई देते हैं, उपयोगकर्ता यात्राओं पर अतिरिक्त जानकारी के लिए उपयोगी है।
* **ब्राउज़र सेशन्स**: खुले ब्राउज़र सत्र से संबंधित डेटा।
* **डाउनलोड्स**: ब्राउज़र के माध्यम से डाउनलोड किए गए फ़ाइलों की रिकॉर्ड।
* **फॉर्म डेटा**: वेब फ़ॉर्मों में दर्ज की गई जानकारी, भविष्य के ऑटोफ़िल सुझाव के लिए सहेजी गई।
* **थंबनेल्स**: वेबसाइट की पूर्वावलोकन छवियाँ।
* **कस्टम डिक्शनरी.txt**: उपयोगकर्ता द्वारा ब्राउज़र के शब्दकोश में जोड़े गए शब्द।

## फायरफॉक्स

फायरफॉक्स प्रोफ़ाइल्स के भीतर उपयोगकर्ता डेटा को ऑर्गनाइज़ करता है, जो ऑपरेटिंग सिस्टम के आधार पर विशेष स्थानों में संग्रहीत होते हैं:

* **Linux**: `~/.mozilla/firefox/`
* **MacOS**: `/Users/$USER/Library/Application Support/Firefox/Profiles/`
* **Windows**: `%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\`

इन निर्देशिकाओं के भीतर एक `profiles.ini` फ़ाइल उपयोगकर्ता प्रोफ़ाइल्स की सूची देती है। प्रत्येक प्रोफ़ाइल के डेटा को `profiles.ini` के समान निर्देशिका में स्थित `Path` चर में नामित फ़ोल्डर में संग्रहीत किया जाता है। यदि किसी प्रोफ़ाइल का फ़ोल्डर गायब है, तो यह हटा दिया गया हो सकता है।

प्रत्येक प्रोफ़ाइल फ़ोल्डर के भीतर, आप कई महत्वपूर्ण फ़ाइलें पा सकते हैं:

* **places.sqlite**: इतिहास, बुकमार्क्स, और डाउनलोड संग्रहीत करता है। [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) जैसे उपकरण Windows पर इतिहास डेटा तक पहुंच प्रदान कर सकते हैं।
* इतिहास और डाउनलोड जानकारी निकालने के लिए विशेष SQL क्वेरी का उपयोग करें।
* **bookmarkbackups**: बुकमार्क की बैकअप संग्रहित करता है।
* **formhistory.sqlite**: वेब फॉर्म डेटा संग्रहित करता है।
* **handlers.json**: प्रोटोकॉल हैंडलर प्रबंधित करता है।
* **persdict.dat**: कस्टम शब्दकोश शब्द।
* **addons.json** और **extensions.sqlite**: स्थापित एड-ऑन्स और एक्सटेंशन की जानकारी।
* **cookies.sqlite**: कुकी संग्रहण, जिसे Windows पर जांच के लिए [MZCookiesView](https://www.nirsoft.net/utils/mzcv.html) उपलब्ध है।
* **cache2/entries** या **startupCache**: कैश डेटा, [MozillaCacheView](https://www.nirsoft.net/utils/mozilla\_cache\_viewer.html) जैसे उपकरणों के माध्यम से पहुंचने योग्य।
* **favicons.sqlite**: फेविकॉन्स संग्रहित करता है।
* **prefs.js**: उपयोगकर्ता सेटिंग्स और पसंद।
* **downloads.sqlite**: पुराने डाउनलोड डेटाबेस, अब places.sqlite में समाहित।
* **thumbnails**: वेबसाइट थंबनेल्स।
* **logins.json**: एन्क्रिप्टेड लॉगिन जानकारी।
* **key4.db** या **key3.db**: संवेदनशील जानकारी सुरक्षित करने के लिए एन्क्रिप्शन कुंजियाँ संग्रहित करता है।

इसके अतिरिक्त, ब्राउज़र की एंटी-फिशिंग सेटिंग्स की जांच करने के लिए `prefs.js` में `browser.safebrowsing` एंट्रीज़ की खोज की जा सकती है, जो सुरक्षित ब्राउज़िंग सुविधाएँ सक्षम या अक्षम हैं इसका संकेत देती है।

मास्टर पासवर्ड को डिक्रिप्ट करने की कोशिश करने के लिए, आप [https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt) का उपयोग कर सकते हैं\
निम्नलिखित स
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

Google Chrome उपयोगकर्ता प्रोफाइल को ऑपरेटिंग सिस्टम के आधार पर विशिष्ट स्थानों में संग्रहित करता है:

- **Linux**: `~/.config/google-chrome/`
- **Windows**: `C:\Users\XXX\AppData\Local\Google\Chrome\User Data\`
- **MacOS**: `/Users/$USER/Library/Application Support/Google/Chrome/`

इन निर्देशिकाओं में, अधिकांश उपयोगकर्ता डेटा **Default/** या **ChromeDefaultData/** फोल्डर में पाया जा सकता है। निम्नलिखित फ़ाइलें महत्वपूर्ण डेटा रखती हैं:

- **History**: URL, डाउनलोड और खोज कीवर्ड रखती है। Windows पर, [ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html) का उपयोग इतिहास को पढ़ने के लिए किया जा सकता है। "Transition Type" स्तंभ में विभिन्न अर्थ होते हैं, जैसे उपयोगकर्ता लिंक पर क्लिक करते हैं, टाइप किए गए URL, फॉर्म सबमिशन और पेज रीलोड्स।
- **Cookies**: कुकीज़ संग्रहित करती है। निरीक्षण के लिए, [ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html) उपलब्ध है।
- **Cache**: कैश डेटा को रखती है। निरीक्षण के लिए, Windows उपयोगकर्ता [ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html) का उपयोग कर सकते हैं।
- **Bookmarks**: उपयोगकर्ता बुकमार्क्स।
- **Web Data**: फॉर्म इतिहास रखती है।
- **Favicons**: वेबसाइट फेविकॉन्स को संग्रहित करती है।
- **Login Data**: उपयोगकर्ता नाम और पासवर्ड जैसी लॉगिन क्रेडेंशियल्स शामिल हैं।
- **Current Session**/**Current Tabs**: वर्तमान ब्राउज़िंग सत्र और खुली टैब्स के बारे में डेटा।
- **Last Session**/**Last Tabs**: क्रोम बंद होने से पहले चल रहे अंतिम सत्र के बारे में जानकारी।
- **Extensions**: ब्राउज़र एक्सटेंशन्स और एडऑन्स के लिए निर्देशिकाएँ।
- **Thumbnails**: वेबसाइट थंबनेल्स को संग्रहित करती है।
- **Preferences**: एक फ़ाइल जिसमें सेटिंग्स के लिए जानकारी होती है, जैसे प्लगइन्स, एक्सटेंशन्स, पॉप-अप्स, सूचनाएँ, और अधिक।
- **Browser’s built-in anti-phishing**: जांचने के लिए कि क्या एंटी-फिशिंग और मैलवेयर सुरक्षा सक्षम हैं, रन करें `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`। आउटपुट में `{"enabled: true,"}` खोजें।

## **SQLite DB डेटा रिकवरी**

पिछले खंडों में आप देख सकते हैं कि च्रोम और फ़ायरफ़ॉक्स डेटा संग्रहित करने के लिए **SQLite** डेटाबेस का उपयोग करते हैं। इस उपकरण [**sqlparse**](https://github.com/padfoot999/sqlparse) या [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases) का उपयोग करके **हटाए गए एंट्रीज़ को बहाल किया जा सकता है**।

## **Internet Explorer 11**

इंटरनेट एक्सप्लोरर 11 अपने डेटा और मेटाडेटा को विभिन्न स्थानों पर संग्रहित करता है, जो संग्रहित जानकारी और उसके संबंधित विवरणों को आसान पहुंच और प्रबंधन के लिए सहायक होता है।

### मेटाडेटा संग्रहण

इंटरनेट एक्सप्लोरर के लिए मेटाडेटा `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` में संग्रहित होता है (VX V01, V16, या V24 होता है)। इसके साथ, `V01.log` फ़ाइल `WebcacheVX.data` के साथ संशोधन समय विसंगतियों को दिखा सकती है, जिससे `esentutl /r V01 /d` का उपयोग करके मरम्मत की आवश्यकता हो सकती है। इस मेटाडेटा, जो एक ESE डेटाबेस में संग्रहित है, को उपकरणों जैसे photorec और [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) का उपयोग करके पुनर्प्राप्त और निरीक्षित किया जा सकता है। **Containers** तालिका में, प्रत्येक डेटा सेगमेंट को संग्रहित करने वाले विशिष्ट तालिकाएँ या कंटेनर्स को अंतर्निहित किया जा सकता है, जिसमें अन्य माइक्रोसॉफ्ट टूल्स जैसे स्काइप के लिए कैश विवरण शामिल हैं।

### कैश निरीक्षण

[IECacheView](https://www.nirsoft.net/utils/ie\_cache\_viewer.html) उपकरण कैश निरीक्षण के लिए अनुमति देता है, जिसमें कैश डेटा निकालने के लिए फ़ोल्डर स्थान की आवश्यकता होती है। कैश के लिए मेटाडेटा में फ़ाइलनाम, डायरेक्टरी, एक्सेस गिनती, URL मूल, और कैश निर्माण, एक्सेस, संशोधन, और समाप्ति समय की टाइमस्टैम्प शामिल होती है।

### कुकीज़ प्रबंधन

कुकीज़ [IECookiesView](https://www.nirsoft.net/utils/iecookies.html) का उपयोग करके जांची जा सकती है, जिसमें मेटाडेटा नाम, URL, एक्सेस गिनती, और विभिन्न समय संबंधित विवरण शामिल होते हैं। स्थायी कुकीज़ `%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies` में संग्रहित होती हैं, जबकि सत्र कुकीज़ मेमोरी में रहती हैं।

### डाउनलोड विवरण

डाउनलोड मेटाडेटा [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) के माध्यम से उपलब्ध है, जिसमें विशिष्ट कंटेनर्स डेटा जैसे URL, फ़ाइल प्रकार, और डाउनलोड स्थान को संग्रहित करते हैं। भौतिक फ़ाइलें `%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory` के तहत पाई जा सकती हैं।

### ब्राउज़िंग इतिहास

ब्राउज़िंग इतिहास की समीक्षा करने के लिए, [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) का उपयोग किया जा सकता है, जिसमें इतिहास फ़ाइलों को निकालने के लिए स्थान और इंटरनेट एक्सप्लोरर के लिए विन्यास की आवश्यकता होती है। यहाँ मेटाडेटा में संशोधन और एक्सेस समय के साथ एक्सेस गिनती शामिल है। इतिहास फ़ाइलें `%userprofile%\Appdata\Local\Microsoft\Windows\History` में स्थित हैं।

### टाइप किए गए URLs

टाइप किए गए URLs और उनके उपयोग समय रजिस्ट्री में `NTUSER.DAT` के तहत `Software\Microsoft\InternetExplorer\TypedURLs` और `Software\Microsoft\InternetExplorer\TypedURLsTime` में संग्रहीत होते हैं, जो उपयोगकर्ता द्वारा दर्ज किए गए अंतिम 50 URL और उनके अंतिम इनपुट समय को ट्रैक करते हैं।

## Microsoft Edge

Microsoft Edge उपयोगकर्ता डेटा को `%userprofile%\Appdata\Local\Packages` में संग्रहित करता है। विभिन्न डेटा प्रकारों के लिए मार्ग हैं:

- **प्रोफ़ाइल पथ**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC`
- **इतिहास, कुकीज़, और डाउनलोड्स**: `C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat`
- **सेटिंग्स, बुकमार्क्स, और रीडिंग सूची**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb`
- **कैश**: `C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
- **अंतिम सक्रिय सत्र**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`

## Safari

Safari डेटा को `/Users/$User/Library/Safari` में संग्रहित करता है। मुख्य फ़ाइलें शामिल हैं:

- **History.db**: `history_visits` और
* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **Twitter** पर **फॉलो** करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PR जमा करके।
