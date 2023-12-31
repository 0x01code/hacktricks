# Python आंतरिक रीड गैजेट्स

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>

## मूल जानकारी

विभिन्न कमजोरियां जैसे कि [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) या [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) आपको **python आंतरिक डेटा पढ़ने की अनुमति दे सकती हैं लेकिन कोड निष्पादित करने की नहीं**. इसलिए, एक पेंटेस्टर को इन पढ़ने की अनुमतियों का सबसे अधिक उपयोग करना होगा ताकि **संवेदनशील विशेषाधिकार प्राप्त कर सकें और कमजोरी को बढ़ा सकें**.

### Flask - सीक्रेट की पढ़ें

Flask एप्लिकेशन का मुख्य पृष्ठ संभवतः **`app`** ग्लोबल ऑब्जेक्ट होगा जहां यह **सीक्रेट कॉन्फ़िगर किया गया है**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
इस मामले में, आप इस ऑब्जेक्ट तक सिर्फ किसी भी गैजेट का उपयोग करके **ग्लोबल ऑब्जेक्ट्स तक पहुँचने** के लिए [**Bypass Python sandboxes पेज**](bypass-python-sandboxes/) से कर सकते हैं।

जिस स्थिति में **वल्नरेबिलिटी एक अलग पायथन फाइल में हो**, आपको फाइलों को ट्रैवर्स करने के लिए एक गैजेट की आवश्यकता होती है ताकि मुख्य फाइल तक पहुँच सकें और **ग्लोबल ऑब्जेक्ट `app.secret_key` तक पहुँच** सकें ताकि Flask सीक्रेट की को बदल सकें और [**इस की को जानकर अधिकारों को बढ़ा सकें**](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign)।

इस तरह का एक पेलोड [इस राइटअप से](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

इस पेलोड का उपयोग करके **`app.secret_key` को बदलें** (आपके ऐप में नाम अलग हो सकता है) ताकि आप नए और अधिक अधिकारों वाले flask कुकीज़ को साइन कर सकें।

### Werkzeug - machine\_id और node uuid

[**इस लेख से इन पेलोड का उपयोग करके**](https://vozec.fr/writeups/tweedle-dum-dee/) आप **machine\_id** और **uuid** नोड तक पहुँच सकते हैं, जो कि [**Werkzeug पिन उत्पन्न करने के लिए आवश्यक मुख्य रहस्य**](../../network-services-pentesting/pentesting-web/werkzeug.md) हैं, जिसका उपयोग करके आप `/console` में पायथन कंसोल तक पहुँच सकते हैं अगर **डीबग मोड सक्षम है:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
ध्यान दें कि आप **`app.py` के सर्वर के स्थानीय पथ** को प्राप्त कर सकते हैं जो वेब पेज में कुछ **त्रुटि** उत्पन्न करके आपको पथ **दिखाएगा**।
{% endhint %}

यदि समस्या एक अलग python फ़ाइल में है, तो मुख्य python फ़ाइल से ऑब्जेक्ट्स तक पहुँचने के लिए पिछली Flask ट्रिक की जाँच करें।

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का अन्य तरीकों से समर्थन करें:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
