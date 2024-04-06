# मूल फोरेंसिक मेथडोलॉजी

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस** चाहिए? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**दी पीएएस परिवार**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटीज़**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>

## छवि बनाना और माउंट करना

{% content-ref url="../../generic-methodologies-and-resources/basic-forensic-methodology/image-acquisition-and-mount.md" %}
[image-acquisition-and-mount.md](../../generic-methodologies-and-resources/basic-forensic-methodology/image-acquisition-and-mount.md)
{% endcontent-ref %}

## मैलवेयर विश्लेषण

यह **आवश्यक नहीं है कि आप छवि के साथ एक बार यह करने के लिए पहला कदम उठाएं**। लेकिन आप इस मैलवेयर विश्लेषण तकनीकों का उपयोग अलग-अलग तरीके से कर सकते हैं यदि आपके पास एक फ़ाइल, एक फ़ाइल-सिस्टम छवि, मेमोरी छवि, पीकैप... है, तो यह **कार्रवाई ध्यान में रखना अच्छा है**:

{% content-ref url="malware-analysis.md" %}
[malware-analysis.md](malware-analysis.md)
{% endcontent-ref %}

## छवि की जांच

यदि आपको एक उपकरण की **फोरेंसिक छवि** दी गई है तो आप **विभाजनों, फ़ाइल-सिस्टम** का उपयोग करना शुरू कर सकते हैं और पोटेंशियली **दिलचस्प फ़ाइलें पुनः प्राप्त** कर सकते हैं (हालांकि हटाई गई भी)। इसे सीखने के लिए:

{% content-ref url="partitions-file-systems-carving/" %}
[partitions-file-systems-carving](partitions-file-systems-carving/)
{% endcontent-ref %}

उपयोग किए गए ओएस और प्लेटफ़ॉर्म के आधार पर विभिन्न दिलचस्प आर्टिफैक्ट्स खोजने चाहिए:

{% content-ref url="windows-forensics/" %}
[windows-forensics](windows-forensics/)
{% endcontent-ref %}

{% content-ref url="linux-forensics.md" %}
[linux-forensics.md](linux-forensics.md)
{% endcontent-ref %}

{% content-ref url="docker-forensics.md" %}
[docker-forensics.md](docker-forensics.md)
{% endcontent-ref %}

## विशेष फ़ाइल-प्रकार और सॉफ़्टवेयर की गहन जांच

यदि आपके पास बहुत **संदेहास्पद फ़ाइल** है, तो **फ़ाइल-प्रकार और सॉफ़्टवेयर** पर निर्भर करता है कि कौन से **ट्रिक्स** उपयोगी हो सकते हैं।\
कुछ दिलचस्प ट्रिक्स सीखने के लिए निम्नलिखित पृष्ठ पढ़ें:

{% content-ref url="specific-software-file-type-tricks/" %}
[specific-software-file-type-tricks](specific-software-file-type-tricks/)
{% endcontent-ref %}

मैं विशेष रूप से पृष्ठ का उल्लेख करना चाहता हूँ:

{% content-ref url="specific-software-file-type-tricks/browser-artifacts.md" %}
[browser-artifacts.md](specific-software-file-type-tricks/browser-artifacts.md)
{% endcontent-ref %}

## मेमोरी डंप जांच

{% content-ref url="memory-dump-analysis/" %}
[memory-dump-analysis](memory-dump-analysis/)
{% endcontent-ref %}

## Pcap जांच

{% content-ref url="pcap-inspection/" %}
[pcap-inspection](pcap-inspection/)
{% endcontent-ref %}

## **एंटी-फोरेंसिक तकनीकें**

एंटी-फोरेंसिक तकनीकों का संभावित उपयोग ध्यान में रखें:

{% content-ref url="anti-forensic-techniques.md" %}
[anti-forensic-techniques.md](anti-forensic-techniques.md)
{% endcontent-ref %}

## धमकी हंटिंग

{% content-ref url="file-integrity-monitoring.md" %}
[file-integrity-monitoring.md](file-integrity-monitoring.md)
{% endcontent-ref %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस** चाहिए? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**दी पीएएस परिवार**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटीज़**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
