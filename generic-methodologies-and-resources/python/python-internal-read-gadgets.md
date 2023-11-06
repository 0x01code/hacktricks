# Python आंतरिक पठन उपकरण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## मूलभूत जानकारी

[**Python Format Strings**](bypass-python-sandboxes/#python-format-string) या [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) जैसी विभिन्न कमजोरियाँ आपको **Python आंतरिक डेटा को पढ़ने की अनुमति देती हैं लेकिन आपको कोड को नहीं चलाने देतीं**। इसलिए, पेंटेस्टर को इन पठन अनुमतियों का उपयोग करके **संवेदनशील विशेषाधिकार प्राप्त करने और कमजोरी को बढ़ाने के लिए** उपयोग करना होगा।

### Flask - गुप्त कुंजी पढ़ें

एक Flask एप्लिकेशन का मुख्य पृष्ठ **`app`** वैश्विक ऑब्जेक्ट होगा जहां इस **गुप्त को विन्यासित** किया जाता है।
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
इस मामले में, यह संभव है कि इस ऑब्जेक्ट तक पहुंचा जा सके, बस किसी गैजेट का उपयोग करके **ग्लोबल ऑब्जेक्ट्स तक पहुंचने** के लिए [**Bypass Python sandboxes page**](bypass-python-sandboxes/) से।

जब **दुर्बलता एक अलग python फ़ाइल में होती है**, तो आपको फ़ाइलों को चलने के लिए एक गैजेट की आवश्यकता होती है ताकि मुख्य फ़ाइल तक पहुंच सकें और **ग्लोबल ऑब्जेक्ट `app.secret_key` तक पहुंचें** और Flask सीक्रेट कुंजी को बदलें और [**इस कुंजी को जानते हुए अधिकारों को बढ़ाएं**](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign)।

इस तरह का एक पेलोड जैसा कि [इस व्राइटअप से](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

इस payload का उपयोग करें ताकि आप `app.secret_key` (आपके ऐप में नाम अलग हो सकता है) को बदल सकें और नए और अधिक विशेषाधिकार flask cookies को साइन करने के लिए उपयोग कर सकें।

### Werkzeug - machine\_id और node uuid

[**इस writeup के payload का उपयोग करके**](https://vozec.fr/writeups/tweedle-dum-dee/) आप **machine\_id** और **uuid** नोड तक पहुंच सकेंगे, जो **मुख्य रहस्य** हैं जिनकी आपको आवश्यकता होगी [**Werkzeug pin उत्पन्न करने**](../../network-services-pentesting/pentesting-web/werkzeug.md) के लिए जिसका उपयोग आप `/console` में पायथन कंसोल तक पहुंचने के लिए कर सकते हैं अगर **debug mode सक्षम है:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
ध्यान दें कि आप **`app.py`** के **सर्वर के स्थानिक पथ** को प्राप्त कर सकते हैं वेब पेज में कुछ **त्रुटि** उत्पन्न करके जो आपको **पथ देगी**।
{% endhint %}

यदि संकटग्रस्तता किसी अलग python फ़ाइल में है, तो मुख्य python फ़ाइल से ऑब्जेक्ट तक पहुंचने के लिए पिछले Flask ट्रिक की जांच करें।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
