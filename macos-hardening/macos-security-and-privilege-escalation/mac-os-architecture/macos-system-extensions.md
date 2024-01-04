# macOS सिस्टम एक्सटेंशन्स

<details>

<summary><strong>AWS हैकिंग सीखें शुरुआत से लेकर एक्सपर्ट तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

## सिस्टम एक्सटेंशन्स / एंडपॉइंट सिक्योरिटी फ्रेमवर्क

कर्नेल एक्सटेंशन्स के विपरीत, **सिस्टम एक्सटेंशन्स यूजर स्पेस** में चलते हैं, न कि कर्नेल स्पेस में, जिससे एक्सटेंशन की खराबी के कारण सिस्टम क्रैश का जोखिम कम हो जाता है।

<figure><img src="../../../.gitbook/assets/image (1) (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

सिस्टम एक्सटेंशन्स के तीन प्रकार हैं: **DriverKit** एक्सटेंशन्स, **नेटवर्क** एक्सटेंशन्स, और **एंडपॉइंट सिक्योरिटी** एक्सटेंशन्स।

### **DriverKit एक्सटेंशन्स**

DriverKit कर्नेल एक्सटेंशन्स के लिए एक विकल्प है जो **हार्डवेयर सपोर्ट प्रदान करता है**। यह डिवाइस ड्राइवर्स (जैसे USB, Serial, NIC, और HID ड्राइवर्स) को यूजर स्पेस में चलाने की अनुमति देता है, न कि कर्नेल स्पेस में। DriverKit फ्रेमवर्क में कुछ I/O Kit क्लासेस के **यूजर स्पेस वर्जन्स** शामिल हैं, और कर्नेल सामान्य I/O Kit इवेंट्स को यूजर स्पेस में फॉरवर्ड करता है, इन ड्राइवर्स के लिए एक सुरक्षित वातावरण प्रदान करता है।

### **नेटवर्क एक्सटेंशन्स**

नेटवर्क एक्सटेंशन्स नेटवर्क व्यवहारों को कस्टमाइज़ करने की क्षमता प्रदान करते हैं। नेटवर्क एक्सटेंशन्स के कई प्रकार हैं:

* **App Proxy**: इसका उपयोग एक VPN क्लाइंट बनाने के लिए किया जाता है जो एक फ्लो-ओरिएंटेड, कस्टम VPN प्रोटोकॉल को लागू करता है। इसका मतलब है कि यह नेटवर्क ट्रैफिक को कनेक्शन (या फ्लो) के आधार पर संभालता है, न कि व्यक्तिगत पैकेट्स के आधार पर।
* **Packet Tunnel**: इसका उपयोग एक VPN क्लाइंट बनाने के लिए किया जाता है जो एक पैकेट-ओरिएंटेड, कस्टम VPN प्रोटोकॉल को लागू करता है। इसका मतलब है कि यह नेटवर्क ट्रैफिक को व्यक्तिगत पैकेट्स के आधार पर संभालता है।
* **Filter Data**: इसका उपयोग नेटवर्क "फ्लो" को फिल्टर करने के लिए किया जाता है। यह फ्लो स्तर पर नेटवर्क डेटा की निगरानी या संशोधन कर सकता है।
* **Filter Packet**: इसका उपयोग व्यक्तिगत नेटवर्क पैकेट्स को फिल्टर करने के लिए किया जाता है। यह पैकेट स्तर पर नेटवर्क डेटा की निगरानी या संशोधन कर सकता है।
* **DNS Proxy**: इसका उपयोग एक कस्टम DNS प्रोवाइडर बनाने के लिए किया जाता है। इसका उपयोग DNS अनुरोधों और प्रतिक्रियाओं की निगरानी या संशोधन के लिए किया जा सकता है।

## एंडपॉइंट सिक्योरिटी फ्रेमवर्क

