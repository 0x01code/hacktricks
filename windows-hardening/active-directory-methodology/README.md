# एक्टिव डायरेक्टरी मेथडोलॉजी

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का** **अनुसरण** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) **और** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **github repos में**।

</details>

## मौलिक अवलोकन

**एक्टिव डायरेक्टरी** एक मौलिक प्रौद्योगिकी के रूप में काम करती है, जो **नेटवर्क प्रशासकों** को नेटवर्क के भीतर **डोमेन**, **उपयोगकर्ता**, और **ऑब्ज
```bash
./kerbrute_linux_amd64 userenum -d lab.ropnop.com --dc 10.10.10.10 usernames.txt #From https://github.com/ropnop/kerbrute/releases

nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='DOMAIN'" <IP>
Nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm='<domain>',userdb=/root/Desktop/usernames.txt <IP>

msf> use auxiliary/gather/kerberos_enumusers

crackmapexec smb dominio.es  -u '' -p '' --users | awk '{print $4}' | uniq
```
* **OWA (Outlook Web Access) सर्वर**

अगर आप नेटवर्क में इन सर्वरों में से एक पाते हैं तो आप इसके खिलाफ **उपयोगकर्ता गणना भी कर सकते हैं**। उदाहरण के लिए, आप [**MailSniper**](https://github.com/dafthack/MailSniper) टूल का उपयोग कर सकते हैं:
```bash
ipmo C:\Tools\MailSniper\MailSniper.ps1
# Get info about the domain
Invoke-DomainHarvestOWA -ExchHostname [ip]
# Enumerate valid users from a list of potential usernames
Invoke-UsernameHarvestOWA -ExchHostname [ip] -Domain [domain] -UserList .\possible-usernames.txt -OutFile valid.txt
# Password spraying
Invoke-PasswordSprayOWA -ExchHostname [ip] -UserList .\valid.txt -Password Summer2021
# Get addresses list from the compromised mail
Get-GlobalAddressList -ExchHostname [ip] -UserName [domain]\[username] -Password Summer2021 -OutFile gal.txt
```
{% hint style="warning" %}
आप [**इस github रेपो**](https://github.com/danielmiessler/SecLists/tree/master/Usernames/Names) में उपयोगकर्ता नामों की सूचियाँ पा सकते हैं और इसमें ([**आंकड़े के अनुसार संभावित उपयोगकर्ता नाम**](https://github.com/insidetrust/statistically-likely-usernames))।

हालांकि, आपको **कंपनी में काम करने वाले लोगों के नाम** का पता होना चाहिए जो आपने इससे पहले किया होना चाहिए। नाम और उपनाम के साथ आप [**namemash.py**](https://gist.github.com/superkojiman/11076951) स्क्रिप्ट का उपयोग करके संभावित मान्य उपयोगकर्ता नाम उत्पन्न कर सकते हैं।
{% endhint %}

### एक या कई उपयोगकर्ता नामों को जानना

ठीक है, तो आपको पहले से ही एक मान्य उपयोगकर्ता नाम पता है लेकिन कोई पासवर्ड नहीं है... तो कोशिश करें:

* [**ASREPRoast**](asreproast.md): यदि किसी उपयोगकर्ता के पास विशेषता _DONT\_REQ\_PREAUTH_ नहीं है तो आप उस उपयोगकर्ता के लिए एक AS\_REP संदेश का अनुरोध कर सकते हैं जो उस उपयोगकर्ता के पासवर्ड के एक उत्पादन द्वारा एन्क्रिप्ट किए गए कुछ डेटा शामिल करेगा।
* [**पासवर्ड स्प्रेइंग**](password-spraying.md): सबसे **सामान्य पासवर्ड** का प्रयास करें प्रत्येक खोजे गए उपयोगकर्ताओं के साथ, शायद कोई उपयोगकर्ता एक बुरा पासवर्ड उपयोग कर रहा हो (ध्यान रखें पासवर्ड नीति को!)।
* ध्यान दें कि आप उपयोगकर्ताओं के मेल सर्वर में पहुंचने के लिए **OWA सर्वरों को स्प्रे कर सकते हैं**।

{% content-ref url="password-spraying.md" %}
[password-spraying.md](password-spraying.md)
{% endcontent-ref %}

### LLMNR/NBT-NS Poisoning

आप **नेटवर्क** के कुछ प्रोटोकॉल को **पॉइजन करके** कुछ चुनौती **हैश** प्राप्त कर सकते हैं:

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

### NTML Relay

यदि आपने सक्रिय निर्देशिका की जांच कर ली है तो आपके पास **अधिक ईमेल और नेटवर्क की बेहतर समझ** होगी। आप **NTML [**रिले हमलों**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack) को बलपूर्वक प्राप्त करने के लिए सक्षम हो सकते हैं ताकि आप AD वातावरण तक पहुंच सकें।

### NTLM Creds चुराना

यदि आप **अन्य पीसी या शेयर** तक पहुंच सकते हैं **शून्य या मेहमान उपयोगकर्ता** के साथ तो आप फ़ाइलें डाल सकते हैं (जैसे एक SCF फ़ाइल) जो यदि किसी प्रकार से एक्सेस किया जाएगा तो वह आपके खिलाफ एक NTML प्रमाणीकरण को ट्रिगर करेगा ताकि आप उसे चुरा सकें:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

## सक्रिय निर्देशिका की जांच करना प्रमाणों/सत्र के साथ

इस चरण के लिए आपको **मान्य डोमेन खाते के प्रमाण या सत्र** को संक्रमित करना होगा। यदि आपके पास कुछ मान्य प्रमाण हैं या एक डोमेन उपयोगकर्ता के रूप में एक शैली है, **तो आपको याद रखना चाहिए कि पहले दिए गए विकल्प अब भी अन्य उपयोगकर्ताओं को संक्रमित करने के लिए विकल्प हैं**।

प्रमाणित जांच शुरू करने से पहले आपको यह जानना चाहिए कि **कर्बेरोस डबल हॉप समस्या क्या है**।

{% content-ref url="kerberos-double-hop-problem.md" %}
[kerberos-double-hop-problem.md](kerberos-double-hop-problem.md)
{% endcontent-ref %}

### जांच

एक खाते को संक्रमित करना एक **बड़ा कदम है पूरे डोमेन को संक्रमित करने के लिए**, क्योंकि आप **सक्रिय निर्देशिका जांच** शुरू कर सकेंगे:

[**ASREPRoast**](asreproast.md) के संदर्भ में अब आप हर संभावित वंशावली उपयोगकर्ता पा सकते हैं, और [**पासवर्ड स्प्रेइंग**](password-spraying.md) के संदर्भ में आप **सभी उपयोगकर्ता नामों की सूची** प्राप्त कर सकते हैं और संक्रमित खाते का पासवर्ड, खाली पासवर्ड और नए उम्मीदवार पासवर्ड का प्रयास कर सकते हैं।

* आप [**बुनियादी जांच करने के लिए CMD का उपयोग कर सकते हैं**](../basic-cmd-for-pentesters.md#domain-info)
* आप [**पावरशेल का उपयोग कर सकते हैं जांच के लिए**](../basic-powershell-for-pentesters/) जो गुप्तचर होगा
* आप [**पावरव्यू का उपयोग कर सकते हैं**](../basic-powershell-for-pentesters/powerview.md) अधिक विस्तृत जानकारी निकालने के लिए
* एक और अद्भुत उपकरण सक्रिय निर्देशिका में जांच के लिए है [**BloodHound**](bloodhound.md)। यह **बहुत गुप्तचर नहीं है** (आपके उपयोग के संग्रहण विध
```bash
## List all tickets (if not admin, only current user tickets)
.\Rubeus.exe triage
## Dump the interesting one by luid
.\Rubeus.exe dump /service:krbtgt /luid:<luid> /nowrap
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))
```
### NTML Relay

यदि आपने सक्रिय निर्देशिका की जांच करने में सफलता प्राप्त की है तो आपके पास **अधिक ईमेल और नेटवर्क की बेहतर समझ** होगी। आप **NTML रिले हमले** को बलपूर्वक कर सकते हैं।

### **कंप्यूटर शेयर्स में Creds खोजें**

अब जब आपके पास कुछ मौलिक क्रेडेंशियल्स हैं, तो आपको देखना चाहिए कि क्या आप **एक्टिव डायरेक्टरी के अंदर साझा किए गए किसी भी दिलचस्प फ़ाइल्स** को खोज सकते हैं। आप इसे मैन्युअल रूप से कर सकते हैं लेकिन यह एक बहुत ही उक्तिशील और दोहरावात्मक कार्य है (और अगर आप सैकड़ों दस्तावेज़ पाते हैं तो आपको जांचने की आवश्यकता होगी)।

[**इस लिंक पर जानने के लिए उपयोग करने वाले उपकरणों के बारे में पढ़ें।**](../../network-services-pentesting/pentesting-smb.md#domain-shared-folders-search)

### NTLM Creds चुराएं

यदि आप **अन्य पीसी या शेयर्स तक पहुंच सकते हैं** तो आप **फ़ाइलें रख सकते हैं** (जैसे एक SCF फ़ाइल) जो यदि किसी प्रकार से एक्सेस किया जाता है तो वह आपके खिलाफ **NTML प्रमाणीकरण को ट्रिगर** करेगा ताकि आप इसे **चुरा** सकें और इसे क्रैक कर सकें:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### CVE-2021-1675/CVE-2021-34527 PrintNightmare

इस सुरक्षा गड़बड़ी ने किसी भी प्रमाणित उपयोगकर्ता को **डोमेन नियंत्रक को कंप्रमाइज** करने की अनुमति दी।

{% content-ref url="printnightmare.md" %}
[printnightmare.md](printnightmare.md)
{% endcontent-ref %}

## एक्टिव डायरेक्टरी पर विशेषाधिकार उन्नति साथ प्रिविलेज्ड क्रेडेंशियल/सत्र

**निम्नलिखित तकनीकों के लिए एक साधारण डोमेन उपयोगकर्ता पर्याप्त नहीं है, आपको इन हमलों को करने के लिए कुछ विशेष विशेषाधिकार/क्रेडेंशियल की आवश्यकता है।**

### हैश निकालना

आशा है कि आपने कुछ स्थानीय व्यवस्थापक खाते को **कंप्रमाइज** करने में सफलता प्राप्त की है [AsRepRoast](asreproast.md), [Password Spraying](password-spraying.md), [Kerberoast](kerberoast.md), [Responder](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md) जिसमें रिले होता है, [EvilSSDP](../../generic-methodologies-and-resources/pentesting-network/spoofing-ssdp-and-upnp-devices.md), [स्थानीय विशेषाधिकारों को उन्नत करना](../windows-local-privilege-escalation/)।\
फिर, यह समय है कि आप सभी हैशों को मेमोरी और स्थानीय रूप से डंप करें।\
[**इन हैश प्राप्त करने के विभिन्न तरीकों के बारे में इस पृष्ठ को पढ़ें।**](broken-reference/)

### हैश पास करें

**जब आपके पास किसी उपयोगकर्ता का हैश होता है**, तो आप इसका उपयोग इसकी अनुकरण करने के लिए कर सकते हैं।\
आपको किसी **उपकरण** का उपयोग करना होगा जो उस **हैश का उपयोग करके** NTLM प्रमाणीकरण करेगा, **या** आप एक नया **सत्रलॉगऑन** बना सकते हैं और उस **हैश** को **LSASS** के अंदर इंजेक्ट कर सकते हैं, ताकि जब भी कोई **NTLM प्रमाणीकरण किया जाता है**, तो उस **हैश का उपयोग किया जाएगा**। आखिरी विकल्प है कि mimikatz क्या करता है।\
[**अधिक जानकारी के लिए इस पृष्ठ को पढ़ें।**](../ntlm/#pass-the-hash)

### हैश पार करें/कुंजी पास करें

यह हमला **उपयोगकर्ता NTLM हैश का उपयोग करके कर्बेरोस टिकटें अनुरोध करने का उद्देश्य रखता है**, सामान्य Pass The Hash over NTLM प्रोटोकॉल के विकल्प के रूप में। इसलिए, यह विशेष रूप से **उपयोगी हो सकता है जहां NTLM प्रोटोकॉल अक्षम है** और केवल **कर्बेरोस को प्रमाणीकरण प्रोटोकॉल के रूप में अनुमति है**।

{% content-ref url="over-pass-the-hash-pass-the-key.md" %}
[over-pass-the-hash-pass-the-key.md](over-pass-the-hash-pass-the-key.md)
{% endcontent-ref %}

### टिकट पार करें

**पास द टिकट (PTT)** हमला विधि में, हमलावर **उपयोगकर्ता का प्रमाणीकरण टिकट चुर
```bash
# Local Auth Spray (once you found some local admin pass or hash)
## --local-auth flag indicate to only try 1 time per machine
crackmapexec smb --local-auth 10.10.10.10/23 -u administrator -H 10298e182387f9cab376ecd08491764a0 | grep +
```
{% hint style="warning" %}
कृपया ध्यान दें कि यह काफी **शोर** करने वाला है और **LAPS** इसे **कम** कर सकता है।
{% endhint %}

### MSSQL दुरुपयोग और विश्वसनीय लिंक

यदि किसी उपयोगकर्ता को **MSSQL इंस्टेंसेस तक पहुंचने** की अनुमति है, तो उसे इसका उपयोग करके **MSSQL होस्ट में कमांड चलाने**, NetNTLM **हैश चुराने** या यहाँ तक कि एक **रिले अटैक** करने की संभावना है।\
इसके अलावा, यदि एक MSSQL इंस्टेंस एक विश्वसनीय है (डेटाबेस लिंक) एक अलग MSSQL इंस्टेंस द्वारा। यदि उपयोगकर्ता को विश्वसनीय डेटाबेस पर अधिकार है, तो उसे दूसरे इंस्टेंस में भी क्वेरी चलाने की अनुमति होगी। ये विश्वास किए जा सकते हैं और किसी बिंदु पर उपयोगकर्ता को एक गलत कॉन्फ़िगर किए गए डेटाबेस का पता लगाने की संभावना है।\
**डेटाब
```
Get-DomainTrust

SourceName      : sub.domain.local    --> current domain
TargetName      : domain.local        --> foreign domain
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST       --> WITHIN_FOREST: Both in the same forest
TrustDirection  : Bidirectional       --> Trust direction (2ways in this case)
WhenCreated     : 2/19/2021 1:28:00 PM
WhenChanged     : 2/19/2021 1:28:00 PM
```
{% hint style="warning" %}
यहाँ **2 विश्वसनीय कुंजी** हैं, एक _Child --> Parent_ के लिए और एक दूसरा _Parent_ --> _Child_ के लिए।\
आप वर्तमान डोमेन द्वारा उपयोग किया जा रहा एक कुंजी के साथ उन्हें देख सकते हैं:
```bash
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\mcorp$"'
```
{% endhint %}

#### SID-History Injection

उपयोग करके विश्वास का दुरुपयोग करते हुए बच्चा/माता डोमेन को उच्च व्यवस्थापक के रूप में उन्नत करें SID-History इंजेक्शन:

{% content-ref url="sid-history-injection.md" %}
[sid-history-injection.md](sid-history-injection.md)
{% endcontent-ref %}

#### Exploit writeable Configuration NC

कैसे Configuration Naming Context (NC) का शोषण किया जा सकता है, इसे समझना महत्वपूर्ण है। Configuration NC एक केंद्रीय भंडार के रूप में काम करता है जो एक एक्टिव डायरेक्टरी (AD) वातानुकूलन में एक वन्य वन्य में कॉन्फ़िगरेशन डेटा के लिए सेवा प्रदान करता है। यह डेटा वन्य में हर डोमेन कंट्रोलर (DC) को पुनर्लेखित किया जाता है, जिसमें लिखने योग्य DCs एक लिखने योग्य प्रतिलिपि को बनाए रखते हैं। इसे शोषित करने के लिए, किसी DC पर **SYSTEM विशेषाधिकार** होना चाहिए, विशेषकर एक बच्चा DC।

**रूट DC साइट को GPO से लिंक करें**

Configuration NC के साइट कंटेनर में एडी वन्य में शामिल सभी डोमेन-ज्वाइंड कंप्यूटर्स की साइट्स के बारे में जानकारी शामिल है। किसी भी DC पर SYSTEM विशेषाधिकार के साथ काम करते हुए, हमलावर रूट DC साइट्स को GPO से लिंक कर सकते हैं। यह कार्रवाई रूट डोमेन को उस साइट पर लागू नीतियों को संशोधित करके क्षमता से क्षमता देने के लिए संभावना है।

विस्तृत जानकारी के लिए, [Bypassing SID Filtering](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-4-bypass-sid-filtering-research) पर अनुसंधान कर सकते हैं।

**वन्य में किसी भी gMSA को क्षति पहुंचाएं**

एक हमला विकेंद्रित gMSAs को लक्षित करने में शामिल है जो डोमेन के भीतर विशेषाधिकृत gMSAs को लक्षित करता है। KDS रूट कुंजी, gMSAs के पासवर्ड गणना के लिए आवश्यक है, Configuration NC में संग्रहीत है। किसी भी DC पर SYSTEM विशेषाधिकार होने पर, KDS रूट कुंजी तक पहुंचना और वन्य में किसी भी gMSA के पासवर्ड की गणना संभव है।

विस्तृत विश्लेषण [Golden gMSA Trust Attacks](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-5-golden-gmsa-trust-attack-from-child-to-parent) पर पाया जा सकता है।

**स्कीमा परिवर्तन हमला**

इस विधि की धैर्य की आवश्यकता है, नए विशेषाधिकृत AD वस्तुओं के निर्माण का इंतजार करना। SYSTEM विशेषाधिकार होने पर, हमलावर एडी स्कीमा को संशोधित कर सकता है ताकि किसी भी उपयोगकर्ता को सभी वर्गों पर पूर्ण नियंत्रण प्रदान किया जा सके। यह नए बनाए गए एडी वस्तुओं पर अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच कर सकता है।

अधिक जानकारी [Schema Change Trust Attacks](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-6-schema-change-trust-attack-from-child-to-parent) पर उपलब्ध है।

**DA से EA के साथ ADCS ESC5 के साथ**

ADCS ESC5 वंरबिलिटी लक्ष्य को पब्लिक की इंफ्रास्ट्रक्चर (PKI) वस्तुओं पर नियंत्रण प्रदान करने के लिए है ताकि एक प्रमाणपत्र टेम्पलेट बनाया जा सके जो वन्य के भीतर किसी भी उपयोगकर्ता के रूप में प्रमाणीकरण सक्षम करता है। PKI वस्तुएं Configuration NC में स्थित होती हैं, लिखने योग्य बच्चा DC को ESC5 हमलों को क्रियान्वित करने की क्षमता प्रदान करती है।

इसके अधिक विवरण [From DA to EA with ESC5](https://posts.specterops.io/from-da-to-ea-with-esc5-f9f045aa105c) में पढ़ा जा सकता है। ADCS की कमी वाले परिदृश्यों में, हमलावर को आवश्यक घटक स्थापित करने की क्षमता होती है, जैसा कि [Escalating from Child Domain Admins to Enterprise Admins](https://www.pkisolutions.com/escalating-from-child-domains-admins-to-enterprise-admins-in-5-minutes-by-abusing-ad-cs-a-follow-up/) में चर्चा की गई है।

### बाहरी वन्य डोमेन - एक-तरफी (इनबाउंड) या द्विदिशील
```powershell
Get-DomainTrust
SourceName      : a.domain.local   --> Current domain
TargetName      : domain.external  --> Destination domain
TrustType       : WINDOWS-ACTIVE_DIRECTORY
TrustAttributes :
TrustDirection  : Inbound          --> Inboud trust
WhenCreated     : 2/19/2021 10:50:56 PM
WhenChanged     : 2/19/2021 10:50:56 PM
```
इस स्थिति में **आपके डोमेन पर विश्वस्तता है** जिसे एक बाह्य डोमेन ने आपको **अनिर्धारित अनुमतियाँ** दी हैं। आपको **अपने डोमेन के मुख्य सिद्धांतों को खोजना होगा जिनके पास बाह्य डोमेन पर कौन सी पहुंच है** और फिर इसे उत्पीड़ित करने का प्रयास करना होगा:
```powershell
Get-DomainTrust -Domain current.local

SourceName      : current.local   --> Current domain
TargetName      : external.local  --> Destination domain
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Outbound        --> Outbound trust
WhenCreated     : 2/19/2021 10:15:24 PM
WhenChanged     : 2/19/2021 10:15:24 PM
```
इस स्थिति में **आपका डोमेन** किसी **विभिन्न डोमेन** से प्रिंसिपल को कुछ **विशेषाधिकार** सौंप रहा है।

हालांकि, जब एक **डोमेन विश्वसनीय होता है** जिसे विश्वसनीय डोमेन द्वारा विश्वसनीय किया जाता है, तो विश्वसनीय डोमेन **एक प्रवेशक का उपयोग करके एक प्रयोगशाला उत्पन्न करता है** जिसे विश्वसनीय पासवर्ड के रूप में उपयोग किया जाता है। जिसका मतलब है कि यह संभव है कि विश्वसनीय डोमेन में प्रवेश करने के लिए विश्वसनीय डोमेन से एक उपयोगकर्ता तक पहुंचने के लिए विश्वसनीय डोमेन से एक उपयोगकर्ता तक पहुंचने के लिए विश्वसनीय डोमेन से एक उपयोगकर्ता तक पहुंचने के लिए एक उपयोगकर्ता को एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए एक प्रवेशक का उपयोग करने के लिए
