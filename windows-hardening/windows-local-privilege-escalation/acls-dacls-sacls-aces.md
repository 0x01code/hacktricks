# ACLs - DACLs/SACLs/ACEs

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=acls-dacls-sacls-aces) का उपयोग करें और दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित **कार्यप्रवाह** को आसानी से निर्मित करें और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=acls-dacls-sacls-aces" %}

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का **विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में **PRs सबमिट करके** अपने हैकिंग ट्रिक्स साझा करें।

</details>

## **पहुंच नियंत्रण सूची (ACL)**

एक पहुंच नियंत्रण सूची (ACL) एक क्रमबद्ध सेट के पहुंच नियंत्रण प्रविष्टियों (ACEs) से मिलकर बनी होती है जो एक वस्तु और उसकी गुणवत्ताओं के लिए संरक्षण निर्धारित करती है। सारांश में, ACL यह निर्धारित करती है कि एक दिए गए वस्तु पर किस सुरक्षा सिद्धांत (उपयोगकर्ता या समूह) द्वारा कौन से कार्रवाई स्वीकृत या निषेधित हैं।

दो प्रकार की ACL होती हैं:

* **विवेकाधीन पहुंच नियंत्रण सूची (DACL):** निर्धारित करती है कि किस उपयोगकर्ता और समूह को एक वस्तु तक पहुंच है या नहीं।
* **सिस्टम पहुंच नियंत्रण सूची (SACL):** एक वस्तु के परामर्श प्रयासों की लेखनी का नियंत्रण करती है।

एक फ़ाइल तक पहुंच करने की प्रक्रिया में, सिस्टम उपयोगकर्ता के पहुंच टोकन के खिलाफ वस्तु की सुरक्षा विवरणकर्ता की जांच करता है ताकि पहुंच प्रदान की जाए और उस पहुंच की व्यापकता, ACEs के आधार पर, निर्धारित की जा सके।

### **मुख्य घटक**

* **DACL:** एक वस्तु के लिए उपयोगकर्ताओं और समूहों को पहुंच अनुमतियों को देने या नकारने वाली ACEs शामिल करती है। यह मुख्य रूप से पहुंच अधिकारों को निर्धारित करने वाली मुख्य ACL है।
* **SACL:** वस्तुओं तक पहुंच की लेखनी के लिए उपयोग की जाती है, जहां ACEs सुरक्षा घटना लॉग में लॉग करने के प्रकार को परिभाषित करती हैं। यह अनधिकृत पहुंच प्रयासों का पता लगाने या पहुंच समस्याओं का समाधान करने के लिए अमूल्य हो सकता है।

### **ACLs के साथ सिस्टम इंटरेक्शन**

प्रत्येक उपयोगकर्ता सत्र से एक पहुंच टोकन जुड़ा होता है जिसमें उस सत्र से संबंधित सुरक्षा सूचना, उपयोगकर्ता, समूह पहचान और विशेषाधिकार शामिल होते हैं। यह टोकन एक लॉगऑन SID भी शामिल करता है जो सत्र को अद्वितीयता से पहचानता है।

स्थानीय सुरक्षा प्राधिकरण (LSASS) ACEs की जांच करके वस्तुओं के प्रति पहुंच प्रारंभ करता है जो सुरक्षा सिद्धांत की कोशिश कर रहे सुरक्षा सिद्धांत के साथ मेल खाते हैं। कोई संबंधित ACEs नहीं मिलते हैं तो पहुंच तुरंत प्रदान की जाती है। अन्यथा, LSASS उपयोगकर्ता के SID के साथ ACEs की तुलना करता है तक पहुंच पात्रता निर्धारित करने के लिए।

### **संक्षेपित प्रक्रिया**

* **ACLs:** DACLs के माध्यम से पहुंच अनुमतियाँ परिभाषित करती हैं और SACLs के माध्यम से लेखनी नियम परिभाषित करती हैं।
* **पहुंच टोकन:** एक सत्र के लिए उपयोगकर्ता, समूह और विशेषाधिकार सूचना शामिल करता है।
* **पहुंच निर्णय:** DACL ACEs को पहुंच टोकन के साथ तुलना करके निर्णय लिया जाता है; लेखनी नियमों का लेखन के लिए उपयोग किया जाता है।

### ACEs

**तीन मुख्य प्रकार की पहुंच नियंत्रण प्रविष्टियाँ (ACEs)** होती हैं:

* **पहुंच निषेधित ACE**: इस ACE निर्दिष्ट उपयोगकर्ताओं या समूहों के लिए एक वस्तु तक पहुंच को स्पष्ट रूप से निषेधित करता है (DACL में)।
* **पहुंच अनुमत ACE**: यह ACE निर्दिष्ट उपयोगकर्ताओं या समूहों के लिए एक वस्तु तक पहुंच को स्पष्ट रूप से प्रदान करता है (DACL में)।
* **सिस्टम ऑडिट ACE**: एक सिस्टम पहुंच नियंत्रण सूची (SACL) में स्थित, यह ACE उपयोगकर्ताओं या समूहों द्वारा एक वस्तु तक पहुंच प्रयासों पर ऑडिट लॉग उत्पन्न करने के लिए जिम्मेदार है। यह दस्तावेज़ करता है कि पहुंच की गई थी या नहीं और पहुंच का प्रकार।

प्रत्येक ACE में **चार महत्वपूर्ण घटक** होते हैं:

1. उपयोगकर्ता या समूह का **सुरक्षा पहचानकर्ता (SID)** (या उनका प्रमुख नाम एक ग्राफिकल प्रतिनिधित्व में)।
2. ACE प्रकार (पहुंच निषेधित, अनुमत या सिस्टम ऑडिट) को पहचानने वाला **ध्वज**।
3. विरासत ध्वज जो निर्धारित करते हैं कि बालक वस्तु अपने मूल से ACE को विरासत में ले सकती हैं।
4. एक [**पहुंच मास्क**](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-dtyp/7a53f60e-e730-4dfe-bbe9-b21b62eb790b?redirectedfrom=MSDN), एक 32-बिट मान जो वस्तु के प्रदत्त अधिकारों को निर्धारित करता है।

पहुंच निर्धारण को क्रमबद्ध रूप से प्रत्येक ACE की जांच करके किया जाता है जब तक:

* एक **पहुंच निषेधित ACE** उपयोगकर्ता टोकन में पहुंच करने वाले विश्वासी को अनुरोधित अधिकारों को स्पष्ट रूप से निषेधित करता है।
* **पहुंच अनुमत ACE(s)** उपयोगकर्ता टोकन में विश्वासी को सभी अनुरोधित अधिकारों को स्पष्ट रूप स
### GUI उदाहरण