एंडपॉइंट सिक्योरिटी एक फ्रेमवर्क है जो Apple द्वारा macOS में प्रदान किया गया है जो सिस्टम सिक्योरिटी के लिए एक सेट ऑफ APIs प्रदान करता है। इसका उद्देश्य **सिक्योरिटी वेंडर्स और डेवलपर्स द्वारा उत्पादों का निर्माण करने के लिए है जो सिस्टम गतिविधि की निगरानी और नियंत्रण कर सकते हैं** ताकि दुर्भावनापूर्ण गतिविधि की पहचान की जा सके और उसके खिलाफ सुरक्षा की जा सके।

यह फ्रेमवर्क **सिस्टम गतिविधि की निगरानी और नियंत्रण के लिए APIs का एक संग्रह प्रदान करता है**, जैसे कि प्रोसेस निष्पादन, फाइल सिस्टम इवेंट्स, नेटवर्क और कर्नेल इवेंट्स।

इस फ्रेमवर्क का मूल कर्नेल में लागू किया गया है, एक कर्नेल एक्सटेंशन (KEXT) के रूप में, जो **`/System/Library/Extensions/EndpointSecurity.kext`** पर स्थित है। इस KEXT में कई मुख्य घटक शामिल हैं:

* **EndpointSecurityDriver**: यह कर्नेल एक्सटेंशन के लिए "प्रवेश बिंदु" के रूप में कार्य करता है। यह OS और एंडपॉइंट सिक्योरिटी फ्रेमवर्क के बीच मुख्य बातचीत का बिंदु है।
* **EndpointSecurityEventManager**: यह घटक कर्नेल हुक्स को लागू करने के लिए जिम्मेदार है। कर्नेल हुक्स फ्रेमवर्क को सिस्टम कॉल्स को इंटरसेप्ट करके सिस्टम इवेंट्स की निगरानी करने की अनुमति देते हैं।
* **EndpointSecurityClientManager**: यह यूजर स्पेस क्लाइंट्स के साथ संचार का प्रबंधन करता है, यह ट्रैक रखता है कि कौन से क्लाइंट्स जुड़े हुए हैं और इवेंट नोटिफिकेशन्स प्राप्त करने की आवश्यकता है।
* **EndpointSecurityMessageManager**: यह यूजर स्पेस क्लाइंट्स को संदेश और इवेंट नोटिफिकेशन्स भेजता है।

एंडपॉइंट सिक्योरिटी फ्रेमवर्क द्वारा निगरानी की जा सकने वाली घटनाएं निम्नलिखित श्रेणियों में वर्गीकृत हैं:

* फाइल इवेंट्स
* प्रोसेस इवेंट्स
* सॉकेट इवेंट्स
* कर्नेल इवेंट्स (जैसे कि कर्नेल एक्सटेंशन को लोड/अनलोड करना या I/O Kit डिवाइस खोलना)

### एंडपॉइंट सिक्योरिटी फ्रेमवर्क आर्किटेक्चर

<figure><img src="../../../.gitbook/assets/image (3) (8).png" alt=""><figcaption></figcaption></figure>

**यूजर-स्पेस संचार** एंडपॉइंट सिक्योरिटी फ्रेमवर्क के साथ IOUserClient क्लास के माध्यम से होता है। दो अलग-अलग सबक्लासेस का उपयोग किया जाता है, कॉलर के प्रकार के आधार पर:

* **EndpointSecurityDriverClient**: इसके लिए `com.apple.private.endpoint-security.manager` एंटाइटलमेंट क
```bash
tccutil reset All
```
इस बायपास और संबंधित बायपास के बारे में **अधिक जानकारी** के लिए [#OBTS v5.0: "The Achilles Heel of EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI) की चर्चा देखें।

अंत में इसे ठीक किया गया था नए अनुमति **`kTCCServiceEndpointSecurityClient`** को सुरक्षा ऐप को देकर जिसे **`tccd`** द्वारा प्रबंधित किया जाता है ताकि `tccutil` उसकी अनुमतियों को साफ न करे जिससे वह चलने से रोका नहीं जा सके।

## संदर्भ

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपने हैकिंग ट्रिक्स शेयर करें।

</details>
