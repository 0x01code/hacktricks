# Active Directory पद्धति

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## मूल अवलोकन

Active Directory नेटवर्क प्रशासकों को डोमेन, उपयोगकर्ता और नेटवर्क के भीतर ऑब्जेक्ट्स बनाने और प्रबंधित करने की अनुमति देता है। उदाहरण के लिए, एक प्रशासक उपयोगकर्ताओं का एक समूह बना सकता है और उन्हें सर्वर पर कुछ निर्देशिकाओं तक विशिष्ट पहुंच अधिकार दे सकता है। जैसे-जैसे नेटवर्क बढ़ता है, Active Directory बड़ी संख्या में उपयोगकर्ताओं को तार्किक समूहों और उपसमूहों में व्यवस्थित करने और प्रत्येक स्तर पर पहुंच नियंत्रण प्रदान करने का एक तरीका प्रदान करता है।

Active Directory संरचना में तीन मुख्य स्तर शामिल हैं: 1) डोमेन, 2) वृक्ष, और 3) वन। कई ऑब्जेक्ट्स (उपयोगकर्ता या उपकरण) जो एक ही डेटाबेस का उपयोग करते हैं, एक ही डोमेन में समूहित किए जा सकते हैं। कई डोमेन को एक ही समूह में जोड़ा जा सकता है जिसे वृक्ष कहा जाता है। कई वृक्षों को एक संग्रह में समूहित किया जा सकता है जिसे वन कहा जाता है। इन स्तरों में से प्रत्येक को विशिष्ट पहुंच अधिकार और संचार विशेषाधिकार दिए जा सकते हैं।

Active Directory की मुख्य अवधारणाएँ:

1. **निर्देशिका** – Active directory के ऑब्जेक्ट्स की सभी जानकारी शामिल होती है
2. **ऑब्जेक्ट** – एक ऑब्जेक्ट निर्देशिका के भीतर लगभग किसी भी चीज को संदर्भित करता है (एक उपयोगकर्ता, समूह, साझा फ़ोल्डर...)
3. **डोमेन** – निर्देशिका के ऑब्जेक्ट्स डोमेन के भीतर समाहित होते हैं। "वन" के भीतर एक से अधिक डोमेन हो सकते हैं और प्रत्येक के अपने ऑब्जेक्ट्स संग्रह होंगे।
4. **वृक्ष** – एक ही मूल के साथ डोमेन का समूह। उदाहरण: _dom.local, email.dom.local, www.dom.local_
5. **वन** – वन संगठन पदानुक्रम का सबसे ऊंचा स्तर है और यह वृक्षों के समूह से बना होता है। वृक्ष विश्वास संबंधों द्वारा जुड़े होते हैं।

Active Directory कई अलग-अलग सेवाएँ प्रदान करता है, जो "Active Directory डोमेन सेवाओं" या AD DS के छत्र के नीचे आती हैं। इन सेवाओं में शामिल हैं:

1. **डोमेन सेवाएँ** – केंद्रीकृत डेटा संग्रहीत करता है और उपयोगकर्ताओं और डोमेन के बीच संचार का प्रबंधन करता है; लॉगिन प्रमाणीकरण और खोज कार्यक्षमता शामिल है
2. **प्रमाणपत्र सेवाएँ** – सुरक्षित प्रमाणपत्र बनाता है, वितरित करता है और प्रबंधित करता है
3. **लाइटवेट निर्देशिका सेवाएँ** – खुले (LDAP) प्रोटोकॉल का उपयोग करके निर्देशिका-सक्षम अनुप्रयोगों का समर्थन करता है
4. **निर्देशिका फेडरेशन सेवाएँ** – एकल सत्र में कई वेब अनुप्रयोगों में एक उपयोगकर्ता को प्रमाणित करने के लिए सिंगल-साइन-ऑन (SSO) प्रदान करता है
5. **अधिकार प्रबंधन** – डिजिटल सामग्री के अनधिकृत उपयोग और वितरण से बचाकर कॉपीराइट सूचना की सुरक्षा करता है
6. **DNS सेवा** – डोमेन नामों को हल करने के लिए उपयोग की जाती है।

