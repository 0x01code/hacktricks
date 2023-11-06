# एक्टिव डायरेक्टरी मेथडोलॉजी

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

## मूल अवलोकन

एक्टिव डायरेक्टरी नेटवर्क प्रशासकों को नेटवर्क के भीतर डोमेन, उपयोगकर्ता और ऑब्जेक्ट्स बनाने और प्रबंधित करने की अनुमति देता है। उदाहरण के लिए, एक एडमिन एक समूह उपयोगकर्ताओं को बना सकता है और उन्हें सर्वर पर निर्दिष्ट निर्देशिकाओं के लिए विशेष पहुंच अधिकार प्रदान कर सकता है। जब नेटवर्क बढ़ता है, एक्टिव डायरेक्टरी एक तरीका प्रदान करता है जिससे एक बड़ी संख्या के उपयोगकर्ताओं को तार्किक समूहों और उप-समूहों में व्यवस्थित किया जा सकता है, जबकि प्रत्येक स्तर पर पहुंच नियंत्रण प्रदान किया जाता है।

एक्टिव डायरेक्टरी संरचना में तीन मुख्य स्तर शामिल होते हैं: 1) डोमेन, 2) ट्रीज़, और 3) फ़ॉरेस्ट। एक ही डेटाबेस का उपयोग करने वाले कई ऑब्जेक्ट्स (उपयोगकर्ता या उपकरण) एक ही डोमेन में समूहित किए जा सकते हैं। कई डोमेनों को एक ही समूह में जोड़ा जा सकता है जिसे ट्री कहा जाता है। कई ट्रीज़ को एक संग्रह में जोड़ा जा सकता है जिसे फ़ॉरेस्ट कहा जाता है। इन सभी स्तरों में विशेष पहुंच अधिकार और संचार विशेषाधिकारों को सौंपा जा सकता है।

एक्टिव डायरेक्टरी कई विभिन्न सेवाएं प्रदान करता है, जो "एक्टिव डायरेक्टरी डोमेन सेवाएं" या AD DS के तहत आती हैं। इन सेवाओं में शामिल हैं:

1. **डोमेन सेवाएं** - केंद्रीयकृत डेटा संग्रहीत करती हैं और उपयोगकर्ताओं और डोमेनों के बीच संचार का प्रबंधन करती हैं; लॉगिन प्रमाणीकरण और खोज क्षमता शामिल होती हैं
2. **सर्टिफिकेट सेवाएं** - सुरक्षित प्रमाणपत्रों को बनाती, वितरित करती और प्रबंधित करती हैं
3. **लाइटवेट डायरेक्टरी सेवाएं** - खुले (LDAP) प्रोटोकॉल का उपयोग करके निर्देशिका-सक
* [**अभियांत्रिकी आक्रमण**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack) का उपयोग करके होस्ट तक पहुंचें
* [**खुलासा करने वाले**](../../generic-methodologies-and-resources/pentesting-network/spoofing-ssdp-and-upnp-devices.md) [**खोखली UPnP सेवाओं का उपयोग करें evil-S**](https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856)
* [**OSINT**](https://book.hacktricks.xyz/external-recon-methodology):
* आंतरिक दस्तावेज़ों, सामाजिक मीडिया, सेवाओं (मुख्य रूप से वेब) के अंदर डोमेन वातावरण और सार्वजनिक रूप से उपलब्ध उपयोगकर्ताओं से उपयोगकर्ता नाम / नाम निकालें।
* यदि आप कंपनी के कर्मचारियों के पूरे नाम पाते हैं, तो आप विभिन्न AD **उपयोगकर्ता नामनिर्धारणों** का प्रयास कर सकते हैं (**[यह पढ़ें](https://activedirectorypro.com/active-directory-user-naming-convention/)**)। सबसे सामान्य नियमांकन हैं: _नामउपनाम_, _नाम.उपनाम_, _नामउप_, _नाम.उपनाम_, _उपनामनाम_, _उपनाम.नाम_, _उपनामन_, _उपनाम.नाम_, _उपनामन_, _उपनाम.नाम_, 3 _यादृच्छिक अक्षर और 3 यादृच्छिक संख्याएं_ (abc123)।
* उपकरण:
* [w0Tx/generate-ad-username](https://github.com/w0Tx/generate-ad-username)
* [urbanadventurer/username-anarchy](https://github.com/urbanadventurer/username-anarchy)

### उपयोगकर्ता गणना

* **अनाम संब/एलडीएपी गणना:** [**पेंटेस्टिंग SMB**](../../network-services-pentesting/pentesting-smb.md) और [**पेंटेस्टिंग LDAP**](../../network-services-pentesting/pentesting-ldap.md) पृष्ठों की जांच करें।
* **Kerbrute गणना**: जब एक **अमान्य उपयोगकर्ता नाम अनुरोध** किया जाता है, सर्वर **Kerberos त्रुटि** कोड _KRB5KDC\_ERR\_C\_PRINCIPAL\_UNKNOWN_ का उपयोग करके प्रतिक्रिया करेगा, जिससे हमें पता चलेगा कि उपयोगकर्ता नाम अमान्य था। **मान्य उपयोगकर्ता नाम** एक AS-REP प्रतिक्रिया में TGT उत्पन्न करेंगे या त्रुटि _KRB5KDC\_ERR\_PREAUTH\_REQUIRED_ को प्रतिक्रिया देंगे, जिससे प्रयोक्ता को पूर्व-प्रमाणीकरण करने की आवश्यकता होती है।
```bash
./kerbrute_linux_amd64 userenum -d lab.ropnop.com --dc 10.10.10.10 usernames.txt #From https://github.com/ropnop/kerbrute/releases

nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='DOMAIN'" <IP>
Nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm='<domain>',userdb=/root/Desktop/usernames.txt <IP>

msf> use auxiliary/gather/kerberos_enumusers

crackmapexec smb dominio.es  -u '' -p '' --users | awk '{print $4}' | uniq
```
* **OWA (Outlook Web Access) सर्वर**

यदि आप नेटवर्क में इनमें से किसी भी सर्वर को खोजते हैं, तो आप इसके खिलाफ **उपयोगकर्ता गणना** भी कर सकते हैं। उदाहरण के लिए, आप [**MailSniper**](https://github.com/dafthack/MailSniper) टूल का उपयोग कर सकते हैं:
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
आप [**इस github रेपो**](https://github.com/danielmiessler/SecLists/tree/master/Usernames/Names) और इस एक ([**statistically-likely-usernames**](https://github.com/insidetrust/statistically-likely-usernames)) में उपयोगकर्ताओं की सूची ढूंढ सकते हैं।

हालांकि, आपको **कंपनी में काम करने वाले लोगों के नाम** की जानकारी होनी चाहिए, जो आपने पहले के रिकॉन स्टेप में किया होना चाहिए। नाम और उपनाम के साथ आप [**namemash.py**](https://gist.github.com/superkojiman/11076951) स्क्रिप्ट का उपयोग करके संभावित मान्य उपयोगकर्ता नाम बना सकते हैं।
{% endhint %}

### एक या कई उपयोगकर्ता नामों को जानना

ठीक है, तो आपको पहले से ही एक मान्य उपयोगकर्ता नाम होता है लेकिन कोई पासवर्ड नहीं है... तो कोशिश करें:

* [**ASREPRoast**](asreproast.md): यदि किसी उपयोगकर्ता के पास विशेषता _DONT\_REQ\_PREAUTH_ नहीं है, तो आप उस उपयोगकर्ता के लिए एक AS\_REP संदेश का अनुरोध कर सकते हैं जो उपयोगकर्ता के पासवर्ड के एक उत्पादन द्वारा एन्क्रिप्ट किए गए कुछ डेटा को समेटेगा।
* [**पासवर्ड स्प्रे**](password-spraying.md): पाए गए उपयोगकर्ताओं के साथ सबसे **सामान्य पासवर्ड** का प्रयास करें, शायद कोई उपयोगकर्ता एक खराब पासवर्ड का उपयोग कर रहा हो (पासवर्ड नीति को ध्यान में रखें!)।
* ध्यान दें कि आप उपयोगकर्ताओं के मेल सर्वर में पहुंच प्राप्त करने के लिए **OWA सर्वरों को स्प्रे** भी कर सकते हैं।

{% content-ref url="password-spraying.md" %}
[password-spraying.md](password-spraying.md)
{% endcontent-ref %}

### LLMNR/NBT-NS Poisoning

आप **नेटवर्क** के कुछ प्रोटोकॉलों को **पॉइजन करके** कुछ चुनौती **हैश** प्राप्त कर सकते हैं:

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

### NTML Relay

यदि आपने सक्रिय निर्देशिका को जांचने का प्रयास किया है, तो आपके पास **अधिक ईमेल और नेटवर्क की बेहतर समझ** होगी। आप AD env में पहुंच प्राप्त करने के लिए NTML [**रिले हमले**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack) कर सकते हैं।

### NTLM Creds चुराएं

यदि आप **अन्य PC या साझा** में **null या guest उपयोगकर्ता** के साथ पहुंच सकते हैं, तो आप **फ़ाइलें रख सकते हैं** (जैसे SCF फ़ाइल), जो यदि किसी तरह से उपयोग की जाएगी, तो वह आपके खिलाफ NTML प्रमाणीकरण को ट्रिगर करेगी, ताकि आप इसे क्रैक कर सकें:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

## प्रमाणीकरण के साथ सक्रिय निर्देशिका का जांचना

इस चरण के लिए आपको **मान्य डोमेन खाते के प्रमाणपत्र या सत्र** को संक्षेप में होना चाहिए। यदि आपके पास कुछ मान्य प्रमाणपत्र या डोमेन उपयोगकर्ता के रूप में शैल मिल गया है, **तो आपको याद रखना चाहिए कि पहले दिए गए विकल्प अन्य उपयोगकर्ताओं को संक्रमित करने के लिए विकल्प हैं**।

प्रमाणीकरण के पहले आपको जानना चाहिए कि **Kerberos डबल हॉप समस्या क्या है।**

{% content-ref url="kerberos-double-hop-problem.md" %}
[kerberos-double-hop-problem.md](kerberos-double-hop-problem.md)
{% endcontent-ref %}

### जांचना

एक खाते को संक्रमित करना **डोमेन को पूरी तरह संक्रमित करने के लिए एक बड़ा कदम है**, क्योंकि आप **सक्रिय निर्देशिका जांचना शुरू कर सकेंगे**:

[**ASREPRoast**](asreproast.md) के संबंध में अब आप हर संभावित विकल्पशील उपयोगकर्ता पा सकते हैं, और [**पासवर्ड स्प्रे**](password-spraying.md) के संबंध में आप **
### केरबेरोस्ट

केरबेरोस्ट का उद्देश्य है **डोमेन उपयोगकर्ता खातों के नाम पर चलने वाली सेवाओं के लिए टीजीएस टिकट हार्वेस्ट करना**। इन टीजीएस टिकट का हिस्सा हैं **उपयोगकर्ता पासवर्ड से निर्मित कुंजी से एन्क्रिप्ट किया जाता है**। इसके परिणामस्वरूप, उनके क्रेडेंशियल्स को **ऑफलाइन में क्रैक किया जा सकता है**।\
इसके बारे में अधिक जानकारी के लिए:

{% content-ref url="kerberoast.md" %}
[kerberoast.md](kerberoast.md)
{% endcontent-ref %}

### रिमोट कनेक्शन (RDP, SSH, FTP, Win-RM, आदि)

जब आपके पास कुछ क्रेडेंशियल्स हो जाएं, तो आप यह जांच सकते हैं कि क्या आपके पास किसी **मशीन** तक पहुंच है। इसके लिए, आप **CrackMapExec** का उपयोग कर सकते हैं और विभिन्न प्रोटोकॉलों के साथ कई सर्वरों पर कनेक्ट करने का प्रयास कर सकते हैं, अपने पोर्ट स्कैन के अनुसार।

### स्थानीय प्रिविलेज उन्नयन

यदि आपके पास कंप्रोमाइज़ किए गए क्रेडेंशियल्स या एक साधारण डोमेन उपयोगकर्ता के रूप में एक सत्र है और आपके पास इस उपयोगकर्ता के साथ किसी भी मशीन तक पहुंच है, तो आपको **स्थानीय रूप से प्रिविलेज उन्नयन करने और क्रेडेंशियल्स की लूट करने** का प्रयास करना चाहिए। इसलिए केवल स्थानीय प्रशासक अधिकारों के साथ आपको संग्रहीत होगा कि आप मेमोरी (LSASS) और स्थानीय रूप से (SAM) अन्य उपयोगकर्ताओं के हैश डंप कर सकते हैं।

इस पुस्तक में [**Windows में स्थानीय प्रिविलेज उन्नयन**](../windows-local-privilege-escalation/) के बारे में एक पूर्ण पृष्ठ है और एक [**चेकलिस्ट**](../checklist-windows-privilege-escalation.md) भी है। इसके अलावा, [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) का उपयोग करना न भूलें।

### मौजूदा सत्र टिकट

यह बहुत **असंभावित है** कि आपको मौजूदा उपयोगकर्ता में कोई **टिकट** मिलेगा जो आपको अप्रत्याशित संसाधनों तक पहुंच देगा, लेकिन आप जांच सकते हैं:
```bash
## List all tickets (if not admin, only current user tickets)
.\Rubeus.exe triage
## Dump the interesting one by luid
.\Rubeus.exe dump /service:krbtgt /luid:<luid> /nowrap
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))
```
### NTML रिले

यदि आपने सक्रिय निर्देशिका की जांच करने में सफलता प्राप्त की है, तो आपके पास **अधिक ईमेल और नेटवर्क की बेहतर समझ** होगी। आप NTML [**रिले हमलों**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack) को बलवान्तरित करने की कोशिश कर सकते हैं।

### कंप्यूटर शेयर में Creds खोजें

अब जब आपके पास कुछ मूलभूत क्रेडेंशियल्स हैं, तो आपको देखना चाहिए कि क्या आप **AD के अंदर साझा की जा रही किसी दिलचस्प फ़ाइल को खोज सकते हैं**। आप इसे मैन्युअल रूप से कर सकते हैं, लेकिन यह एक बहुत ही उबाऊ और दोहरावार कार्य है (और अगर आप सैंड्रेडों को चेक करने के लिए सैंड्रेडों को ढूंढते हैं तो और भी ज्यादा)।

[**इस लिंक पर जाकर आप उपयोग कर सकते हैं उपकरणों के बारे में जानने के लिए।**](../../network-services-pentesting/pentesting-smb.md#domain-shared-folders-search)

### NTLM Creds चुराएं

यदि आप **अन्य PC या शेयर तक पहुंच सकते हैं**, तो आप **फ़ाइलें रख सकते हैं** (जैसे SCF फ़ाइल), जो यदि किसी तरह से एक्सेस होती है, तो वह आपके खिलाफ **NTML प्रमाणीकरण को ट्रिगर करेगी**, ताकि आप इसे **चुरा सकें** और इसे क्रैक कर सकें:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### CVE-2021-1675/CVE-2021-34527 PrintNightmare

यह सुरक्षा दुर्बलता किसी भी प्रमाणित उपयोगकर्ता को डोमेन नियंत्रक को कंप्रमाइज़ करने की अनुमति देती थी।

{% content-ref url="printnightmare.md" %}
[printnightmare.md](printnightmare.md)
{% endcontent-ref %}

## उच्चाधिकार सक्रिय निर्देशिका परियोजना में उच्चाधिकार सक्रिय निर्देशिका परियोजना में

**निम्नलिखित तकनीकों के लिए एक साधारण डोमेन उपयोगकर्ता पर्याप्त नहीं है, आपको कुछ विशेष विशेषाधिकार / क्रेडेंशियल चाहिए होंगे जिसका उपयोग इन हमलों को करने के लिए किया जा सकता है।**

### हैश निष्कर्षण

आशा है कि आपने [AsRepRoast](asreproast.md), [Password Spraying](password-spraying.md), [Kerberoast](kerberoast.md), [Responder](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md) जैसे उपकरणों का उपयोग करके कुछ स्थानीय व्यवस्थापक खाता कंप्रमाइज़ कर लिया है, [EvilSSDP](../../generic-methodologies-and-resources/pentesting-network/spoofing-ssdp-and-upnp-devices.md), [इस्केलेटिंग प्रिविलेजेज़ लोकली](../windows-local-privilege-escalation/) करने के लिए।\
फिर, अब आपको सभी हैश को मेमोरी और स्थानीयता में डंप करने का समय है।\
[**इस पृष्ठ को पढ़ें जहां आपको हैश प्राप्त करने के विभिन्न तरीकों के बारे में जानकारी मिलेगी।**](broken-reference)

### हैश पार करें

**जब आपके पास किसी उपयोगकर्ता का हैश होता है**, तो आप इसका उपयोग उसकी अनुकरण करने के लिए कर सकते हैं।\
आपको कुछ **उपकरण** का उपयोग करना होगा जो इस **हैश** का उपयोग करके **NTLM प्रमाणीकरण का कार्य** करेगा, **या** आप एक नया **सत्रलॉगन** बना सकते हैं और उस **हैश** को **LSASS** के अंदर **इंजेक्ट** कर सकते हैं, ताकि जब कोई **NTLM प्रमाणीकरण कार्य** किया जाता है, तो उस **हैश** का उपयोग होगा। आखिरी विकल्प है वह कार्य जो mimikatz करता है।\
[**अधिक जानकारी के लिए इस पृष्ठ को पढ़ें।**](../ntlm/#pass-the-hash)

### हैश पार करें / कुंजी पार करें

यह हमला **उपयोगकर्ता NTLM हैश का उपयोग करके Kerberos टिकटें अनुरोधित करने** का उद्देश्य रखता है, सामान्य Pass The Hash over NTLM प्रोटोकॉल के विकल्प के रूप में। इसलिए, यह विशेष रूप से **उपयोगी हो सकता है जहां NTLM प्रोटोकॉल अक्षम है** और केवल **Kerberos को ही अनुमति है** जैसे प्रमाणीकरण प्रोटोकॉल।

{% content-ref url="over-pass-the-hash-pass-the-key.md" %}
[over-pass-the-hash-pass-the-key.md](over-pass-the-hash-pass-the-key.md)
{% endcontent-ref %}

### टिकट पार करें

यह हमला Pass the Key के समान है, लेकिन यहां हैश का उपयोग करके टिकट का चोरी किया जाता है और इसके मालिक के रूप में प्रमाणित करने के लिए उपयोग किया जाता है।

{% content-ref url="pass-the-ticket.md" %}
[pass-the-ticket.md](pass-the-ticket.md)
{% endcontent-ref %}

### क्रेडेंशियल पुनःउपयोग

यदि आपके पास किसी स्थानीय प्रशासक का **हैश** या **पासवर्ड** है, तो आपको उसके साथ अन्य **PCs** में स्थानीय रूप से **लॉगिन करने की कोशिश** करनी चाहिए।
```bash
# Local Auth Spray (once you found some local admin pass or hash)
## --local-auth flag indicate to only try 1 time per machine
crackmapexec smb --local-auth 10.10.10.10/23 -u administrator -H 10298e182387f9cab376ecd08491764a0 | grep +
```
{% hint style="warning" %}
ध्यान दें कि यह काफी **शोरगुल** है और **LAPS** इसे **कम कर सकता है**।
{% endhint %}

### MSSQL दुरुपयोग और विश्वसनीय लिंक

यदि किसी उपयोगकर्ता को **MSSQL इंस्टेंसेस तक पहुंच की अनुमति** होती है, तो वह इसे **कमांड निष्पादित** करने के लिए उपयोग कर सकता है (यदि SA के रूप में चल रहा है), NetNTLM **हैश चुरा सकता है** या यहां तक कि एक **रिले हमला** भी कर सकता है।\
इसके अलावा, यदि एक MSSQL इंस्टेंस एक अलग MSSQL इंस्टेंस द्वारा विश्वसनीय (डेटाबेस लिंक) माना जाता है। यदि उपयोगकर्ता को विश्वसनीय डेटाबेस पर अधिकार होते हैं, तो वह दूसरे इंस्टेंस में भी क्वेरी निष्पादित करने के लिए विश्वसनीय संबंध का उपयोग कर सकता है। इन विश्वसनीयताओं को चेन किया जा सकता है और किसी बिंदु पर उपयोगकर्ता को ऐसा डेटाबेस मिल सकता है जहां उसे कमांड निष्पादित कर सकता है।\
**डेटाबेसों के बीच के लिंक वन वन वन विश्वसनीयताओं के बारे में काम करते हैं।**

{% content-ref url="abusing-ad-mssql.md" %}
[abusing-ad-mssql.md](abusing-ad-mssql.md)
{% endcontent-ref %}

### असीमित डिलीगेशन

यदि आपको किसी कंप्यूटर ऑब्जेक्ट में [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx) विशेषता मिलती है और आपके पास कंप्यूटर में डोमेन अधिकार हैं, तो आप कंप्यूटर पर लॉगिन करने वाले हर उपयोगकर्ता के TGT को मेमोरी से डंप कर सकेंगे।\
इसलिए, यदि **डोमेन व्यवस्थापक कंप्यूटर पर लॉगिन करता है**, तो आप उसके TGT को डंप कर सकेंगे और [पास द टिकट](pass-the-ticket.md) का उपयोग करके उसकी अनुकरण कर सकेंगे।\
सीमित डिलीगेशन के कारण आप एक प्रिंट सर्वर को भी **स्वचालित रूप से संक्रमित** कर सकते हैं (आशा है कि यह एक DC होगा)।

{% content-ref url="unconstrained-delegation.md" %}
[unconstrained-delegation.md](unconstrained-delegation.md)
{% endcontent-ref %}

### सीमित डिलीगेशन

यदि किसी उपयोगकर्ता या कंप्यूटर को "सीमित डिलीगेशन" की अनुमति है, तो वह कंप्यूटर में कुछ सेवाओं तक पहुंच के लिए **किसी भी उपयोगकर्ता की अनुकरण कर सकेगा**।\
फिर, यदि आप इस उपयोगकर्ता/कंप्यूटर के हैश को **संक्रमित** कर लेते हैं, तो आप किसी भी उपयोगकर्ता (यहां तक कि डोमेन व्यवस्थापक) की अनुकरण करने के लिए सक्षम होंगे।

{% content-ref url="constrained-delegation.md" %}
[constrained-delegation.md](constrained-delegation.md)
{% endcontent-ref %}

### संसाधन-आधारित सीमित डिलीगेशन

यदि आपको उस कंप्यूटर के AD ऑब्जेक्ट पर WRITE अनुमति है, तो आप उस दूरस्थ कंप्यूटर पर **उच्च अधिकारों वाले निर्देशिका में कोड निष्पादित करने के साथ संबंधित अनुमतियों के साथ कोड निष्पादित कर सकते हैं**।

{% content-ref url="resource-based-constrained-delegation.md" %}
[resource-based-constrained-delegation.md](resource-based-constrained-delegation.md)
{% endcontent-ref %}

### ACL दुरुपयोग

संक्रमित उपयोगकर्ता को कुछ **दोमेन ऑब्जेक्ट्स पर दिलचस्प अधिकार** हो सकते हैं जो आपको बाद में **साइडवेजिकली ले जाने** या **अधिकारों को उन्नत करने** की अनुमति देते हैं।

{% content-ref url="acl-persistence-abuse/" %}
[acl-persistence-abuse](acl-persistence-abuse/)
{% endcontent-ref %}

### प्रिंटर स्पूलर सेवा दुरुपयोग

यदि आप डोमेन में किसी **स्पूल सेवा को सुन रहा** है, तो आप इसे **दुरुपयोग** करके **नए क्रेडेंशियल्स प्राप्त** करने और **अधिकारों को उन्नत करने** के लिए सक्षम हो सकते हैं।\
[**यहां स्पूलर सेवाओं का दुरुपयोग कैसे करें के बारे में अधिक जानकारी।**](printers-spooler-service-abuse.md)

### तृतीय पक्ष सत्र दुरुपयोग

यदि **अन्य उपयोगकर्ता** **संक्रमित** मशीन का **उपयोग** करते हैं, तो आप मेमोरी से **क्रेडें
### गोल्डन टिकट

एक मान्य **TGT के रूप में किसी भी उपयोगकर्ता** को बनाया जा सकता है **krbtgt AD खाते** के NTLM हैश का उपयोग करके। TGS की तुलना में TGT का जाल बुनने का लाभ यह है कि यह **किसी भी सेवा** (या मशीन) तक पहुंचने में सक्षम होता है और उसे आपत्तिजनक उपयोगकर्ता के रूप में उपयोग कर सकता है।

{% content-ref url="golden-ticket.md" %}
[golden-ticket.md](golden-ticket.md)
{% endcontent-ref %}

### डायमंड टिकट

ये गोल्डन टिकट के सामान होते हैं, जिन्हें एक ऐसे तरीके से बनाया जाता है जो **सामान्य गोल्डन टिकट के पता लगाने के तंत्रों को छोड़ देता है।**

{% content-ref url="diamond-ticket.md" %}
[diamond-ticket.md](diamond-ticket.md)
{% endcontent-ref %}

### **प्रमाणपत्र खाता स्थिरता**

**किसी खाते के प्रमाणपत्र होना या उन्हें अनुरोध करने की क्षमता** उपयोगकर्ता के खाते में स्थिर रहने का एक बहुत अच्छा तरीका है (यदि उसने पासवर्ड बदल दिया हो):

{% content-ref url="ad-certificates/account-persistence.md" %}
[account-persistence.md](ad-certificates/account-persistence.md)
{% endcontent-ref %}

### **प्रमाणपत्र डोमेन स्थिरता**

**प्रमाणपत्रों का उपयोग करके उच्च अधिकारों के साथ डोमेन में स्थिर रहना भी संभव है:**

{% content-ref url="ad-certificates/domain-persistence.md" %}
[domain-persistence.md](ad-certificates/domain-persistence.md)
{% endcontent-ref %}

### AdminSDHolder समूह

**AdminSDHolder** ऑब्जेक्ट की **पहुंच नियंत्रण सूची (ACL)** को एक टेम्पलेट के रूप में उपयोग किया जाता है ताकि इसे **सक्रिय डायरेक्टरी** में सभी "संरक्षित समूहों" और उनके सदस्यों में **अनुमतियों** की **प्रतिलिपि** कर सकें। संरक्षित समूहों में विशेषाधिकारी समूह शामिल होते हैं जैसे डोमेन व्यवस्थापक, प्रशासक, एंटरप्राइज व्यवस्थापक और स्कीमा व्यवस्थापक, बैकअप ऑपरेटर्स और krbtgt।
इस समूह की डिफ़ॉल्ट ACL सभी "संरक्षित समूहों" में कॉपी की जाती है। इसका उद्देश्य यह है कि इन महत्वपूर्ण समूहों में इच्छित या अकस्मात बदलावों से बचा जा सके। हालांकि, यदि कोई हमलावर उदाहरण के लिए समूह **AdminSDHolder** की ACL में परिवर्तन करता है, नियमित उपयोगकर्ता को पूरी अनुमति देता है, तो इस उपयोगकर्ता को संरक्षित समूह के अंदर के सभी समूहों पर पूरी अनुमति होगी (एक घंटे में)।
और यदि कोई इस उपयोगकर्ता को डोमेन व्यवस्थापकों से हटाने की कोशिश करता है (उदाहरण के लिए) एक घंटे या उससे कम समय में, तो उपयोगकर्ता समूह में वापस आ जाएगा।
[**यहां AdminDSHolder समूह के बारे में अधिक जानकारी।**](privileged-groups-and-token-privileges.md#adminsdholder-group)

### DSRM क्रेडेंशियल्स

प्रत्येक **DC** में एक **स्थानीय प्रशासक** खाता होता है। इस मशीन में व्यवस्थापक अधिकार होने पर, आप मिमीकेट्ज का उपयोग करके **स्थानीय प्रशासक हैश को डंप** कर सकते हैं। फिर, इस पासवर्ड को **सक्रिय करने के लिए रजिस्ट्री को संशोधित** करके आप इस स्थानीय प्रशासक उपयोगकर्ता के लिए रिमोट रूप से पहुंच कर सकते हैं।

{% content-ref url="dsrm-credentials.md" %}
[dsrm-credentials.md](dsrm-credentials.md)
{% endcontent-ref %}

### ACL स्थिरता

आप किसी विशेष डोमेन ऑब्जेक्ट पर कुछ **विशेष अनुमतियाँ** दे सकते हैं जो भविष्य में उपयोगकर्ता को **अधिकारों का उन्नयन** करने की अनुमति देंगी।

{% content-ref url="acl-persistence-abuse/" %}
[acl-persistence-abuse](acl-persistence-abuse/)
{% endcontent-ref %}

### सुरक्षा विवरण

**सुरक्षा विवरण** का उपयोग एक ऑब्जेक्ट के ऊपर उसके **अनुमतियों** को **संग्रहित** करने के लिए किया जाता है। यदि आप केवल एक ऑब्जेक्ट के **सुरक्षा विवरण** में थोड़ा सा बदलाव कर सकते हैं, तो आप उस ऑब्जेक्ट पर ब
### विभिन्न विश्वास

यह महत्वपूर्ण है कि **एक विश्वास एकतरफा या दोतरफा हो सकता है**। दोतरफा विकल्प में, दोनों डोमेन एक-दूसरे पर विश्वास करेंगे, लेकिन **एकतरफा** विश्वास संबंध में एक डोमेन विश्वासित और दूसरा विश्वास करने वाला डोमेन होगा। इस मामले में, **आप केवल विश्वासित डोमेन से विश्वास करने वाले डोमेन के भीतर संसाधनों तक पहुंच कर सकेंगे**।

यदि डोमेन A डोमेन B पर विश्वास करता है, तो A विश्वास करने वाला डोमेन है और B विश्वासित डोमेन है। इसके अलावा, **डोमेन A** में यह एक **आउटबाउंड विश्वास** होगा; और **डोमेन B** में यह एक **इनबाउंड विश्वास** होगा।

**विभिन्न विश्वास संबंध**

* **पैरेंट-चाइल्ड** - एक ही वनस्पति का हिस्सा - एक बाल डोमेन अपने माता-पिता के साथ एक द्विपक्षीय प्रवाहित विश्वास रखता है। यह शायद वह सबसे सामान्य प्रकार का विश्वास होगा जिससे आप पर संपर्क करेंगे।
* **क्रॉस-लिंक** - बच्चा डोमेन के बीच एक "शॉर्टकट विश्वास" के रूप में - संदर्भ समयों को सुधारने के लिए। सामान्यतः एक जटिल वनस्पति में संदर्भों को वनस्पति रूट तक फ़िल्टर करना होता है और फिर लक्षित डोमेन तक वापस आना होता है, इसलिए भूगोलिक रूप से फैले हुए स्थिति के लिए, क्रॉस-लिंक समय को कम करने के लिए संभव हो सकते हैं।
* **बाह्य** - अलग-थलग डोमेन के बीच स्वतः ही गैर-प्रवाहित विश्वास। "[बाह्य विश्वास वनस्पति के बाहरी संसाधनों तक पहुंच प्रदान करते हैं जो पहले से ही वनस्पति विश्वास द्वारा जुड़ा नहीं है।](https://technet.microsoft.com/en-us/library/cc773178\(v=ws.10\).aspx)" बाह्य विश्वास एसआईडी फ़िल्टरिंग को प्रयोग करते हैं, जो इस पोस्ट के बाद में चर्चित सुरक्षा संरक्षण है।
* **ट्री-रूट** - वनस्पति रूट डोमेन और आप जो नई ट्री रूट जोड़ रहे हैं के बीच एक स्वतः ही द्विपक्षीय प्रवाहित विश्वास। मैंने ट्री-रूट विश्वास को बहुत बार नहीं देखा है, लेकिन [Microsoft दस्तावेज़ीकरण](https://technet.microsoft.com/en-us/library/cc773178\(v=ws.10\).aspx) से, यह जब आप एक नई वनस्पति वृक्ष वनस्पति में बनाते हैं तो बनाए जाते हैं। ये वन-वनस्पति विश्वास हैं, और वे [द्विपक्षीय प्रवाहितता को संरक्षित रखते हैं](https://technet.microsoft.com/en-us/library/cc757352\(v=ws.10\).aspx) जबकि वृक्ष को एक अलग डोमेन नाम (बच्चा.माता.com के बजाय) होने देते हैं।
* **वनस्पति** - दो वनस्पति रूट डोमेन के बीच एक प्रवाहित विश्वास। वनस्पति विश्वास भी एसआईडी फ़िल्टरिंग को प्रयोग करते हैं।
* **MIT** - एक गैर-विंडोज [RFC4120-संगत](https://tools.ietf.org/html/rfc4120) केरबेरोस डोमेन के साथ एक विश्वास। मैं भविष्य में MIT विश्वास में और गहराई से जाने की उम्मीद करता हूँ।

#### **विश्वास संबंधों** में अन्य अंतर

* एक विश्वास संबंध भी **प्रवाहित** (A विश्वास B, B विश्वास C, तो A विश्वास C) या **गैर-प्रवाहित** हो सकता है।
* एक विश्वास संबंध को **द्विपक्षीय विश्वास** (दोनों एक-दूसरे पर विश्वास करते हैं) या **एकतरफा विश्वास** (केवल एक विश्वास करता है) के रूप में सेट अप किया जा सकता है।

### हमला मार्ग

1. विश्वास संबंधों की **जांच** करें
2. जांचें कि क्या कोई **सुरक्षा मुख्य** (उपयोगकर्ता/समूह/कंप्यूटर) **दूसरे डोमेन** के संसाधनों का **पहुंच** रखता है, शायद ACE प्रविष्टियों द्वारा या दूसरे डोमेन के समूहों में होने के कारण। **डोमेनों के बीच संबंधों** की तलाश करें (विश्वास इसके लिए बनाया गया होगा शायद)।
1. इस मामले में kerberoast एक और विकल्प हो सकता है।
3. विश्वास करने वाले खातों को **अधिकार** दें जो डोमेन के माध्यम से **पिवट** कर सकते हैं।

एक डोमेन से दूसरे विदेशी/विश्वास करने वाले डोमेन में सुरक्षा मुख्य (उपयोगकर्ता/समूह/कंप्यूटर) के तीन **मुख्य** तरीके हैं जिनसे संसाधनों में पहुंच हो सकती है:

* वे व्यक्तिगत मशीनों पर **स्थानीय समूहों** में जोड़े जा सकते हैं, जैसे कि सर्वर पर स्थ
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
यहाँ **2 विश्वसनीय कुंजी** हैं, एक _बाल --> माता-पिता_ के लिए और दूसरी _माता-पिता_ --> _बाल_ के लिए।\
आप मौजूदा डोमेन द्वारा उपयोग की जाने वाली कुंजी को इस्तेमाल कर सकते हैं:
```bash
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\mcorp$"'
```
{% endhint %}

#### SID-History Injection

ट्रस्ट के साथ बच्चा / माता-पिता डोमेन को एंटरप्राइज एडमिन के रूप में उन्नत करें, SID-History इंजेक्शन का दुरुपयोग करके:

{% content-ref url="sid-history-injection.md" %}
[sid-history-injection.md](sid-history-injection.md)
{% endcontent-ref %}

#### Exploit writeable Configuration NC

Configuration NC एक वनस्पति के लिए विन्यास सूचना का प्राथमिक संग्रह है और इसे वनस्पति में हर DC में प्रतिलिपि की जाती है। इसके अलावा, वनस्पति में हर लिखने योग्य DC (वापसी नहीं करने वाले DCs) वनस्पति के एक लिखने योग्य प्रतिलिपि को रखता है। इसे उत्पन्न करने के लिए (बच्चा) DC पर सिस्टम के रूप में चलाने की आवश्यकता होती है।

यह संभव है कि विभिन्न तरीकों से मूल डोमेन को प्रभावित किया जा सकता है, जो नीचे चर्चित हैं।

##### रूट DC साइट को GPO से लिंक करें
Configuration NC में साइट्स कंटेनर एडी वनस्पति में शामिल कंप्यूटरों की सभी साइट्स को संगठित करता है। संगठन में किसी भी DC के रूप में सिस्टम के रूप में चलाते समय, इसमें शामिल हैं वनस्पति के रूट DCs की साइट (और इसलिए इन्हें प्रभावित करें) को GPO से लिंक किया जा सकता है।

अधिक विवरण यहां पढ़ें: [Bypass SID filtering research](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-4-bypass-sid-filtering-research).

##### वनस्पति में किसी भी gMSA को प्रभावित करें
हमला लक्षित डोमेन में विशेषाधिकारी gMSAs पर निर्भर करता है।

वनस्पति में gMSAs के पासवर्ड की गणना के लिए उपयोग किए जाने वाले KDS रूट कुंजी को Configuration NC में संग्रहीत किया जाता है। वनस्पति में किसी भी DC के रूप में सिस्टम के रूप में चलाते समय, एक व्यक्ति KDS रूट कुंजी को पढ़ सकता है और वनस्पति में किसी भी gMSA का पासवर्ड गणना कर सकता है।

अधिक विवरण यहां पढ़ें: [Golden gMSA trust attack from child to parent](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-5-golden-gmsa-trust-attack-from-child-to-parent).

##### स्कीमा बदलने का हमला
हमलावर को नए विशेषाधिकारी AD ऑब्जेक्ट्स के निर्माण के लिए प्रतीक्षा करने की आवश्यकता होती है।

वनस्पति में किसी भी DC के रूप में सिस्टम के रूप में चलाते समय, एक व्यक्ति सभी कक्षाओं पर किसी भी उपयोगकर्ता को पूर्ण नियंत्रण प्रदान कर सकता है। यह नियंत्रण उपयोग करके एक ACE बनाया जा सकता है जो किसी भी संकटग्रस्त प्रमुखलग्नक को पूर्ण नियंत्रण प्रदान करने वाले संकटग्रस्त प्रमुखलग्नक के डिफ़ॉल्ट सुरक्षा विवरण में बनाया जाता है। संशोधित AD ऑब्जेक्ट प्रकार के सभी नए उदाहरणों में इस ACE होगा।

अधिक विवरण यहां पढ़ें: [Schema change trust attack from child to parent](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-6-schema-change-trust-attack-from-child-to-parent).

##### DA से EA तक ADCS ESC5 के साथ
ADCS ESC5 (विकल्पनीय PKI ऑब्जेक्ट पहुंच नियंत्रण) हमले PKI ऑब्जेक्ट पर नियंत्रण का दुरुपयोग करते हैं ताकि एक ऐसा विकल्पनीय प्रमाणपत्र बनाया जा सके जिसका दुरुपयोग वनस्पति में किसी भी उपयोगकर्ता के रूप में किया जा सके। क्योंकि सभी PKI ऑब्जेक्ट्स Configuration NC में संग्रहीत होते हैं, इसलिए यदि किसी भी लिखने योग्य (बच्चा) DC में हमलावर ने प्रभावित किया है, तो वह ESC5 को क्रियान्वित कर सकता है।

अधिक विवरण यहां पढ़ें: [From DA to EA with ESC5](https://posts.specterops.io/from-da-to-ea-with-esc5-f9f045aa105c)

यदि AD वनस्पति में ADCS नहीं है, तो हमलावर आवश्यक घटक बना सकता है जैसा कि यहां वर्णित है: [Escalating from child domain’s admins to enterprise admins in 5 minutes by abusing AD CS, a follow up](https://www.pkisolutions.com/escalating-from-child-domains-admins-to-enterprise-admins-in-5-minutes-by-abusing-ad-cs-a-follow-up/).

### बाहरी वनस्पति डोमेन - एकतरफा (इनबाउंड) या द्विदिशीय
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
इस परिदृश्य में **आपके डोमेन को एक बाहरी डोमेन द्वारा विश्वसनीयता प्राप्त है** जिससे आपको इस पर अनिर्धारित अनुमतियाँ मिलती हैं। आपको **ढूंढ़ना होगा कि आपके डोमेन के कौन से प्रमुख बाहरी डोमेन पर कौन सी पहुंच रखते हैं** और फिर इसे उत्पन्न करने का प्रयास करें:

{% content-ref url="external-forest-domain-oneway-inbound.md" %}
[external-forest-domain-oneway-inbound.md](external-forest-domain-oneway-inbound.md)
{% endcontent-ref %}

### बाहरी वन डोमेन - एक-तरफा (आउटबाउंड)
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
इस स्थिति में **आपका डोमेन** किसी **विशेषाधिकार** को किसी **अलग डोमेन** के प्रिंसिपल को **विश्वास कर रहा है**।

हालांकि, जब एक **डोमेन विश्वास करता है** विश्वास करने वाले डोमेन द्वारा, विश्वासित डोमेन **एक प्रयोगकर्ता बनाता है** जिसका नाम **पूर्वानुमानित होता है** और जिसका **पासवर्ड विश्वासित पासवर्ड के रूप में उपयोग किया जाता है**। इसका मतलब है कि यह संभव है कि आप **विश्वास करने वाले डोमेन में प्रवेश करने के लिए विश्वास करने वाले डोमेन से एक प्रयोगकर्ता तक पहुंच सकते हैं** और इसे जांचने और अधिक विशेषाधिकार को उन्नत करने का प्रयास कर सकते हैं:

{% content-ref url="external-forest-domain-one-way-outbound.md" %}
[external-forest-domain-one-way-outbound.md](external-forest-domain-one-way-outbound.md)
{% endcontent-ref %}

विश्वासित डोमेन को संकट में डालने का एक और तरीका है **एक [SQL विश्वासित लिंक](abusing-ad-mssql.md#mssql-trusted-links) का पता लगाना** जो डोमेन विश्वास के **उलट दिशा** में बनाया गया है (जो बहुत आम नहीं है)।

विश्वासित डोमेन को संकट में डालने का एक और तरीका है कि एक मशीन में प्रतीक्षा करें जहां से विश्वासित डोमेन का एक प्रयोगकर्ता पहुंच कर सकता है और **RDP** के माध्यम से लॉगिन कर सकता है। फिर, हमलावर RDP सत्र प्रक्रिया में कोड इंजेक्शन कर सकते हैं और वहां से पीड़ित का मूल डोमेन **तक पहुंच सकते हैं**।
इसके अलावा, यदि **पीड़ित ने अपनी हार्ड ड्राइव माउंट की है**, तो हमलावर RDP सत्र प्रक्रिया से आक्रमणकारी **हार्ड ड्राइव के स्टार्टअप फ़ोल्डर में बैकडोर्स** संग्रहित कर सकता है। इस तकनीक को **RDPInception** कहा जाता है।

{% content-ref url="rdp-sessions-abuse.md" %}
[rdp-sessions-abuse.md](rdp-sessions-abuse.md)
{% endcontent-ref %}

### डोमेन विश्वास दुरुपयोग निवारण

**SID फ़िल्टरिंग:**

* वन वन विश्वास में SID इतिहास विश्वास करने वाले हमलों से बचें।
* सभी इंटर-वन विश्वासों पर डिफ़ॉल्ट रूप से सक्षम है। इंट्रा-वन विश्वासों को डिफ़ॉल्ट रूप से सुरक्षित माना जाता है (MS को वन नहीं डोमेन को सुरक्षा सीमा माना जाता है)।
* लेकिन, SID फ़िल्टरिंग को अनुप्रयोगों और प्रयोगकर्ता पहुंच को तोड़ने की संभावना होती है, इसलिए यह अक्सर अक्षम हो जाता है।
* चयनात्मक प्रमाणीकरण
* इंटर-वन विश्वास में, यदि चयनात्मक प्रमाणीकरण कॉन्फ़िगर किया गया है, तो विश्वास करने वाले डोमेन / वन में डोमेन और सर्वरों के बीच के प्रयोगकर्ताओं को स्वचालित रूप से प्रमाणित नहीं किया जाएगा। विश्वास करने वाले डोमेन / वन में डोमेन और सर्वरों के लिए व्यक्तिगत पहुंच दी जानी चाहिए।
* लिखने योग्य कॉन्फ़िगरेशन NC शोषण और विश्वास खाता हमला रोकता है।

[**ired.team में डोमेन विश्वास के बारे में अधिक जानकारी।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain)

## AD -> Cloud & Cloud -> AD

{% embed url="https://cloud.hacktricks.xyz/pentesting-cloud/azure-security/az-lateral-movements/azure-ad-connect-hybrid-identity" %}

## कुछ सामान्य सुरक्षा उपाय

[**यहां क्रेडेंशियल की सुरक्षा कैसे करें के बारे में अधिक जानें।**](../stealing-credentials/credentials-protections.md)\
**कृपया, तकनीक की
