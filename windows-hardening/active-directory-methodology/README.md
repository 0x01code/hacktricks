# Active Directory Methodology

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर फॉलो\*\* करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मौलिक अवलोकन

**एक्टिव डायरेक्टरी** एक मौलिक प्रौद्योगिकी के रूप में काम करती है, जिससे **नेटवर्क प्रशासक** नेटवर्क में **डोमेन**, **उपयोगकर्ता**, और \*\*ऑब्ज

```bash
./kerbrute_linux_amd64 userenum -d lab.ropnop.com --dc 10.10.10.10 usernames.txt #From https://github.com/ropnop/kerbrute/releases

nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='DOMAIN'" <IP>
Nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm='<domain>',userdb=/root/Desktop/usernames.txt <IP>

msf> use auxiliary/gather/kerberos_enumusers

crackmapexec smb dominio.es  -u '' -p '' --users | awk '{print $4}' | uniq
```

* **OWA (Outlook Web Access) सर्वर**

अगर आप नेटवर्क में इन सर्वरों में से एक पाते हैं तो आप इसके खिलाफ **उपयोगकर्ता गणना कर सकते हैं**। उदाहरण के लिए, आप [**MailSniper**](https://github.com/dafthack/MailSniper) टूल का उपयोग कर सकते हैं:

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
आप [**इस github रेपो**](https://github.com/danielmiessler/SecLists/tree/master/Usernames/Names) और इस एक ([**आंकड़े के अनुसार संभावित उपयोगकर्ता नाम**](https://github.com/insidetrust/statistically-likely-usernames)) में उपयोगकर्ता नामों की सूचियाँ पा सकते हैं।

हालांकि, आपको **कंपनी में काम करने वाले लोगों के नाम** का पता होना चाहिए जो आपने इससे पहले किया होना चाहिए। नाम और उपनाम के साथ आप [**namemash.py**](https://gist.github.com/superkojiman/11076951) स्क्रिप्ट का उपयोग करके संभावित मान्य उपयोगकर्ता नाम उत्पन्न कर सकते हैं।
{% endhint %}

### एक या कई उपयोगकर्ता नामों को जानना

ठीक है, तो आपको पहले से ही एक मान्य उपयोगकर्ता नाम पता है लेकिन कोई पासवर्ड नहीं है... तो कोशिश करें:

* [**ASREPRoast**](asreproast.md): यदि किसी उपयोगकर्ता के पास विशेषता _DONT\_REQ\_PREAUTH_ नहीं है, तो आप उस उपयोगकर्ता के लिए एक AS\_REP संदेश का अनुरोध कर सकते हैं जो उस उपयोगकर्ता के पासवर्ड के एक विकल्प से एन्क्रिप्ट किए गए कुछ डेटा शामिल करेगा।
* [**पासवर्ड स्प्रेइंग**](password-spraying.md): सबसे **सामान्य पासवर्ड** को प्रयास करें प्रत्येक खोजे गए उपयोगकर्ताओं के साथ, शायद कोई उपयोगकर्ता एक बुरा पासवर्ड उपयोग कर रहा हो (ध्यान रखें पासवर्ड नीति को!)।
* ध्यान दें कि आप उपयोगकर्ताओं के मेल सर्वर में पहुंचने की कोशिश करने के लिए भी **OWA सर्वरों को स्प्रे** कर सकते हैं।

{% content-ref url="password-spraying.md" %}
[password-spraying.md](password-spraying.md)
{% endcontent-ref %}

### LLMNR/NBT-NS Poisoning

आप **नेटवर्क** के कुछ प्रोटोकॉल को **पॉइजन करके** कुछ चुनौती **हैश** प्राप्त कर सकते हैं जिन्हें क्रैक करने के लिए:

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

### NTML Relay

यदि आपने सक्रिय निर्देशिका की जांच करने में सफलता प्राप्त की है तो आपके पास **अधिक ईमेल और नेटवर्क की बेहतर समझ** होगी। आप \*\*NTML [**रिले हमले**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack) करने के लिए सक्षम हो सकते हैं ताकि आप AD एनवायरनमें पहुंच सकें।

### NTLM Creds चुराना

यदि आप **अन्य पीसी या शेयर** तक पहुंच सकते हैं **शून्य या मेहमान उपयोगकर्ता** के साथ, तो आप फ़ाइलें डाल सकते हैं (जैसे एक SCF फ़ाइल) जो यदि किसी प्रकार से एक्सेस किया जाता है तो वह आपके खिलाफ एक NTML प्रमाणीकरण को **ट्रिगर** करेगा ताकि आप उसे **चुरा** सकें और **NTLM चुनौती** को क्रैक कर सकें:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

## सक्रिय निर्देशिका की गणना सह साक्षात्कार/सत्र

इस चरण के लिए आपको **मान्य डोमेन खाते के प्रमाण पत्र या सत्र का संक्षिप्त करना चाहिए।** यदि आपके पास कुछ मान्य प्रमाण पत्र हैं या एक डोमेन उपयोगकर्ता के रूप में एक शैली है, **तो आपको याद रखना चाहिए कि पहले दिए गए विकल्प अब भी अन्य उपयोगकर्ताओं को संक्रमित करने के लिए विकल्प हैं**।

प्रमाणीकृत गणना शुरू करने से पहले आपको यह जानना चाहिए कि **कर्बेरोस डबल हॉप समस्या क्या है।**

{% content-ref url="kerberos-double-hop-problem.md" %}
[kerberos-double-hop-problem.md](kerberos-double-hop-problem.md)
{% endcontent-ref %}

### गणना

एक खाते को संक्रमित करना एक **बड़ा कदम है पूरे डोमेन को संक्रमित करने के लिए**, क्योंकि आप **सक्रिय निर्देशिका गणना शुरू कर सकेंगे:**

[**ASREPRoast**](asreproast.md) के संदर्भ में अब आप हर संभावित वंशावली उपयोगकर्ता पा सकते हैं, और [**पासवर्ड स्प्रेइंग**](password-spraying.md) के संदर्भ में आप **सभी उपयोगकर्ता नामों की सूची** प्राप्त कर सकते हैं और संक्रमित खाते का पासवर्ड, खाली पासवर्ड और नए उम्मीदवार पासवर्ड का प्रयास कर सकते हैं।

* आप [**बुनियादी जानकारी के लिए CMD का उपयोग कर सकते हैं**](../basic-cmd-for-pentesters.md#domain-info)
* आप [**पावरशेल का उपयोग कर सकते हैं जांच के लिए**](../basic-powershell-for-pentesters/) जो गुप्तचर होगा
* आप [**पावरव्यू का उपयोग कर सकते हैं**](../basic-powershell-for-pentesters/powerview.md) अधिक विस्तृत जानकारी निकालने के लिए
* एक और अद्भुत उपकरण सक्रिय निर्देशिका में जांच के लिए है [**BloodHound**](bloodhound.md)। यह **बहुत गुप्तचर नहीं है** (आपके उपयोग किए गए संग्रहण विधियों पर निर्भर करता है), लेकिन **यदि आपको परवाह नहीं है** तो आपको इसे एक बार जरूर देना चाहिए। यहाँ उपयोगकर्ता कहां RDP कर सकते हैं, अन्य समूहों के लिए पथ खोजें, आदि।
* **अन्य स्वचालित एडी गणना उपकरण हैं:** [**AD Explorer**](bloodhound.md#ad-explorer)**,** [**ADRecon**](bloodhound.md#adrecon)**,** [**Group3r**](bloodhound.md#group3r)**,** [**PingCastle**](bloodhound.md#pingcastle)**.**
* [**AD के DNS रिकॉर्ड**](ad-dns-records.md) क्योंकि वे दिलचस्प जानकारी शामिल कर सकते हैं।
* एक **GUI उपकरण** जिसका उपयोग आप निर्देशिका की गणना के लिए कर सकते हैं **AdExplorer.exe** से **SysInternal** सुइट से।
* आप फ़ील्ड _userPassword_ और _unixUserPassword_ में प्रमाण पत्र खोजने के लिए **ldapsearch** का उपयोग कर सकते हैं, या यहाँ तक कि _Description_ के लिए। cf. [PayloadsAllTheThings पर AD उपयोगकर्ता टिप्पणी में पासवर्ड](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#password-in-ad-user-comment) अन्य विधियों के लिए।
* यदि आप **Linux** का उपयोग कर रहे हैं, तो आप [**pywerview**](https://github.com/the-useless-one/pywerview) का उपयोग करके डोमेन की गणना कर सकते हैं।
* आप ऑटोमेटेड उपकरणों का प्रयास कर सकते हैं जैसे:
* [**tomcarver16/ADSearch**](https://github.com/tomcarver16/ADSearch)
* [**61106960/adPEAS**](https://github.com/61106960/adPEAS)
* **सभी डोमेन उपयोगकर्ताओं को निकालना**

Windows से सभी डोमेन उपयोगकर्ता नाम आसान है (`net user /domain`, `Get-DomainUser` या `wmic useraccount get name,sid`)। Linux में, आप इस्तेमाल कर सकते हैं: `GetADUsers.py -all -dc-ip 10.10.10.110 domain.com/username` या `enum4linux -a -u "user" -p "password" <DC IP>`

> यदि यह गणना खंड छोटा लग रहा है तो यह सभी का सबसे महत्वपूर्ण हिस्सा है। लिंक

```bash
## List all tickets (if not admin, only current user tickets)
.\Rubeus.exe triage
## Dump the interesting one by luid
.\Rubeus.exe dump /service:krbtgt /luid:<luid> /nowrap
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))
```

### NTML Relay

यदि आपने सक्रिय निर्देशिका की जांच करने में सफलता प्राप्त की है, तो आपके पास **अधिक ईमेल और नेटवर्क की बेहतर समझ** होगी। आप **NTML रिले हमले** को बलपूर्वक कर सकते हैं।

### **कंप्यूटर शेयर में Creds खोजें**

अब जब आपके पास कुछ मौलिक क्रेडेंशियल्स हैं, तो आपको यह देखना चाहिए कि क्या आप **एक्टिव डायरेक्टरी के अंदर साझा किए गए किसी भी दिलचस्प फ़ाइल्स** को खोज सकते हैं। आप इसे मैन्युअल रूप से कर सकते हैं, लेकिन यह एक बहुत ही उक्तिशील और दोहरावपूर्ण कार्य है (और अगर आप सैकड़ों दस्तावेज़ पाते हैं तो आपको जांचने की आवश्यकता है)।

[**इस लिंक पर जाएं और उपयोग करने वाले उपकरणों के बारे में जानें।**](../../network-services-pentesting/pentesting-smb/#domain-shared-folders-search)

### NTLM Creds चुराएं

यदि आप **अन्य पीसी या शेयर तक पहुंच सकते हैं**, तो आप **फ़ाइलें रख सकते हैं** (जैसे एक SCF फ़ाइल) जो यदि किसी प्रकार से एक्सेस किया जाएगा तो वह आपके खिलाफ **NTML प्रमाणीकरण को ट्रिगर** करेगा ताकि आप इसे **चुरा** सकें और इसे क्रैक कर सकें:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### CVE-2021-1675/CVE-2021-34527 PrintNightmare

इस सुरक्षा गड़बड़ी ने किसी भी प्रमाणित उपयोगकर्ता को **डोमेन नियंत्रक को कंप्रमाइज** करने की अनुमति दी।

{% content-ref url="printnightmare.md" %}
[printnightmare.md](printnightmare.md)
{% endcontent-ref %}

## एक्टिव डायरेक्टरी पर विशेषाधिकार उन्नति से उन्नति

**निम्नलिखित तकनीकों के लिए एक साधारण डोमेन उपयोगकर्ता पर्याप्त नहीं है, आपको इन हमलों को करने के लिए कुछ विशेष विशेषाधिकार/प्रमाणीकरण की आवश्यकता है।**

### हैश निकालना

आशा है कि आपने कुछ स्थानीय व्यवस्थापक खाते को **कंप्रमाइज** करने में सफलता प्राप्त की है [AsRepRoast](asreproast.md), [Password Spraying](password-spraying.md), [Kerberoast](kerberoast.md), [Responder](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md) जिसमें रिले होता है, [EvilSSDP](../../generic-methodologies-and-resources/pentesting-network/spoofing-ssdp-and-upnp-devices.md), [स्थानीय विशेषाधिकारों को उन्नत करना](../windows-local-privilege-escalation/)।\
फिर, यह समय है कि आप सभी हैशों को मेमोरी और स्थानीय रूप से डंप करें।\
[**इन हैश प्राप्त करने के विभिन्न तरीकों के बारे में इस पृष्ठ को पढ़ें।**](https://github.com/carlospolop/hacktricks/blob/in/windows-hardening/active-directory-methodology/broken-reference/README.md)

### हैश पास करें

**जब आपके पास किसी उपयोगकर्ता का हैश होता है**, तो आप इसका उपयोग उसकी **अनुकरण** करने के लिए कर सकते हैं।\
आपको किसी **उपकरण** का उपयोग करना होगा जो उस **हैश** का उपयोग करके **NTLM प्रमाणीकरण करेगा**, **या** आप एक नया **सत्रलॉगऑन** बना सकते हैं और उस **हैश** को **LSASS** के अंदर इंजेक्ट कर सकते हैं, ताकि जब कोई **NTLM प्रमाणीकरण किया जाता है**, तो उस **हैश** का उपयोग किया जाएगा। आखिरी विकल्प है जिसे mimikatz करता है।\
[**अधिक जानकारी के लिए इस पृष्ठ को पढ़ें।**](../ntlm/#pass-the-hash)

### हैश पास करने के ऊपर/कुंजी पास करना

यह हमला **उपयोगकर्ता NTLM हैश का उपयोग करके कर्बेरोस टिकटें अनुरोध करने का उद्देश्य रखता है**, सामान्य Pass The Hash के ऊपर NTLM प्रोटोकॉल में। इसलिए, यह विशेष रूप से **उपयोगी हो सकता है जहां NTLM प्रोटोकॉल निषेधित है** और केवल **कर्बेरोस को प्रमाणीकरण प्रोटोकॉल के रूप में अनुमति है**।

{% content-ref url="over-pass-the-hash-pass-the-key.md" %}
[over-pass-the-hash-pass-the-key.md](over-pass-the-hash-pass-the-key.md)
{% endcontent-ref %}

### टिकट पास करें

**पास द टिकट (PTT)** हमला विधि में, हमलावर **उपयोगकर्ता का प्रमाणीकरण टिकट चुराते हैं** उनके पासवर्ड या हैश मानों के बजाय। यह चोरी गई टिकट फिर उपयोगकर्ता का अनुकरण करने के लिए किया जाता है, जिससे नेटवर्क के भीतर संसाधनों और सेवाओं तक अनधिकृत पहुंच प्राप्त होती है।

{% content-ref url="pass-the-ticket.md" %}
[pass-the-ticket.md](pass-the-ticket.md)
{% endcontent-ref %}

### प्रमाणीकरण पुनः उपयोग

यदि आपके पास **हैश** या **स्थानीय प्रशासक** का **पासवर्ड** है, तो आपको इसका प्रयास करना चाहिए कि आप इसका उपयोग करके अन्य **पीसी** में स्थानीय रूप से **लॉगिन** करें।

```bash
# Local Auth Spray (once you found some local admin pass or hash)
## --local-auth flag indicate to only try 1 time per machine
crackmapexec smb --local-auth 10.10.10.10/23 -u administrator -H 10298e182387f9cab376ecd08491764a0 | grep +
```

{% hint style="warning" %}
कृपया ध्यान दें कि यह काफी **शोर** करने वाला है और **LAPS** इसे **कम** कर सकता है।
{% endhint %}

### MSSQL दुरुपयोग और विश्वसनीय लिंक

यदि किसी उपयोगकर्ता को **MSSQL इंस्टेंसेस तक पहुंचने** की अनुमति है, तो उसे इसका उपयोग करके MSSQL होस्ट में (यदि SA के रूप में चल रहा है) **कमांड्स को निष्पादित** करने, NetNTLM **हैश** चुरा लेने या यहां तक कि एक **रिले** **हमला** भी कर सकता है।\
इसके अलावा, यदि एक MSSQL इंस्टेंस एक विश्वसनीय है (डेटाबेस लिंक) एक विभिन्न MSSQL इंस्टेंस द्वारा। यदि उपयोगकर्ता को विश्वसनीय डेटाबेस पर अधिकार है, तो उसे दूसरे इंस्टेंस में भी क्वेरी निष्पादित करने की अनुमति होगी। ये विश्वास करने की अनुमति देते हैं और किसी बिगड़ी हुई डेटाबेस को ढूंढने की संभावना है जहां उसे कमांड्स निष्पादित करने की अनुमति हो सकती है।\
\*\*डेटाब

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
आप वर्तमान डोमेन द्वारा उपयोग की गई एक कुंजी को निम्नलिखित के साथ देख सकते हैं:

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

कैसे Configuration Naming Context (NC) का शोषण किया जा सकता है, इसे समझना महत्वपूर्ण है। Configuration NC एक केंद्रीय भंडार के रूप में काम करता है जो एक एक्टिव डायरेक्टरी (AD) वातानुकूलन में एक वन्य वन्य में कॉन्फ़िगरेशन डेटा के लिए सेवा प्रदान करता है। यह डेटा वन्य में हर डोमेन कंट्रोलर (DC) को पुनर्लेखित किया जाता है, जिसमें लिखने योग्य DCs एक लिखने योग्य प्रतिलिपि को बनाए रखते हैं। इसे शोषित करने के लिए, किसी DC पर **SYSTEM विशेषाधिकार** होना चाहिए, बेहतर है एक बच्चा DC।

**रूट DC साइट को GPO से लिंक करें**

Configuration NC के साइट कंटेनर में एडी फॉरेस्ट के सभी डोमेन-ज्वाइंड कंप्यूटर्स के साइट्स के बारे में जानकारी शामिल है। किसी भी DC पर SYSTEM विशेषाधिकारों के साथ काम करते हुए, हमलावर रूट DC साइट्स को GPO से लिंक कर सकते हैं। यह कार्रवाई रूट डोमेन को खतरे में डाल सकती है जिसे इन साइट्स पर लागू नीतियों को हाथ में लेकर।

विस्तृत जानकारी के लिए, [Bypassing SID Filtering](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-4-bypass-sid-filtering-research) पर अनुसंधान कर सकते हैं।

**वन्य में किसी भी gMSA को कमबख्त करें**

एक हमला वेक्टर डोमेन के भीतर विशेषाधिकृत gMSAs को लक्षित करने में शामिल है। KDS रूट कुंजी, gMSAs के पासवर्ड की गणना के लिए आवश्यक है, Configuration NC के अंदर संग्रहीत है। किसी भी DC पर SYSTEM विशेषाधिकार होने पर, KDS रूट कुंजी तक पहुंचना और वन्य में किसी भी gMSA के पासवर्ड की गणना संभव है।

विस्तृत विश्लेषण [Golden gMSA Trust Attacks](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-5-golden-gmsa-trust-attack-from-child-to-parent) पर पाया जा सकता है।

**स्कीमा परिवर्तन हमला**

इस विधि के लिए धैर्य की आवश्यकता है, नए विशेषाधिकृत AD वस्तुओं के निर्माण का इंतजार करना। SYSTEM विशेषाधिकारों के साथ, हमलावर एडी स्कीमा को संशोधित कर सकता है ताकि किसी भी उपयोगकर्ता को सभी वर्गों पर पूर्ण नियंत्रण प्रदान किया जा सके। यह नए बनाए गए एडी वस्तुओं पर अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अनधिकृत पहुंच और नियंत्रण करने की अ

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

इस स्थिति में **आपके डोमेन पर विश्वस्तता है** जिसे एक बाह्य डोमेन ने दी है, जिससे आपको **अनिर्धारित अनुमतियाँ** हैं। आपको ढूंढ़ना होगा **कि आपके डोमेन के कौन-कौन से मुख्य सिद्धांत हैं जो बाह्य डोमेन पर किस प्रकार की पहुंच है** और फिर इसे उत्पीड़ित करने का प्रयास करना होगा:

{% content-ref url="external-forest-domain-oneway-inbound.md" %}
[external-forest-domain-oneway-inbound.md](external-forest-domain-oneway-inbound.md)
{% endcontent-ref %}

### बाह्य वन वन डोमेन - एक-तरफा (आउटबाउंड)

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

इस स्थिति में **आपका डोमेन** किसी **विशेषाधिकार** को **एक विभिन्न डोमेन** से एक प्रिंसिपल को **विश्वसनीय बनाता है**।

हालांकि, जब एक **डोमेन विश्वसनीय होता है** विश्वसनीय डोमेन एक उपयोगकर्ता बनाता है जिसका **निर्धारित नाम** होता है जो **विश्वसनीय पासवर्ड** के रूप में उपयोग किया जाता है। जिसका मतलब है कि यह संभव है कि **विश्वसनीय डोमेन में प्रवेश प्राप्त करने के लिए विश्वसनीय डोमेन से एक उपयोगकर्ता तक पहुंचना**:

{% content-ref url="external-forest-domain-one-way-outbound.md" %}
[external-forest-domain-one-way-outbound.md](external-forest-domain-one-way-outbound.md)
{% endcontent-ref %}

विश्वसनीय डोमेन को कंप्रमाइज करने का एक और तरीका है [**SQL trusted link**](abusing-ad-mssql.md#mssql-trusted-links) खोजना जो डोमेन विश्वास के विपरीत दिशा में बनाया गया है (जो बहुत सामान्य नहीं है)।

विश्वसनीय डोमेन को कंप्रमाइज करने का एक और तरीका है जब एक **विश्वसनीय डोमेन से उपयोगकर्ता एक्सेस कर सकता है** एक मशीन में प्रतीक्षा करना जहां से **RDP** के माध्यम से लॉगिन किया जा सकता है। फिर, हमलावर RDP सत्र प्रक्रिया में कोड इंजेक्ट कर सकता है और **पीड़ित के मूल डोमेन तक पहुंच सकता है**। इसके अतिरिक्त, यदि **पीड़ित ने अपनी हार्ड ड्राइव माउंट की है**, तो RDP सत्र प्रक्रिया से हमलावर **हार्ड ड्राइव के स्टार्टअप फ़ोल्डर में बैकडोर्स** स्टोर कर सकता है। इस तकनीक को **RDPInception** कहा जाता है।

{% content-ref url="rdp-sessions-abuse.md" %}
[rdp-sessions-abuse.md](rdp-sessions-abuse.md)
{% endcontent-ref %}

### डोमेन विश्वास दुरुपयोग संशोधन

### **SID फ़िल्टरिंग:**

* वन फ़ॉरेस्ट ट्रस्ट के बीच SID इतिहास विश्वास पर आक्रमण का जोखिम SID फ़िल्टरिंग द्वारा कम किया जाता है, जो स्वचालित है सभी इंटर-फ़ॉरेस्ट ट्रस्ट पर। इसका आधार माइक्रोसॉफ्ट के दृष्टिकोण के अनुसार वन फ़ॉरेस्ट विश्वास को सुरक्षित मानते हुए है।
* हालांकि, एक चीज है: SID फ़िल्टरिंग अनुप्रयोगों और उपयोगकर्ता एक्सेस को विघटित कर सकता है, जिससे इसका कभी-कभी निषेध किया जाता है।

### **चयनात्मक प्रमाणीकरण:**

* इंटर-फ़ॉरेस्ट ट्रस्ट के लिए, चयनात्मक प्रमाणीकरण का उपयोग सुनिश्चित करता है कि दो वनों के उपवनों से उपयोगकर्ताओं को स्वचालित रूप से प्रमाणित नहीं किया जाता है। बजाय इसके, विश्वसनीय डोमेन या वन के भीतर डोमेन और सर्वरों तक पहुंच के लिए उपयोगकर्ताओं को स्पष्ट अनुमतियाँ आवश्यक होती हैं।
* यह महत्वपूर्ण है कि ये उपाय लिखने वाले कॉन्फ़िगरेशन नेमिंग संदर्भ (NC) या विश्वास खाते पर हमलों के खिलाफ सुरक्षित नहीं रखते।

[**ired.team में डोमेन विश्वास के बारे में अधिक जानकारी।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain)