AD DS Windows Server (Windows Server 10 सहित) के साथ शामिल है और क्लाइंट सिस्टम का प्रबंधन करने के लिए डिजाइन किया गया है। जबकि नियमित संस्करण के Windows चलाने वाले सिस्टमों में AD DS की प्रशासनिक विशेषताएँ नहीं होती हैं, वे Active Directory का समर्थन करते हैं। इसका मतलब है कि कोई भी Windows कंप्यूटर एक Windows कार्य समूह से जुड़ सकता है, बशर्ते उपयोगकर्ता के पास सही लॉगिन क्रेडेंशियल हों।\
**से:** [**https://techterms.com/definition/active\_directory**](https://techterms.com/definition/active\_directory)

### **Kerberos प्रमाणीकरण**

AD पर **हमला करने के लिए** आपको **Kerberos प्रमाणीकरण प्रक्रिया** को वास्तव में अच्छी तरह से **समझना** होगा।\
[**यह पृष्ठ पढ़ें यदि आप अभी भी नहीं जानते कि यह कैसे काम करता है।**](kerberos-authentication.md)

## चीट शीट

आप [https://wadcoms.github.io/](https://wadcoms.github.io) पर जा सकते हैं ताकि आप जल्दी से देख सकें कि आप AD को सूचीबद्ध करने/शोषण करने के लिए कौन से कमांड चला सकते हैं।

## Recon Active Directory (कोई क्रेडेंशियल/सत्र नहीं)

यदि आपके पास AD वातावरण तक पहुँच है लेकिन कोई क्रेडेंशियल/सत्र नहीं है, तो आप कर सकते हैं:

* **नेटवर्क का पेंटेस्ट करें:**
* नेटवर्क को स्कैन करें, मशीनें और खुले पोर्ट्स ढूँढें और कोशिश करें कि **कमजोरियों का शोषण करें** या **उनसे क्रेडेंशियल निकालें** (उदाहरण के लिए, [प्रिंटर्स बहुत दिलचस्प लक्ष्य हो सकते हैं](ad-information-in-printers.md)).
* DNS का सूचीबद्ध करना डोमेन के महत्वपूर्ण सर्वरों के बारे में जानकारी दे सकता है जैसे वेब, प्रिंटर्स, शेयर्स, वीपीएन, मीडिया, आदि।
* `gobuster dns -d domain.local -t 25 -w /opt/Seclist/Discovery/DNS/subdomain-top2000.txt`
* इसे कैसे करने के बारे में अधिक जानकारी के लिए सामान्य [**पेंटेस्टिंग पद्धति**](../../generic-methodologies-and-resources/pentesting-methodology.md) देखें।
* **smb सेवाओं पर नल और अतिथि पहुँच की जाँच करें** (यह आधुनिक Windows संस्करणों पर काम नहीं करेगा):
* `enum4linux -a -u "" -p "" <DC IP> && enum4linux -a -u "guest" -p "" <DC IP>`
* `smbmap -
```bash
./kerbrute_linux_amd64 userenum -d lab.ropnop.com --dc 10.10.10.10 usernames.txt #From https://github.com/ropnop/kerbrute/releases

nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='DOMAIN'" <IP>
Nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm='<domain>',userdb=/root/Desktop/usernames.txt <IP>

msf> use auxiliary/gather/kerberos_enumusers

crackmapexec smb dominio.es  -u '' -p '' --users | awk '{print $4}' | uniq
```
* **OWA (Outlook Web Access) सर्वर**

यदि आपको नेटवर्क में इनमें से कोई सर्वर मिलता है, तो आप **इसके खिलाफ उपयोगकर्ता सूचीकरण (user enumeration) कर सकते हैं**। उदाहरण के लिए, आप [**MailSniper**](https://github.com/dafthack/MailSniper) टूल का उपयोग कर सकते हैं:
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
आप इस GitHub रेपो में उपयोगकर्ता नामों की सूचियाँ पा सकते हैं [**this github repo**](https://github.com/danielmiessler/SecLists/tree/master/Usernames/Names) \*\*\*\* और इसमें भी ([**statistically-likely-usernames**](https://github.com/insidetrust/statistically-likely-usernames))।

हालांकि, आपके पास कंपनी में काम करने वाले लोगों के **नाम** पहले से होने चाहिए जो आपने इससे पहले के चरण में पता लगाए होंगे। नाम और उपनाम के साथ आप स्क्रिप्ट [**namemash.py**](https://gist.github.com/superkojiman/11076951) का उपयोग कर संभावित मान्य उपयोगकर्ता नाम उत्पन्न कर सकते हैं।
{% endhint %}

### एक या अधिक उपयोगकर्ता नाम जानना

ठीक है, तो आप जानते हैं कि आपके पास पहले से ही एक मान्य उपयोगकर्ता नाम है लेकिन कोई पासवर्ड नहीं... तो कोशिश करें:

* [**ASREPRoast**](asreproast.md): यदि किसी उपयोगकर्ता के पास _DONT_REQ_PREAUTH_ विशेषता **नहीं है**, तो आप उस उपयोगकर्ता के लिए **AS_REP संदेश का अनुरोध कर सकते हैं** जिसमें उपयोगकर्ता के पासवर्ड के व्युत्पन्न से एन्क्रिप्ट किया गया कुछ डेटा होगा।
* [**Password Spraying**](password-spraying.md): आइए सबसे **आम पासवर्ड** की कोशिश करें प्रत्येक खोजे गए उपयोगकर्ताओं के साथ, शायद कुछ उपयोगकर्ता एक खराब पासवर्ड का उपयोग कर रहे हों (पासवर्ड नीति को ध्यान में रखें!)।
* ध्यान दें कि आप **OWA सर्वरों पर भी स्प्रे कर सकते हैं** ताकि उपयोगकर्ताओं के मेल सर्वरों तक पहुँच प्राप्त कर सकें।

{% content-ref url="password-spraying.md" %}
[password-spraying.md](password-spraying.md)
{% endcontent-ref %}

### LLMNR/NBT-NS Poisoning

आप **नेटवर्क** के कुछ प्रोटोकॉल को **poisoning** करके कुछ चैलेंज **हैशेस** प्राप्त करने में सक्षम हो सकते हैं:

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

### NTML Relay

यदि आपने सक्रिय निर्देशिका का अनुक्रमण किया है, तो आपके पास **अधिक ईमेल और नेटवर्क की बेहतर समझ** होगी। आप NTML [**relay attacks**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack) को मजबूर करने में सक्षम हो सकते हैं ताकि AD पर्यावरण तक पहुँच प्राप्त कर सकें।

### Steal NTLM Creds

यदि आप **null या guest उपयोगकर्ता** के साथ अन्य पीसी या शेयरों तक **पहुँच सकते हैं**, तो आप **फाइलें रख सकते हैं** (जैसे कि SCF फाइल) जो यदि किसी तरह से एक्सेस की जाती हैं तो वे **आपके खिलाफ NTML प्रमाणीकरण ट्रिगर करेंगी** ताकि आप **NTLM चैलेंज को चुरा सकें** और उसे क्रैक कर सकें:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

## साखों/सत्र के साथ सक्रिय निर्देशिका का अनुक्रमण

इस चरण के लिए आपके पास एक मान्य डोमेन खाते की **साखें या सत्र समझौता किया गया होना चाहिए।** यदि आपके पास कुछ मान्य साखें हैं या आप एक डोमेन उपयोगकर्ता के रूप में एक शेल हैं, तो आपको याद रखना चाहिए कि अन्य उपयोगकर्ताओं को समझौता करने के लिए पहले दिए गए विकल्प अभी भी विकल्प हैं।

प्रमाणित अनुक्रमण शुरू करने से पहले आपको पता होना चाहिए कि **Kerberos double hop problem** क्या है।

{% content-ref url="kerberos-double-hop-problem.md" %}
[kerberos-double-hop-problem.md](kerberos-double-hop-problem.md)
{% endcontent-ref %}

### अनुक्रमण

एक खाते को समझौता करना पूरे डोमेन को समझौता करने के लिए **एक बड़ा कदम है**, क्योंकि आप **सक्रिय निर्देशिका अनुक्रमण** शुरू करने में सक्षम होंगे:

[**ASREPRoast**](asreproast.md) के बारे में अब आप हर संभावित कमजोर उपयोगकर्ता को ढूंढ सकते हैं, और [**Password Spraying**](password-spraying.md) के बारे में आप सभी उपयोगकर्ता नामों की **सूची प्राप्त कर सकते हैं** और समझौता किए गए खाते का पासवर्ड, खाली पासवर्ड और नए आशाजनक पासवर्ड की कोशिश कर सकते हैं।

* आप [**CMD का उपयोग करके एक बुनियादी रेकॉन कर सकते हैं**](../basic-cmd-for-pentesters.md#domain-info)
* आप [**पावरशेल का उपयोग करके रेकॉन के लिए**](../basic-powershell-for-pentesters/) भी कर सकते हैं जो अधिक चुपके होगा
* आप [**पावरव्यू का उपयोग करके**](../basic-powershell-for-pentesters/powerview.md) अधिक विस्तृत जानकारी निकाल सकते हैं
* सक्रिय निर्देशिका में रेकॉन के लिए एक और अद्भुत उपकरण [**BloodHound**](bloodhound.md) है। यह **बहुत चुपके नहीं है** (आप जो संग्रह विधियां उपयोग करते हैं उसके आधार पर), लेकिन **यदि आपको परवाह नहीं है** तो आपको इसे जरूर आजमाना चाहिए। जानिए कहाँ उपयोगकर्ता RDP कर सकते हैं, अन्य समूहों के लिए पथ ढूंढें, आदि।
* **अन्य स्वचालित AD अनुक्रमण उपकरण हैं:** [**AD Explorer**](bloodhound.md#ad-explorer)**,** [**ADRecon**](bloodhound.md#adrecon)**,** [**Group3r**](bloodhound.md#group3r)**,** [**PingCastle**](bloodhound.md#pingcastle)**.**
* [**AD के DNS रिकॉर्ड**](ad-dns-records.md) क्योंकि वे दिलचस्प जानकारी शामिल कर सकते हैं।
* एक **GUI के साथ उपकरण** जिसका उपयोग आप निर्देशिका का अनुक्रमण करने के लिए कर सकते हैं वह है **AdExplorer.exe** **SysInternal** सुइट से।
* आप **ldapsearch** का उपयोग करके LDAP डेटाबेस में भी खोज कर सकते हैं ताकि _userPassword_ और _unixUserPassword_ फ़ील्ड्स में, या यहां तक कि _Description_ में साखों की तलाश कर सकें। cf. [PayloadsAllTheThings पर AD उपयोगकर्ता टिप्पणी में पासवर्ड](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#password-in-ad-user-comment) अन्य विधियों के लिए।
* यदि आप **लिनक्स** का उपयोग कर रहे हैं, तो आप [**pywerview**](https://github.com/the-useless-one/pywerview) का उपयोग करके डोमेन का अनुक्रमण भी कर सकते हैं।
* आप स्वचालित उपकरणों की भी कोशिश कर सकते हैं जैसे:
* [**tomcarver16/ADSearch**](https://github.com/tomcarver16/ADSearch)
* [**61106960/adPEAS**](https://github.com/61106960/adPEAS)
*   **सभी डोमेन उपयोगकर्ताओं को निकालना**

विंडोज से सभी डोमेन उपयोगकर्ता नाम प्राप्त करना बहुत आसान है (`net user /domain`, `Get-DomainUser` या `wmic useraccount get name,sid`)। लिनक्स में, आप उपयोग कर सकते हैं: `GetADUsers.py -all -dc-ip 10.10.10.110 domain.com/username` या `enum4linux -a -u "user" -p "password" <DC IP>`

> भले ही यह अनुक्रमण अनुभाग छोटा लगता है, यह सबसे महत्वपूर्ण
```bash
## List all tickets (if not admin, only current user tickets)
.\Rubeus.exe triage
## Dump the interesting one by luid
.\Rubeus.exe dump /service:krbtgt /luid:<luid> /nowrap
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))
```
### NTML Relay

यदि आपने सक्रिय निर्देशिका का संचयन कर लिया है तो आपके पास **अधिक ईमेल और नेटवर्क की बेहतर समझ** होगी। आप NTML [**रिले हमलों**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack) को मजबूर कर सकते हैं।

### **कंप्यूटर शेयर्स में क्रेड्स खोजें**

अब जब आपके पास कुछ मूलभूत क्रेडेंशियल्स हैं, तो आपको चेक करना चाहिए कि क्या आप **AD के अंदर साझा की जा रही कोई दिलचस्प फाइलें पा** सकते हैं। आप यह मैन्युअली कर सकते हैं लेकिन यह एक बहुत ही उबाऊ और दोहराव वाला कार्य है (और और भी अगर आपको सैकड़ों दस्तावेज़ चेक करने की जरूरत हो)।

[**इस लिंक का अनुसरण करें और जानें कि आप कौन से उपकरण उपयोग कर सकते हैं।**](../../network-services-pentesting/pentesting-smb.md#domain-shared-folders-search)

### Steal NTLM Creds

यदि आप **अन्य पीसी या शेयर्स तक पहुँच** सकते हैं तो आप **फाइलें रख** सकते हैं (जैसे कि SCF फाइल) जिसे अगर किसी तरह से एक्सेस किया जाता है तो यह **आपके खिलाफ NTML प्रमाणीकरण को ट्रिगर** करेगा ताकि आप **NTLM चैलेंज को चुरा** सकें और उसे क्रैक कर सकें:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### CVE-2021-1675/CVE-2021-34527 PrintNightmare

इस कमजोरी ने किसी भी प्रमाणित उपयोगकर्ता को **डोमेन कंट्रोलर को समझौता** करने की अनुमति दी थी।

{% content-ref url="printnightmare.md" %}
[printnightmare.md](printnightmare.md)
{% endcontent-ref %}

## Privilege escalation on Active Directory WITH privileged credentials/session

**निम्नलिखित तकनीकों के लिए एक सामान्य डोमेन उपयोगकर्ता पर्याप्त नहीं है, आपको इन हमलों को करने के लिए कुछ विशेष विशेषाधिकार/क्रेडेंशियल्स की आवश्यकता होती है।**

### Hash extraction

आशा है कि आपने [AsRepRoast](asreproast.md), [Password Spraying](password-spraying.md), [Kerberoast](kerberoast.md), [Responder](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md) सहित रिलेइंग, [EvilSSDP](../../generic-methodologies-and-resources/pentesting-network/spoofing-ssdp-and-upnp-devices.md), [स्थानीय रूप से विशेषाधिकार बढ़ाना](../windows-local-privilege-escalation/) का उपयोग करके किसी स्थानीय व्यवस्थापक खाते को **समझौता** कर लिया है।\
तब, यह समय है सभी हैशेज को मेमोरी और स्थानीय रूप से डंप करने का।\
[**इस पृष्ठ को पढ़ें और विभिन्न तरीकों के बारे में जानें जिनसे हैशेज प्राप्त किए जा सकते हैं।**](broken-reference/)

### Pass the Hash

**एक बार जब आपके पास उपयोगकर्ता का हैश हो**, आप उसका उपयोग करके उसे **अनुकरण** कर सकते हैं।\
आपको किसी **उपकरण** का उपयोग करना होगा जो उस **हैश का उपयोग करके NTLM प्रमाणीकरण करेगा**, **या** आप एक नया **sessionlogon बना** सकते हैं और उस **हैश को LSASS के अंदर इंजेक्ट** कर सकते हैं, ताकि जब भी कोई **NTLM प्रमाणीकरण किया जाता है**, वह **हैश उपयोग किया जाएगा।** आखिरी विकल्प वह है जो mimikatz करता है।\
[**अधिक जानकारी के लिए इस पृष्ठ को पढ़ें।**](../ntlm/#pass-the-hash)

### Over Pass the Hash/Pass the Key

यह हमला **उपयोगकर्ता के NTLM हैश का उपयोग करके Kerberos टिकट का अनुरोध करने के लिए** किया जाता है, NTLM प्रोटोकॉल के ऊपर Pass The Hash के सामान्य विकल्प के रूप में। इसलिए, यह विशेष रूप से **नेटवर्क में उपयोगी हो सकता है जहां NTLM प्रोटोकॉल अक्षम है** और केवल **Kerberos को प्रमाणीकरण प्रोटोकॉल के रूप में अनुमति है**।

{% content-ref url="over-pass-the-hash-pass-the-key.md" %}
[over-pass-the-hash-pass-the-key.md](over-pass-the-hash-pass-the-key.md)
{% endcontent-ref %}

### Pass the Ticket

यह हमला Pass the Key के समान है, लेकिन टिकट का अनुरोध करने के लिए हैशेज का उपयोग करने के बजाय, **टिकट स्वयं चुराई जाती है** और उसके मालिक के रूप में प्रमाणित करने के लिए उपयोग की जाती है।

{% content-ref url="pass-the-ticket.md" %}
[pass-the-ticket.md](pass-the-ticket.md)
{% endcontent-ref %}

### Credentials Reuse

यदि आपके पास **हैश** या **पासवर्ड** किसी **स्थानीय प्रशासक** का है तो आपको अन्य **पीसी** पर **स्थानीय रूप से लॉगिन** करने का प्रयास करना चाहिए।
```bash
# Local Auth Spray (once you found some local admin pass or hash)
## --local-auth flag indicate to only try 1 time per machine
crackmapexec smb --local-auth 10.10.10.10/23 -u administrator -H 10298e182387f9cab376ecd08491764a0 | grep +
```
{% hint style="warning" %}
ध्यान दें कि यह काफी **noisy** है और **LAPS** इसे **mitigate** करेगा।
{% endhint %}

### MSSQL Abuse & Trusted Links

यदि किसी उपयोगकर्ता को **MSSQL instances तक पहुंचने की अनुमति** है, तो वह इसका उपयोग MSSQL होस्ट में **commands execute** करने के लिए कर सकता है (यदि SA के रूप में चल रहा हो), **NetNTLM hash चुरा** सकता है या यहां तक कि **relay attack** भी कर सकता है।\
इसके अलावा, यदि कोई MSSQL instance विश्वासपात्र (database link) है जिसे एक अलग MSSQL instance द्वारा विश्वास किया जाता है। यदि उपयोगकर्ता को विश्वासपात्र डेटाबेस पर अधिकार हैं, तो वह **विश्वास संबंध का उपयोग करके दूसरे instance में भी queries execute करने में सक्षम होगा**। ये विश्वास श्रृंखलाबद्ध हो सकते हैं और किसी बिंदु पर उपयोगकर्ता को एक गलत कॉन्फ़िगर डेटाबेस मिल सकता है जहां वह commands execute कर सकता है।\
**डेटाबेस के बीच के लिंक forest trusts के पार भी काम करते हैं।**

{% content-ref url="abusing-ad-mssql.md" %}
[abusing-ad-mssql.md](abusing-ad-mssql.md)
{% endcontent-ref %}

### Unconstrained Delegation

यदि आपको कोई Computer object [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx) विशेषता के साथ मिलती है और आपके पास कंप्यूटर में डोमेन अधिकार हैं, तो आप हर उस उपयोगकर्ता के TGTs को मेमोरी से dump करने में सक्षम होंगे जो कंप्यूटर पर लॉगिन करता है।\
इसलिए, यदि कोई **Domain Admin कंप्यूटर पर लॉगिन करता है**, तो आप उसके TGT को dump कर सकते हैं और [Pass the Ticket](pass-the-ticket.md) का उपयोग करके उसका अनुकरण कर सकते हैं।\
Constrained delegation की मदद से आप **automatically Print Server को compromise** भी कर सकते हैं (आशा है कि यह एक DC होगा)।

{% content-ref url="unconstrained-delegation.md" %}
[unconstrained-delegation.md](unconstrained-delegation.md)
{% endcontent-ref %}

### Constrained Delegation

यदि किसी उपयोगकर्ता या कंप्यूटर को "Constrained Delegation" की अनुमति है, तो वह किसी कंप्यूटर में कुछ सेवाओं तक पहुंचने के लिए **किसी भी उपयोगकर्ता का अनुकरण कर सकता है**।\
फिर, यदि आप इस उपयोगकर्ता/कंप्यूटर का **hash compromise** करते हैं, तो आप कुछ सेवाओं तक पहुंचने के लिए **किसी भी उपयोगकर्ता का अनुकरण कर सकते हैं** (यहां तक कि domain admins का भी)।

{% content-ref url="constrained-delegation.md" %}
[constrained-delegation.md](constrained-delegation.md)
{% endcontent-ref %}

### Resourced-based Constrain Delegation

यदि आपके पास किसी रिमोट कंप्यूटर के AD object पर WRITE अधिकार हैं, तो आप **elevated privileges के साथ कोड execution प्राप्त कर सकते हैं**।

{% content-ref url="resource-based-constrained-delegation.md" %}
[resource-based-constrained-delegation.md](resource-based-constrained-delegation.md)
{% endcontent-ref %}

### ACLs Abuse

समझौता किए गए उपयोगकर्ता के पास कुछ डोमेन ऑब्जेक्ट्स पर **रोचक अधिकार** हो सकते हैं जो आपको **लेटरल मूव**/**अधिकार बढ़ाने** में मदद कर सकते हैं।

{% content-ref url="acl-persistence-abuse/" %}
[acl-persistence-abuse](acl-persistence-abuse/)
{% endcontent-ref %}

### Printer Spooler service abuse

यदि आप किसी भी **Spool service को सुनते हुए** डोमेन के अंदर पा सकते हैं, तो आप इसका **दुरुपयोग** करके **नई credentials प्राप्त कर सकते हैं** और **अधिकार बढ़ा सकते हैं**।\
[**Spooler services का दुरुपयोग कैसे करें इसकी अधिक जानकारी यहां है।**](printers-spooler-service-abuse.md)

### Third party sessions abuse

यदि **अन्य उपयोगकर्ता** **access** करते हैं **compromised** मशीन को, तो संभव है कि **मेमोरी से credentials एकत्र करें** और यहां तक कि **उनकी प्रक्रियाओं में beacons inject करें** उनका अनुकरण करने के लिए।\
आमतौर पर उपयोगकर्ता RDP के माध्यम से सिस्टम तक पहुंचेंगे, इसलिए यहां आपके पास third party RDP sessions पर हमला करने के लिए कुछ तरीके हैं:

{% content-ref url="rdp-sessions-abuse.md" %}
[rdp-sessions-abuse.md](rdp-sessions-abuse.md)
{% endcontent-ref %}

### LAPS

**LAPS** आपको **local Administrator password का प्रबंधन करने** की अनुमति देता है (जो **randomised**, अद्वितीय है, और **नियमित रूप से बदला जाता है**) डोमेन-जुड़े कंप्यूटरों पर। ये पासवर्ड Active Directory में केंद्रीय रूप से संग्रहीत होते हैं और ACLs का उपयोग करके अधिकृत उपयोगकर्ताओं तक सीमित होते हैं। यदि आपके पास **इन पासवर्डों को पढ़ने के लिए पर्याप्त अनुमति है तो आप अन्य कंप्यूटरों पर जा सकते हैं**।

{% content-ref url="laps.md" %}
[laps.md](laps.md)
{% endcontent-ref %}

### Certificate Theft

समझौता किए गए मशीन से प्रमाणपत्र एकत्र करना वातावरण के अंदर अधिकार बढ़ाने का एक तरीका हो सकता है:

{% content-ref url="ad-certificates/certificate-theft.md" %}
[certificate-theft.md](ad-certificates/certificate-theft.md)
{% endcontent-ref %}

### Certificate Templates Abuse

यदि कमजोर टेम्पलेट्स कॉन्फ़िगर किए गए हैं तो उनका दुरुपयोग करके अधिकार बढ़ाना संभव है:

{% content-ref url="ad-certificates/domain-escalation.md" %}
[domain-escalation.md](ad-certificates/domain-escalation.md)
{% endcontent-ref %}

## Post-exploitation with high privilege account

### Dumping Domain Credentials

एक बार जब आप **Domain Admin** या और भी बेहतर **Enterprise Admin** अधिकार प्राप्त कर लेते हैं, तो आप **domain database**: _ntds.dit_ को **dump** कर सकते हैं।

[**DCSync attack के बारे में अधिक जानकारी यहां मिल सकती है**](dcsync.md).

[**NTDS.dit कैसे चुराएं इसके बारे में अधिक जानकारी यहां मिल सकती है**](broken-reference/)

### Privesc as Persistence

पहले चर्चा की गई कुछ तकनीकों का उपयोग persistence के लिए किया जा सकता है।\
उदाहरण के लिए आप कर सकते हैं:

*   [**Kerberoast**](kerberoast.md) के लिए उपयोगकर्ताओं को संवेदनशील बनाएं

```powershell
Set-DomainObject -Identity <username> -Set @{serviceprincipalname="fake/NOTHING"}r
```
*   [**ASREPRoast**](asreproast.md) के लिए उपयोगकर्ताओं को संवेदनशील बनाएं

```powershell
Set-DomainObject -Identity <username> -XOR @{UserAccountControl=4194304}
```
*   एक उपयोगकर्ता को [**DCSync**](./#dcsync) अधिकार दें

```powershell
Add-DomainObjectAcl -TargetIdentity "DC=SUB,DC=DOMAIN,DC=LOCAL" -PrincipalIdentity bfarmer -Rights DCSync
```

### Silver Ticket

Silver ticket attack **एक वैध TGS को तैयार करने पर आधारित है एक बार जब सेवा का NTLM hash अपने पास हो** (जैसे कि **PC account hash**). इस प्रकार, यह संभव है कि **उस सेवा तक पहुंच प्राप्त करें** एक कस्टम TGS **किसी भी उपयोगकर्ता के रूप में** बनाकर (जैसे कि कंप्यूटर पर विशेषाधिकार प्राप्त करना)।

{% content-ref url="silver-ticket.md" %}
[silver-ticket.md](silver-ticket.md)
{% endcontent-ref %}

### Golden Ticket

एक वैध **TGT किसी भी उपयोगकर्ता के रूप में** बनाया जा सकता है **krbtgt AD account के NTLM hash का उपयोग करके**। TGS के बजाय TGT बनाने का लाभ यह है कि आप **किसी भी सेवा तक पहुंच सकते हैं** (या मशीन) डोमेन में अनुकरण किए गए उपयोगकर्ता के रूप में।

{% content-ref url="golden-ticket.md" %}
[golden-ticket.md](golden-ticket.md)
{% endcontent-ref %}

### Diamond Ticket

ये golden tickets की तरह होते हैं जो एक तरीके से बनाए जाते हैं जो **common golden tickets detection mechanisms को बायपास करते हैं।**

{% content-ref
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
यहाँ **2 विश्वसनीय कुंजियाँ** हैं, एक _Child --> Parent_ के लिए और दूसरी _Parent_ --> _Child_ के लिए।\
आप वर्तमान डोमेन द्वारा प्रयुक्त कुंजी को इसके साथ देख सकते हैं:
```bash
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\mcorp$"'
```
#### SID-History Injection

एंटरप्राइज एडमिन के रूप में एस्कलेट करें और चाइल्ड/पेरेंट डोमेन में ट्रस्ट का दुरुपयोग करके SID-History इंजेक्शन के साथ:

{% content-ref url="sid-history-injection.md" %}
[sid-history-injection.md](sid-history-injection.md)
{% endcontent-ref %}

#### लिखने योग्य Configuration NC का शोषण

Configuration NC एक फॉरेस्ट के लिए कॉन्फ़िगरेशन जानकारी का प्राथमिक भंडार है और इसे फॉरेस्ट के हर DC में रेप्लिकेट किया जाता है। इसके अलावा, फॉरेस्ट के हर लिखने योग्य DC (रीड-ओनली DCs नहीं) में Configuration NC की एक लिखने योग्य प्रति होती है। इसका शोषण करने के लिए (चाइल्ड) DC पर SYSTEM के रूप में चलाना आवश्यक है।

रूट डोमेन को विभिन्न तरीकों से समझौता करना संभव है जो नीचे दिए गए हैं।

**रूट DC साइट पर GPO लिंक करें**

Configuration NC में Sites कंटेनर में AD फॉरेस्ट में डोमेन-जुड़े कंप्यूटरों की सभी साइट्स होती हैं। जब फॉरेस्ट के किसी भी DC पर SYSTEM के रूप में चल रहा हो, तो साइट्स पर GPOs लिंक करना संभव है, जिसमें फॉरेस्ट रूट DCs की साइट्स भी शामिल हैं, और इस प्रकार इन्हें समझौता कर सकते हैं।

अधिक जानकारी यहाँ पढ़ी जा सकती है [Bypass SID filtering research](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-4-bypass-sid-filtering-research)।

**फॉरेस्ट में किसी भी gMSA को समझौता करें**

यह हमला लक्षित डोमेन में विशेषाधिकार प्राप्त gMSAs पर निर्भर करता है।

KDS Root key, जिसका उपयोग फॉरेस्ट में gMSAs के पासवर्ड की गणना के लिए किया जाता है, Configuration NC में संग्रहीत होता है। जब फॉरेस्ट के किसी भी DC पर SYSTEM के रूप में चल रहा हो, तो कोई KDS Root key को पढ़ सकता है और फॉरेस्ट में किसी भी gMSA का पासवर्ड की गणना कर सकता है।

अधिक जानकारी यहाँ पढ़ी जा सकती है: [Golden gMSA trust attack from child to parent](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-5-golden-gmsa-trust-attack-from-child-to-parent)।

**Schema change attack**

इस हमले के लिए हमलावर को नए विशेषाधिकार प्राप्त AD ऑब्जेक्ट्स के बनाए जाने का इंतजार करना पड़ता है।

जब फॉरेस्ट के किसी भी DC पर SYSTEM के रूप में चल रहा हो, तो कोई भी यूजर को AD Schema में सभी क्लासेस पर पूर्ण नियंत्रण प्रदान कर सकता है। इस नियंत्रण का दुरुपयोग किसी भी AD ऑब्जेक्ट के डिफ़ॉल्ट सिक्योरिटी डिस्क्रिप्टर में एक ACE बनाने के लिए किया जा सकता है जो समझौता किए गए प्रिंसिपल को पूर्ण नियंत्रण प्रदान करता है। संशोधित AD ऑब्जेक्ट प्रकारों के सभी नए उदाहरणों में यह ACE होगा।

अधिक जानकारी यहाँ पढ़ी जा सकती है: [Schema change trust attack from child to parent](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-6-schema-change-trust-attack-from-child-to-parent)।

**ADCS ESC5 के साथ DA से EA तक**

ADCS ESC5 (Vulnerable PKI Object Access Control) हमले PKI ऑब्जेक्ट्स पर नियंत्रण का दुरुपयोग करते हैं ताकि एक भेद्य प्रमाणपत्र टेम्पलेट बनाया जा सके जिसका उपयोग फॉरेस्ट में किसी भी यूजर के रूप में प्रमाणित करने के लिए किया जा सकता है। चूंकि सभी PKI ऑब्जेक्ट्स Configuration NC में संग्रहीत होते हैं, इसलिए यदि किसी ने फॉरेस्ट में किसी भी लिखने योग्य (चाइल्ड) DC को समझौता किया है, तो वे ESC5 को निष्पादित कर सकते हैं।

अधिक जानकारी यहाँ पढ़ी जा सकती है: [From DA to EA with ESC5](https://posts.specterops.io/from-da-to-ea-with-esc5-f9f045aa105c)

यदि AD फॉरेस्ट में ADCS नहीं है, तो हमलावर यहाँ वर्णित के अनुसार आवश्यक घटक बना सकता है: [Escalating from child domain’s admins to enterprise admins in 5 minutes by abusing AD CS, a follow up](https://www.pkisolutions.com/escalating-from-child-domains-admins-to-enterprise-admins-in-5-minutes-by-abusing-ad-cs-a-follow-up/).

### बाहरी फॉरेस्ट डोमेन - एक-तरफा (इनबाउंड) या द्विदिश (bidirectional)
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
इस परिदृश्य में **आपके डोमेन पर विश्वास किया गया है** एक बाहरी डोमेन द्वारा जो आपको **अनिर्धारित अनुमतियाँ** प्रदान करता है। आपको यह पता लगाना होगा कि **आपके डोमेन के कौन से सिद्धांतकार बाहरी डोमेन पर किस प्रकार की पहुँच रखते हैं** और फिर इसका दोहन करने की कोशिश करें:

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
```markdown
इस परिदृश्य में **आपका डोमेन** कुछ **विशेषाधिकार** **अलग डोमेन** से आए प्रिंसिपल को **विश्वास** कर रहा है।

