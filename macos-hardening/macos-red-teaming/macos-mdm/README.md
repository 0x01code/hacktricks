# macOS MDM

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को PR जमा करके।**

</details>

## मूलभूत

### MDM (मोबाइल डिवाइस प्रबंधन) क्या है?

[मोबाइल डिवाइस प्रबंधन](https://en.wikipedia.org/wiki/Mobile\_device\_management) (MDM) एक प्रौद्योगिकी है जो आमतौर पर मोबाइल फोन, लैपटॉप, डेस्कटॉप और टैबलेट जैसे **अंतयः-उपयोगकर्ता कंप्यूटिंग उपकरणों** का प्रशासन करने के लिए उपयोग की जाती है। Apple प्लेटफॉर्मों जैसे iOS, macOS और tvOS के मामले में, इसका अर्थ होता है कि व्यवस्थापकों द्वारा इन उपकरणों का प्रबंधन करने के लिए उपयोग किए जाने वाले विशेषताओं, API और तकनीकों का एक विशेष सेट होता है। MDM के माध्यम से उपकरणों का प्रबंधन करने के लिए एक संगत वाणिज्यिक या ओपन-सोर्स MDM सर्वर की आवश्यकता होती है जो [MDM Protocol](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf) के समर्थन को लागू करता है।

* **केंद्रीयकृत उपकरण प्रबंधन** को प्राप्त करने का एक तरीका
* MDM प्रोटोकॉल के समर्थन को लागू करने वाले एक MDM सर्वर की आवश्यकता होती है
* MDM सर्वर एमडीएम कमांड भेज सकता है, जैसे कि रिमोट वाइप या "इस कॉन्फ़िगरेशन को स्थापित करें"

### मूलभूत DEP (डिवाइस एनरोलमेंट प्रोग्राम) क्या है?

[डिवाइस एनरोलमेंट प्रोग्राम](https://www.apple.com/business/site/docs/DEP\_Guide.pdf) (DEP) एक सेवा है जिसे Apple द्वारा प्रदान किया जाता है जो iOS, macOS और tvOS उपकरणों के **मोबाइल डिवाइस प्रबंधन (MDM) एनरोलमेंट** को **सरल** बनाती है और **शून्य-स्पर्श विन्यास** प्रदान करती है। यह अधिक पारंपरिक वितरण विधियों के विपरीत है, जो उपयोगकर्ता या प्रशासक को उपकरण को कॉन्फ़िगर करने के लिए कार्रवाई करने या MDM सर्वर के साथ मैन्युअल एनरोलमेंट करने की आवश्यकता होती है, DEP इस प्रक्रिया को बूटस्ट्रैप करने का प्रयास करता है, **जिससे उपयोगकर्ता एक नया Apple उपकरण खोल सकता है और इसे संगठन में उपयोग के लिए तत्पर किया जा सकता है**।

प्रशासक अपने संगठन के MDM सर्वर में उपकरणों को स्वचालित रूप से एनरोल करने के लिए DEP का उपयोग कर सकते हैं। एक बार उपकरण एनरोल हो जाता है, **बहुत से मामलों में इसे संगठन द्वारा स्वीकृत "विश्वसनी
## सीरियल नंबर

2010 के बाद निर्मित Apple उपकरणों में आमतौर पर **12-अक्षरीय अल्फान्यूमेरिक** सीरियल नंबर होते हैं, जिसमें **पहले तीन अंक निर्माण स्थान** को प्रतिष्ठानित करते हैं, अगले **दो** निर्माण के **साल** और **सप्ताह** को दर्शाते हैं, अगले **तीन** अंक एक **अद्वितीय पहचानकर्ता** प्रदान करते हैं, और **अंतिम** **चार** अंक मॉडल नंबर को प्रतिष्ठानित करते हैं।

{% content-ref url="macos-serial-number.md" %}
[macos-serial-number.md](macos-serial-number.md)
{% endcontent-ref %}

## पंजीकरण और प्रबंधन के लिए चरण

1. उपकरण रिकॉर्ड निर्माण (रिसेलर, Apple): नए उपकरण के लिए रिकॉर्ड बनाया जाता है
2. उपकरण रिकॉर्ड आवंटन (ग्राहक): उपकरण को एक MDM सर्वर को आवंटित किया जाता है
3. उपकरण रिकॉर्ड सिंक (MDM विक्रेता): MDM उपकरण रिकॉर्ड को सिंक करता है और DEP प्रोफ़ाइल को Apple को पुश करता है
4. DEP चेक-इन (उपकरण): उपकरण को उसकी DEP प्रोफ़ाइल मिलती है
5. प्रोफ़ाइल प्राप्ति (उपकरण)
6. प्रोफ़ाइल स्थापना (उपकरण) a. MDM, SCEP और रूट CA payloads सहित
7. MDM आदेश जारी करना (उपकरण)

![](<../../../.gitbook/assets/image (564).png>)

फ़ाइल `/Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/System/Library/PrivateFrameworks/ConfigurationProfiles.framework/ConfigurationProfiles.tbd` उन फ़ंक्शन्स को निर्यात करती है जो पंजीकरण प्रक्रिया के **उच्च स्तरीय "चरण"** के रूप में मान्य हो सकते हैं।

### चरण 4: DEP चेक-इन - सक्रियण रिकॉर्ड प्राप्त करना

यह प्रक्रिया तब होती है जब एक **उपयोगकर्ता पहली बार Mac को बूट करता है** (या पूरी तरह से मिटाने के बाद)

![](<../../../.gitbook/assets/image (568).png>)

या `sudo profiles show -type enrollment` को निष्पादित करने पर

* **उपकरण के लिए DEP सक्षम है या नहीं** यह निर्धारित करें
* सक्रियण रिकॉर्ड DEP "प्रोफ़ाइल" के लिए आंतरिक नाम है
* इंटरनेट से जुड़ने पर शुरू होता है
* **`CPFetchActivationRecord`** द्वारा चलाया जाता है
* **`cloudconfigurationd`** द्वारा क्रियान्वित किया जाता है XPC के माध्यम से। **"सेटअप सहायक**" (जब उपकरण पहली बार बूट होता है) या **`profiles`** कमांड इस डेमन को प्राप्त करने के लिए **संपर्क करेंगे**।
* LaunchDaemon (हमेशा रूप में चलता है जैसे कि रूट)

इसे **`MCTeslaConfigurationFetcher`** द्वारा प्रदर्शित किए जाने वाले कुछ चरणों का पालन करते हुए सक्रियण रिकॉर्ड प्राप्त करने के लिए कुछ कदम लिए जाते हैं। इस प्रक्रिया में एक एन्क्रिप्शन उपयोग किया जाता है जिसे **अब्सिंथ** कहा जाता है।

1. **प्रमाणपत्र** प्राप्त करें
1. GET [https://iprofiles.apple.com/resource/certificate.cer](https://iprofiles.apple.com/resource/certificate.cer)
2. प्रमाणपत्र से राज्य की शुरुआत करें (**`NACInit`**)
1. विभिन्न उपकरण-विशिष्ट डेटा का उपयोग करें (जैसे कि **`IOKit`** के माध्यम से **सीरियल नंबर**)
3. **सत्र कुंजी** प्राप्त करें
1. POST [https://iprofiles.apple.com/session](https://iprofiles.apple.com/session)
4. सत्र स्थापित करें (**`NACKeyEstablishment`**)
5. अनुरोध करें
1. डेटा `{ "action": "RequestProfileConfiguration", "sn": "" }` भेजकर [https://iprofiles.apple.com/macProfile](https://iprofiles.apple.com/macProfile) पर POST करें
2. JSON पेलोड Absinthe (**`NACSign`**) का उपयोग करके एन्क्रिप्ट किया जाता है
3. सभी अनुरोध HTTPs के माध्यम से, अंतर्निहित रूट प्रमाणपत्रों का उपयोग किया जाता है

![](<../../../.gitbook/assets/image (566).png>)

प्रतिक्रिया एक JSON शब्दकोश है जिसमें कुछ महत्वपूर्ण डेटा है जैसे:

* **url**: सक्रियण प्रोफ़ाइल के लिए MDM विक्रेता होस्ट का URL
* **anchor-certs**: विश्वसनीय एंकर के रूप में उपयोग होने वाले DER प्रमाणपत्रों का एक सरणी

### **चरण 5: प्रोफ़ाइल प्राप्ति**

![](<../../../.gitbook/assets/image (567).png>)

* DEP प्रोफ़ाइल में दिए गए **url पर अनुरोध भेजा जाता है**।
* यदि उपलब्ध
### **चरण 7: MDM कमांड के लिए सुनना**

* MDM चेक-इन के बाद, विक्रेता **APNs का उपयोग करके पुश सूचनाएं जारी कर सकता है**
* प्राप्ति के बाद, **`mdmclient`** द्वारा हैंडल किया जाता है
* MDM कमांड के लिए पोल करने के लिए, एक अनुरोध ServerURL पर भेजा जाता है
* पहले से स्थापित MDM पेलोड का उपयोग किया जाता है:
* पिनिंग अनुरोध के लिए **`ServerURLPinningCertificateUUIDs`**
* TLS क्लाइंट प्रमाणपत्र के लिए **`IdentityCertificateUUID`**

## हमले

### अन्य संगठनों में उपकरणों को नामांकित करना

पहले ही टिप्पणी में कहा गया है कि किसी संगठन में एक उपकरण को नामांकित करने के लिए **केवल उस संगठन के एक सीरियल नंबर की आवश्यकता होती है**। एक बार उपकरण नामांकित होने के बाद, कई संगठन नए उपकरण पर संवेदनशील डेटा स्थापित करेंगे: प्रमाणपत्र, एप्लिकेशन, WiFi पासवर्ड, VPN कॉन्फ़िगरेशन [और इत्यादि](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf)।\
इसलिए, यदि नामांकन प्रक्रिया सही ढंग से संरक्षित नहीं है, तो यह हमलावर्धक एंट्रीपॉइंट हो सकता है:

{% content-ref url="enrolling-devices-in-other-organisations.md" %}
[enrolling-devices-in-other-organisations.md](enrolling-devices-in-other-organisations.md)
{% endcontent-ref %}

## **संदर्भ**

* [https://www.youtube.com/watch?v=ku8jZe-MHUU](https://www.youtube.com/watch?v=ku8jZe-MHUU)
* [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स को हमें PR के माध्यम से साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके।**

</details>