[**यहाँ से उदाहरण**](https://secureidentity.se/acl-dacl-sacl-and-the-ace/)

यह एक फ़ोल्डर के क्लासिक सुरक्षा टैब है जो ACL, DACL और ACEs दिखाता है:

![http://secureidentity.se/wp-content/uploads/2014/04/classicsectab.jpg](../../.gitbook/assets/classicsectab.jpg)

अगर हम **एडवांस्ड बटन** पर क्लिक करेंगे तो हमें इनहेरिटेंस जैसे अधिक विकल्प मिलेंगे:

![http://secureidentity.se/wp-content/uploads/2014/04/aceinheritance.jpg](../../.gitbook/assets/aceinheritance.jpg)

और अगर आप एक सुरक्षा प्रिंसिपल जोड़ते या संपादित करते हैं:

![http://secureidentity.se/wp-content/uploads/2014/04/editseprincipalpointers1.jpg](../../.gitbook/assets/editseprincipalpointers1.jpg)

और अंत में हमारे पास Auditing टैब में SACL है:

![http://secureidentity.se/wp-content/uploads/2014/04/audit-tab.jpg](../../.gitbook/assets/audit-tab.jpg)

### सरलिता से पहुंच नियंत्रण का स्पष्टीकरण

संसाधनों तक पहुंच को प्रबंधित करते समय, जैसे कि एक फ़ोल्डर, हम सूची और नियम का उपयोग करते हैं जिसे पहुंच नियंत्रण सूची (ACLs) और पहुंच नियंत्रण प्रविष्टियाँ (ACEs) कहा जाता है। ये निर्धारित करते हैं कि कौन किस डेटा तक पहुंच सकता है या नहीं।
#### Denying Access to a Specific Group

Imagine you have a folder named Cost, and you want everyone to access it except for a marketing team. By setting up the rules correctly, we can ensure that the marketing team is explicitly denied access before allowing everyone else. This is done by placing the rule to deny access to the marketing team before the rule that allows access to everyone.

#### Allowing Access to a Specific Member of a Denied Group

Let's say Bob, the marketing director, needs access to the Cost folder, even though the marketing team generally shouldn't have access. We can add a specific rule (ACE) for Bob that grants him access, and place it before the rule that denies access to the marketing team. This way, Bob gets access despite the general restriction on his team.

#### Understanding Access Control Entries

ACEs are the individual rules in an ACL. They identify users or groups, specify what access is allowed or denied, and determine how these rules apply to sub-items (inheritance). There are two main types of ACEs:

* **Generic ACEs**: These apply broadly, affecting either all types of objects or distinguishing only between containers (like folders) and non-containers (like files). For example, a rule that allows users to see the contents of a folder but not to access the files within it.
* **Object-Specific ACEs**: These provide more precise control, allowing rules to be set for specific types of objects or even individual properties within an object. For instance, in a directory of users, a rule might allow a user to update their phone number but not their login hours.

Each ACE contains important information like who the rule applies to (using a Security Identifier or SID), what the rule allows or denies (using an access mask), and how it's inherited by other objects.

#### Key Differences Between ACE Types

* **Generic ACEs** are suitable for simple access control scenarios, where the same rule applies to all aspects of an object or to all objects within a container.
* **Object-Specific ACEs** are used for more complex scenarios, especially in environments like Active Directory, where you might need to control access to specific properties of an object differently.

In summary, ACLs and ACEs help define precise access controls, ensuring that only the right individuals or groups have access to sensitive information or resources, with the ability to tailor access rights down to the level of individual properties or object types.

### Access Control Entry Layout
| ACE क्षेत्र   | विवरण                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| प्रकार        | ACE के प्रकार को दर्शाने वाला ध्वज। Windows 2000 और Windows Server 2003 छह प्रकार के ACE का समर्थन करते हैं: तीन सामान्य ACE प्रकार जो सभी सुरक्षित वस्तुओं से जुड़े होते हैं। तीन वस्तु-विशिष्ट ACE प्रकार जो सक्रिय निर्देशिका वस्तुओं के लिए हो सकते हैं।                                                                                                                                                                                                                                                            |
| झंडे       | विरासत और ऑडिटिंग को नियंत्रित करने वाले बिट झंडे।                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| आकार        | ACE के लिए आवंटित मेमोरी बाइटों की संख्या।                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| पहुंच चिह्न | वस्तु के लिए पहुंच अधिकारों का मान जिसके बिट वस्तु के लिए पहुंच अधिकारों के लिए होते हैं। बिट को या तो चालू या बंद किया जा सकता है, लेकिन सेटिंग का अर्थ ACE प्रकार पर निर्भर करता है। उदाहरण के लिए, यदि उस बिट को जो पढ़ने की अनुमति देता है उसे चालू किया जाता है, और ACE प्रकार निषेध है, तो ACE वस्तु की अनुमतियों को पढ़ने की अधिकार को निषेधित करता है। यदि वही बिट चालू है लेकिन ACE प्रकार अनुमति है, तो ACE वस्तु की अनुमतियों को पढ़ने की अधिकार प्रदान करता है। पहुंच चिह्न के अधिक विवरण अगले तालिका में दिखाई देते हैं। |
| SID         | इस ACE द्वारा नियंत्रित या मॉनिटर किए जाने वाले उपयोगकर्ता या समूह की पहचान करता है।                                                                                                                                                                                                                                                                                                                                                                                                                                 |

### पहुंच चिह्न लेआउट

| बिट (सीमा) | अर्थ                            | विवरण/उदाहरण                       |
| ----------- | ---------------------------------- | ----------------------------------------- |
| 0 - 15      | वस्तु-विशिष्ट पहुंच अधिकार      | डेटा पढ़ें, क्रियान्वयन, डेटा जोड़ें           |
| 16 - 22     | मानक पहुंच अधिकार             | हटाएं, लिखें ACL, मालिक लिखें            |
| 23          | सुरक्षा ACL तक पहुंच सकता है            |                                           |
| 24 - 27     | आरक्षित                           |                                           |
| 28          | सार्वजनिक सभी (पढ़ें, लिखें, क्रियान्वयन) | सब कुछ नीचे                          |
| 29          | सार्वजनिक क्रियान्वयन                    | किसी कार्यक्रम को निष्पादित करने के लिए सभी चीजें |
| 30          | सार्वजनिक लिखें                      | एक फ़ाइल में लिखने के लिए सभी चीजें   |
| 31          | सार्वजनिक पढ़ें                       | एक फ़ाइल को पढ़ने के लिए सभी चीजें       |

## संदर्भ

* [https://www.ntfs.com/ntfs-permissions-acl-use.htm](https://www.ntfs.com/ntfs-permissions-acl-use.htm)
* [https://secureidentity.se/acl-dacl-sacl-and-the-ace/](https://secureidentity.se/acl-dacl-sacl-and-the-ace/)
* [https://www.coopware.in2.info/\_ntfsacl\_ht.htm](https://www.coopware.in2.info/\_ntfsacl\_ht.htm)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन देखना चाहते हैं **HackTricks** में या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PR जमा करके।

</details>

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest का उपयोग करें**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=acls-dacls-sacls-aces) और दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित **कार्यप्रणालियों** को आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=acls-dacls-sacls-aces" %}
