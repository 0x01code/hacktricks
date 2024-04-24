# macOS संवेदनशील स्थान और दिलचस्प डेमन्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks और HackTricks Cloud** github repos में PRs सबमिट करके।

</details>

## पासवर्ड

### शैडो पासवर्ड

शैडो पासवर्ड उपयोगकर्ता के कॉन्फ़िगरेशन के साथ संग्रहीत होता है पीलिस्ट में जो स्थित है **`/var/db/dslocal/nodes/Default/users/`** में।\
निम्नलिखित वनलाइनर का उपयोग किया जा सकता है **सभी उपयोगकर्ताओं के बारे में सभी जानकारी को डंप करने** के लिए (हैश जानकारी सहित):

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
{% endcode %}

[**इस तरह के स्क्रिप्ट**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2) या [**इस तरह के**](https://github.com/octomagon/davegrohl.git) का उपयोग **हैश** को **हैशकैट प्रारूप** में बदलने के लिए किया जा सकता है।

एक वैकल्पिक वन-लाइनर जो सभी गैर-सेवा खातों के क्रेडेंशियल को हैशकैट प्रारूप में डंप करेगा `-m 7100` (macOS PBKDF2-SHA512):

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
{% endcode %}

### कीचेन डंप

ध्यान दें कि सुरक्षा बाइनरी का उपयोग करते समय **पासवर्ड डिक्रिप्ट करने के लिए डंप** करने पर कई प्रॉम्प्ट उपयोगकर्ता से इस ऑपरेशन की अनुमति देने के लिए पूछेंगे।
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
इस टिप्पणी के आधार पर [juuso/keychaindump#10 (comment)](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) यह लगता है कि ये उपकरण Big Sur में अब काम नहीं कर रहे हैं।
{% endhint %}

### Keychaindump Overview

एक उपकरण जिसका नाम **keychaindump** है, इसका विकास macOS की keychains से पासवर्ड निकालने के लिए किया गया है, लेकिन नए macOS संस्करणों जैसे Big Sur पर इसकी सीमाएँ हैं, जैसा कि [चर्चा](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) में इंगित किया गया है। **keychaindump** का उपयोग अटैकर को **root** तक पहुंचने और वर्चस्व को उन्नत करने की आवश्यकता है। यह उपकरण उस तथ्य का शोषण करता है कि keychain डिफ़ॉल्ट रूप से उपयोक्ता लॉगिन के बाद अनलॉक होता है ताकि अनुकूलता के लिए अनुप्रयोग उसे बार-बार उपयोगकर्ता का पासवर्ड नहीं मांगें। हालांकि, यदि एक उपयोक्ता हर उपयोग के बाद अपनी keychain को लॉक करने का विकल्प चुनता है, तो **keychaindump** अप्रभावी हो जाता है।

**Keychaindump** एक विशिष्ट प्रक्रिया को लक्ष्य बनाकर काम करता है जिसे **securityd** कहा जाता है, जिसे Apple ने प्रमाणीकरण और यांत्रिक आपरेशनों के लिए डेमन के रूप में वर्णित किया है, जो keychain तक पहुंचने के लिए महत्वपूर्ण है। निकालने की प्रक्रिया में उपयोगकर्ता के लॉगिन पासवर्ड से निकली गई एक **मास्टर की** की पहचान करना महत्वपूर्ण है। यह कुंजी keychain फ़ाइल को पढ़ने के लिए आवश्यक है। **keychaindump** **securityd** की मेमोरी हीप को स्कैन करके **मास्टर की** को ढूंढता है, `vmmap` कमांड का उपयोग करके, जहां संभावित कुंजीयों को `MALLOC_TINY` के रूप में झंझावाती क्षेत्रों में पहचाना जाता है। इन मेमोरी स्थानों की जांच के लिए निम्नलिखित कमांड का उपयोग किया जाता है:
```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```
पॉटेंशियल मास्टर की को पहचानने के बाद, **keychaindump** विशिष्ट पैटर्न (`0x0000000000000018`) की खोज करता है जो मास्टर की के लिए उम्मीदवार को दर्शाता है। इस की का उपयोग करने के लिए और कई कदम चाहिए होते हैं, जैसा कि **keychaindump** के स्रोत कोड में विस्तार से वर्णित है। इस क्षेत्र पर ध्यान केंद्रित विश्लेषकों को ध्यान देना चाहिए कि कीचेन को डिक्रिप्ट करने के लिए महत्वपूर्ण डेटा **securityd** प्रक्रिया की मेमोरी में संग्रहित है। **keychaindump** चलाने के लिए एक उदाहरण कमांड है:
```bash
sudo ./keychaindump
```
### चेनब्रेकर

[**चेनब्रेकर**](https://github.com/n0fate/chainbreaker) का उपयोग एक फोरेंसिक तरीके से OSX कीचेन से निम्नलिखित प्रकार की जानकारी निकालने के लिए किया जा सकता है:

* हैश वाला Keychain पासवर्ड, [hashcat](https://hashcat.net/hashcat/) या [John the Ripper](https://www.openwall.com/john/) के साथ क्रैकिंग के लिए उपयुक्त
* इंटरनेट पासवर्ड
* जेनेरिक पासवर्ड
* निजी कुंजी
* सार्वजनिक कुंजी
* X509 प्रमाणपत्र
* सुरक्षित नोट्स
* एप्पलशेयर पासवर्ड

कीचेन अनलॉक पासवर्ड, [volafox](https://github.com/n0fate/volafox) या [volatility](https://github.com/volatilityfoundation/volatility) का उपयोग करके प्राप्त मास्टर कुंजी, या सिस्टमकी जैसा अनलॉक फ़ाइल के साथ, चेनब्रेकर भी सादा पाठ पासवर्ड प्रदान करेगा।

बिना इन तरीकों में से किसी एक का उपयोग करके Keychain को अनलॉक करने के, चेनब्रेकर सभी अन्य उपलब्ध जानकारी प्रदर्शित करेगा।

#### **Keychain कुंजियों को डंप करें**
```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```
#### **सिस्टमकी के साथ कीचेन कुंजी (पासवर्ड के साथ) डंप करें**
```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **हैश को क्रैक करके कीचेन कुंजी (साथ में पासवर्ड) डंप करें**
```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **मेमोरी डंप के साथ कीचेन कुंजी (पासवर्ड के साथ) डंप करें**

[इन चरणों का पालन करें](../#dumping-memory-with-osxpmem) एक **मेमोरी डंप** करने के लिए
```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **उपयोगकर्ता के पासवर्ड का उपयोग करके कीचेन कुंजी (साथ में पासवर्ड) डंप करें**

यदि आप उपयोगकर्ता का पासवर्ड जानते हैं तो आप इसका उपयोग करके **उपयोगकर्ता की कीचेन कुंजियों को डंप और डिक्रिप्ट** कर सकते हैं।
```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```
### kcpassword

**kcpassword** फ़ाइल एक फ़ाइल है जो **उपयोगकर्ता का लॉगिन पासवर्ड** रखती है, लेकिन केवल तभी जब सिस्टम के मालिक ने **स्वचालित लॉगिन** को सक्षम किया है। इसलिए, उपयोगकर्ता को पासवर्ड के बिना स्वचालित रूप से लॉग इन किया जाएगा (जो बहुत ही सुरक्षित नहीं है)।

पासवर्ड फ़ाइल में **`/etc/kcpassword`** रखा जाता है जो कि कुंजी **`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`** के साथ xored होता है। अगर उपयोगकर्ता का पासवर्ड कुंजी से अधिक लंबा है, तो कुंजी का पुनः उपयोग किया जाएगा।\
इससे पासवर्ड को पुनः प्राप्त करना काफी आसान हो जाता है, उदाहरण के लिए [**इस तरह के**](https://gist.github.com/opshope/32f65875d45215c3677d) स्क्रिप्ट का उपयोग करके।

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

अधिकांश दिलचस्प जानकारी **ब्लॉब** में होगी। इसलिए आपको उस सामग्री को **निकालना** होगा और **मानव** **पठनीय** बनाना होगा या **`strings`** का उपयोग करना होगा। इस तक पहुंचने के लिए आप यह कर सकते हैं:
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
{% endcode %}

### नोट्स

उपयोगकर्ताओं के **नोट्स** यहाँ मिल सकते हैं `~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite`

{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```
{% endcode %}

## Preferences

macOS ऐप्स में preferences **`$HOME/Library/Preferences`** में स्थित होते हैं और iOS में वे `/var/mobile/Containers/Data/Application/<UUID>/Library/Preferences` में होते हैं।

macOS में cli टूल **`defaults`** का उपयोग **Preferences फ़ाइल को संशोधित करने** के लिए किया जा सकता है।

**`/usr/sbin/cfprefsd`** XPC सेवाएं `com.apple.cfprefsd.daemon` और `com.apple.cfprefsd.agent` का दावा करता है और preferences को संशोधित करने जैसे कार्रवाई करने के लिए कॉल किया जा सकता है।

## System Notifications

### Darwin Notifications

Notifications के लिए मुख्य डेमन **`/usr/sbin/notifyd`** है। Notifications प्राप्त करने के लिए, clients को `com.apple.system.notification_center` Mach port के माध्यम से पंजीकृत करना होता है (उन्हें `sudo lsmp -p <pid notifyd>` के साथ जांचें)। डेमन को फ़ाइल `/etc/notify.conf` के साथ समाकृत किया जा सकता है।

Notifications के लिए उपयोग किए जाने वाले नाम अद्वितीय प्रतिलिपि DNS नोटेशन होते हैं और जब उनमें से किसी एक को भेजा जाता है, तो उन client(s) को जो इसे संभाल सकते हैं, वह इसे प्राप्त करेंगे।

वर्तमान स्थिति को डंप करना संभव है (और सभी नाम देखना) notifyd प्रक्रिया को सिग्नल SIGUSR2 भेजकर और उत्पन्न फ़ाइल पढ़कर: `/var/run/notifyd_<pid>.status`:
```bash
ps -ef | grep -i notifyd
0   376     1   0 15Mar24 ??        27:40.97 /usr/sbin/notifyd

sudo kill -USR2 376

cat /var/run/notifyd_376.status
[...]
pid: 94379   memory 5   plain 0   port 0   file 0   signal 0   event 0   common 10
memory: com.apple.system.timezone
common: com.apple.analyticsd.running
common: com.apple.CFPreferences._domainsChangedExternally
common: com.apple.security.octagon.joined-with-bottle
[...]
```
### वितरित सूचना केंद्र

**वितरित सूचना केंद्र** जिसका मुख्य बाइनरी है **`/usr/sbin/distnoted`**, एक और तरीका है सूचनाएं भेजने का। इसमें कुछ XPC सेवाएं उजागर की गई हैं और यह कुछ जांच कार्रवाई करता है ताकि ग्राहकों की पुष्टि की जा सके।

### एप्पल पुश सूचनाएं (APN)

इस मामले में, एप्लिकेशन **विषय** के लिए पंजीकरण कर सकती हैं। ग्राहक एप्पल के सर्वरों से संपर्क करके **`apsd`** के माध्यम से एक टोकन उत्पन्न करेगा।\
फिर, प्रदाताओं ने भी एक टोकन उत्पन्न किया होगा और वे एप्पल के सर्वरों से संपर्क स्थापित कर सकेंगे ताकि संदेश ग्राहकों को भेज सकें। ये संदेश स्थानीय रूप से **`apsd`** द्वारा प्राप्त किए जाएंगे जो उस एप्लिकेशन को सूचना पहुंचाएगा जो इसका इंतजार कर रहा होगा।

पसंद स्थान `/Library/Preferences/com.apple.apsd.plist` में स्थित है।

मैकओएस में संदेशों का स्थानीय डेटाबेस है `/Library/Application\ Support/ApplePushService/aps.db` और आईओएस में `/var/mobile/Library/ApplePushService` में है। इसमें 3 तालिकाएं हैं: `incoming_messages`, `outgoing_messages` और `channel`।
```bash
sudo sqlite3 /Library/Application\ Support/ApplePushService/aps.db
```
डेमन और कनेक्शन के बारे में जानकारी प्राप्त करना भी संभव है:
```bash
/System/Library/PrivateFrameworks/ApplePushService.framework/apsctl status
```
## उपयोगकर्ता सूचनाएँ

ये सूचनाएँ हैं जो उपयोगकर्ता को स्क्रीन पर दिखानी चाहिए:

* **`CFUserNotification`**: यह API एक संदेश के साथ स्क्रीन पर एक पॉप-अप दिखाने का एक तरीका प्रदान करती है।
* **बुलेटिन बोर्ड**: यह iOS में एक बैनर दिखाता है जो गायब हो जाता है और सूचना केंद्र में संग्रहीत किया जाएगा।
* **`NSUserNotificationCenter`**: यह MacOS में iOS बुलेटिन बोर्ड है। सूचनाएँ के डेटाबेस `/var/folders/<user temp>/0/com.apple.notificationcenter/db2/db` में स्थित है।