हालांकि, जब एक **डोमेन विश्वसनीय होता है** तो विश्वसनीय डोमेन एक **उपयोगकर्ता बनाता है** जिसका नाम **अनुमानित होता है** और जो **पासवर्ड के रूप में विश्वसनीय पासवर्ड** का उपयोग करता है। इसका मतलब है कि विश्वसनीय डोमेन के अंदर जाने के लिए विश्वास करने वाले डोमेन के एक उपयोगकर्ता को **पहुंच संभव है** ताकि उसे गिना जा सके और अधिक विशेषाधिकारों को बढ़ाने की कोशिश की जा सके:

{% content-ref url="external-forest-domain-one-way-outbound.md" %}
[external-forest-domain-one-way-outbound.md](external-forest-domain-one-way-outbound.md)
{% endcontent-ref %}

विश्वसनीय डोमेन को समझौता करने का एक और तरीका है [**SQL विश्वसनीय लिंक**](abusing-ad-mssql.md#mssql-trusted-links) को ढूंढना जो डोमेन विश्वास की **विपरीत दिशा** में बनाया गया है (जो बहुत आम नहीं है)।

विश्वसनीय डोमेन को समझौता करने का एक और तरीका है उस मशीन में प्रतीक्षा करना जहां विश्वसनीय डोमेन का **उपयोगकर्ता पहुंच सकता है** **RDP** के माध्यम से लॉगिन करने के लिए। फिर, हमलावर RDP सत्र प्रक्रिया में कोड इंजेक्ट कर सकता है और **पीड़ित के मूल डोमेन तक पहुंच सकता है**।\
इसके अलावा, अगर **पीड़ित ने अपनी हार्ड ड्राइव माउंट की है**, तो **RDP सत्र** प्रक्रिया से हमलावर हार्ड ड्राइव के **स्टार्टअप फोल्डर में बैकडोर्स** स्टोर कर सकता है। इस तकनीक को **RDPInception** कहा जाता है।

{% content-ref url="rdp-sessions-abuse.md" %}
[rdp-sessions-abuse.md](rdp-sessions-abuse.md)
{% endcontent-ref %}

### डोमेन विश्वास के दुरुपयोग की रोकथाम

**SID फिल्टरिंग:**

* वन विश्वास के पार SID इतिहास विशेषता के दुरुपयोग से बचें।
* सभी अंतर-वन विश्वासों पर डिफ़ॉल्ट रूप से सक्षम। इंट्रा-फ़ॉरेस्ट विश्वासों को डिफ़ॉल्ट रूप से सुरक्षित माना जाता है (MS वन को डोमेन के बजाय सुरक्षा सीमा मानता है)।
* लेकिन, चूंकि SID फिल्टरिंग से एप्लिकेशन और उपयोगकर्ता पहुंच को टूटने की संभावना होती है, इसे अक्सर अक्षम किया जाता है।
* चयनात्मक प्रमाणीकरण
* एक अंतर-वन विश्वास में, यदि चयनात्मक प्रमाणीकरण कॉन्फ़िगर किया गया है, तो विश्वासों के बीच उपयोगकर्ताओं को स्वचालित रूप से प्रमाणित नहीं किया जाएगा। विश्वास करने वाले डोमेन/वन में डोमेन और सर्वरों तक व्यक्तिगत पहुंच दी जानी चाहिए।
* लिखने योग्य Configration NC शोषण और विश्वास खाता हमले को रोकता नहीं है।

[**डोमेन विश्वासों के बारे में अधिक जानकारी ired.team में।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain)

## AD -> क्लाउड & क्लाउड -> AD

{% embed url="https://cloud.hacktricks.xyz/pentesting-cloud/azure-security/az-lateral-movements/azure-ad-connect-hybrid-identity" %}

## कुछ सामान्य रक्षा

[**यहां जानें कि क्रेडेंशियल्स की सुरक्षा कैसे करें।**](../stealing-credentials/credentials-protections.md)\
**कृपया, प्रत्येक तकनीक के विवरण में कुछ माइग्रेशन खोजें।**

* डोमेन एडमिन्स को डोमेन कंट्रोलर्स के अलावा किसी अन्य होस्ट पर लॉगिन करने की अनुमति न दें
* कभी भी DA विशेषाधिकारों के साथ सेवा न चलाएं
* यदि आपको डोमेन एडमिन विशेषाधिकारों की आवश्यकता है, तो समय सीमित करें: `Add-ADGroupMember -Identity ‘Domain Admins’ -Members newDA -MemberTimeToLive (New-TimeSpan -Minutes 20)`

### छल

* पासवर्ड कभी समाप्त नहीं होता
* प्रतिनिधित्व के लिए विश्वसनीय
* SPN के साथ उपयोगकर्ता
* विवरण में पासवर्ड
* उच्च विशेषाधिकार समूहों के सदस्य उपयोगकर्ता
* अन्य उपयोगकर्ताओं, समूहों या कंटेनरों पर ACL अधिकारों वाले उपयोगकर्ता
* कंप्यूटर ऑब्जेक्ट्स
* ...
* [https://github.com/samratashok/Deploy-Deception](https://github.com/samratashok/Deploy-Deception)
* `Create-DecoyUser -UserFirstName user -UserLastName manager-uncommon -Password Pass@123 | DeployUserDeception -UserFlag PasswordNeverExpires -GUID d07da11f-8a3d-42b6-b0aa-76c962be719a -Verbose`

## छल की पहचान कैसे करें

**उपयोगकर्ता ऑब्जेक्ट्स के लिए:**

* ObjectSID (डोमेन से अलग)
* lastLogon, lastlogontimestamp
* Logoncount (बहुत कम संख्या संदिग्ध है)
* whenCreated
* Badpwdcount (बहुत कम संख्या संदिग्ध है)

**सामान्य:**

* कुछ समाधान सभी संभावित विशेषताओं में जानकारी भरते हैं। उदाहरण के लिए, DC जैसे 100% वास्तविक कंप्यूटर ऑब्जेक्ट की विशेषताओं की तुलना करें। या RID 500 (डिफ़ॉल्ट एडमिन) के खिलाफ उपयोगकर्ताओं की।
* जांचें कि कुछ बहुत अच्छा है तो सच होने के लिए
* [https://github.com/JavelinNetworks/HoneypotBuster](https://github.com/JavelinNetworks/HoneypotBuster)

### Microsoft ATA पता लगाने को बायपास करना

#### उपयोगकर्ता सूचीकरण

ATA केवल तब शिकायत करता है जब आप DC में सत्रों को सूचीबद्ध करने की कोशिश करते हैं, इसलिए यदि आप DC में सत्रों की तलाश नहीं करते हैं लेकिन बाकी होस्टों में, तो आपको शायद पता नहीं चलेगा।

#### टिकट अनुकरण निर्माण (Over pass the hash, golden ticket...)

हमेशा टिकटों को **aes** कुंजियों का उपयोग करके बनाएं क्योंकि ATA जो मालिकाना है वह NTLM का अवमूल्यन है।

#### DCSync

यदि आप इसे डोमेन कंट्रोलर से नहीं चलाते हैं, तो ATA आपको पकड़ लेगा, क्षमा करें।

## और उपकरण

* [Powershell स्क्रिप्ट डोमेन ऑडिटिंग ऑटोमेशन के लिए](https://github.com/phillips321/adaudit)
* [Python स्क्रिप्ट एक्टिव डायरेक्टरी को सूचीबद्ध करने के लिए](https://github.com/ropnop/windapsearch)
* [Python स्क्रिप्ट एक्टिव डायरेक्टरी को सूचीबद्ध करने के लिए](https://github.com/CroweCybersecurity/ad-ldap-enum)

## संदर्भ

* [http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/](http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकार
