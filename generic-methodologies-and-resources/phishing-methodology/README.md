# फिशिंग मेथडोलॉजी

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मेथडोलॉजी

1. पीड़ित का जासूसी करें
1. **पीड़ित डोमेन** का चयन करें।
2. पीड़ित द्वारा उपयोग किए जाने वाले **लॉगिन पोर्टल्स की खोज** के लिए कुछ मौलिक वेब जांच करें और **निर्णय** लें कि आप किसे **अनुकरण** करेंगे।
3. कुछ **OSINT** का उपयोग करके **ईमेल खोजें**।
2. पर्यावरण तैयार करें
1. फिशिंग मूल्यांकन के लिए जिस डोमेन का उपयोग करने जा रहे हैं, उसे **खरीदें**
2. ईमेल सेवा संबंधित रिकॉर्ड (SPF, DMARC, DKIM, rDNS) कॉन्फ़िगर करें
3. **gophish** के साथ VPS कॉन्फ़िगर करें
3. अभियान तैयार करें
1. **ईमेल टेम्पलेट** तैयार करें
2. प्रमाणपत्र चोरी करने के लिए **वेब पेज** तैयार करें
4. अभियान शुरू करें!

## समान डोमेन नाम उत्पन्न करें या विश्वसनीय डोमेन खरीदें

### डोमेन नाम वेरिएशन तकनीक

* **कीवर्ड**: मूल डोमेन का महत्वपूर्ण **कीवर्ड शामिल है** (जैसे, zelster.com-management.com)।
* **हाइफेन सबडोमेन**: एक सबडोमेन के लिए **डॉट को हाइफेन में बदलें** (जैसे, www-zelster.com)।
* **नया TLD**: एक **नया TLD** का उपयोग करके समान डोमेन (जैसे, zelster.org)
* **होमोग्लिफ**: डोमेन नाम में एक अक्षर को **उसके समान दिखने वाले अक्षरों से बदलता है** (जैसे, zelfser.com)।
* **ट्रांसपोज़िशन:** डोमेन नाम में दो अक्षरों को **आपस में बदल देता है** (जैसे, zelster.com)।
* **एकजीवनीकरण/बहुवचनीकरण**: डोमेन नाम के अंत में "s" जोड़ता या हटाता है (जैसे, zeltsers.com)।
* **अभाव**: डोमेन नाम से एक अक्षर को **हटा देता है** (जैसे, zelser.com)।
* **पुनरावृत्ति**: डोमेन नाम में एक अक्षर को **दोहराता है** (जैसे, zeltsser.com)।
* **प्रतिस्थापन**: होमोग्लिफ की तरह लेकिन कम गुप्तचर। डोमेन नाम में एक अक्षर को बदलता है, शायद मूल अक्षर के पास कुंजीपटल पर मौजूद अक्षर के साथ (जैसे, zektser.com)।
* **सबडोमेन्ड**: डोमेन नाम में एक **डॉट** डालता है (जैसे, ze.lster.com)।
* **इन्सर्शन**: डोमेन नाम में एक अक्षर **डालता है** (जैसे, zerltser.com)।
* **गायब डॉट**: डोमेन नाम के अंत में TLD जोड़ता है। (जैसे, zelstercom.com)

**स्वचालित उपकरण**

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy)

**वेबसाइटें**

* [https://dnstwist.it/](https://dnstwist.it)
* [https://dnstwister.report/](https://dnstwister.report)
* [https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/](https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/)

### बिटफ्लिपिंग

कंप्यूटिंग की दुनिया में, सभी कुछ यादृच्छिक रूप से बिट्स (शून्य और एक) में संग्रहित होता है, आपके कंप्यूटिंग उपकरण की अस्थायी स्मृति में।\
यह डोमेनों के लिए भी लागू होता है। उदाहरण के लिए, _windows.com_ आपके कंप्यूटिंग उपकरण की अस्थायी स्मृति में _01110111..._ बन जाता है।\
हालांकि, यदि इनमें से कोई एक बिट सौर फ्लेयर, कॉस्मिक रे, या हार्डवेयर त्रुटि के कारण स्वचालित रूप से फ्लिप हो जाए, तो क्या होगा? यानी 0 में से एक 1 बन जाता है और उल्टा।\
DNS अनुरोध को इस संकल्प के लागू करने पर, संभावना है कि **DNS सर्वर तक पहुंचने वाला डोमेन** **वास्तविक रूप से पहले अनुरोधित डोमेन से अलग हो सकता है**।

उदाहरण के लिए, डोमेन windows.com में 1 बिट संशोधन _windnws.com._ में बदल सकता है।\
**हमलावर विकल्पित उपयोगकर्ताओं को अपने बुनियादी संरचना पर पुनर्निर्देशित करने के लिए पीड़ित से संबंधित जितने भी बिट-फ्लिपिंग डोमेन हो सकते हैं, उन्हें पंजीकृत कर सकते हैं**।

अधिक जानकारी के लिए पढ़ें [https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/](https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/)

### विश्वसनीय डोमेन खरीदें

आप [https://www.expireddomains.net/](https://www.expireddomains.net) में एक समाप्त होने वाले डोमेन की खोज कर सकते हैं जिसका आप उपयोग कर सकते हैं।\
यह सुनिश्चित करने के लिए कि आप जिस समाप्त होने वाले डोमेन को खरीदने जा रहे हैं, **उसमें पहले से अच्छा SEO है**, आप यह देख सकते हैं कि यह कैसे वर्गीकृत है:

* [http://www.fortiguard.com/webfilter](http://www.fortiguard.com/webfilter)
* [https://urlfiltering.paloaltonetworks.com/query/](https://urlfiltering.paloaltonetworks.com/query/)

## ईमेल खोजना

* [https://github.com/laramies/theHarvester](https://github.com/laramies/theHarvester) (100% मुफ्त)
* [https://phonebook.cz/](https://phonebook.cz) (100% मुफ्त)
* [https://maildb.io/](https://maildb.io)
* [https://hunter.io/](https://hunter.io)
* [https://anymailfinder.com/](https://anymailfinder.com)

अधिक मान्य ईमेल पते खोजने या जिन्हें आप पहले से ही खोज चुके हैं, उन्हें सत्यापित करने के लिए आप जांच सकते हैं कि क्या आप पीड़ित के smtp सर्वर को ब्रूट-फोर्स कर सकते हैं। [यहां सीखें कि ईमेल पता सत्यापित/खोजें कैसे करें](../../network-services-pentesting/pentesting-smtp/#username-bruteforce-enumeration)।\
इसके अतिरिक्त, यदि उपयोगकर्ता **अपने मेल तक पहुंचने के लिए किसी वेब पोर्टल का उपयोग करते हैं**, तो आप यह जांच सकते हैं कि क्या यह **उपयोगकर्ता नाम ब्रूट फो
```bash
ssh -L 3333:127.0.0.1:3333 <user>@<ip>
```
### विन्यास

**TLS प्रमाणपत्र विन्यास**

इस कदम से पहले आपको **पहले ही उस डोमेन को खरीद लिया होना चाहिए** जिसका आप उपयोग करने जा रहे हैं और यह **वहाँ पहुंचना चाहिए** जहाँ आप **gophish** को विन्यास कर रहे हैं।
```bash
DOMAIN="<domain>"
wget https://dl.eff.org/certbot-auto
chmod +x certbot-auto
sudo apt install snapd
sudo snap install core
sudo snap refresh core
sudo apt-get remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
certbot certonly --standalone -d "$DOMAIN"
mkdir /opt/gophish/ssl_keys
cp "/etc/letsencrypt/live/$DOMAIN/privkey.pem" /opt/gophish/ssl_keys/key.pem
cp "/etc/letsencrypt/live/$DOMAIN/fullchain.pem" /opt/gophish/ssl_keys/key.crt​
```
**मेल कॉन्फ़िगरेशन**

इंस्टॉलेशन शुरू करें: `apt-get install postfix`

फिर निम्नलिखित फ़ाइलों में डोमेन जोड़ें:

* **/etc/postfix/virtual\_domains**
* **/etc/postfix/transport**
* **/etc/postfix/virtual\_regexp**

**/etc/postfix/main.cf** के अंदर निम्नलिखित चरों के मान भी बदलें

`myhostname = <domain>`\
`mydestination = $myhostname, <domain>, localhost.com, localhost`

अंत में, **`/etc/hostname`** और **`/etc/mailname`** फ़ाइलों को अपने डोमेन नाम पर संशोधित करें और **अपने VPS को पुनः आरंभ करें।**

अब, `mail.<domain>` के लिए **DNS A रिकॉर्ड** बनाएं जो VPS के **IP पते** पर पॉइंट करता है और `mail.<domain>` पर पॉइंट करने वाला एक **DNS MX** रिकॉर्ड बनाएं।

अब हम एक ईमेल भेजने का परीक्षण करें:
```bash
apt install mailutils
echo "This is the body of the email" | mail -s "This is the subject line" test@email.com
```
**Gophish कॉन्फ़िगरेशन**

Gophish के निष्पादन को रोकें और इसे कॉन्फ़िगर करें।\
`/opt/gophish/config.json` को निम्नलिखित में संशोधित करें (https का उपयोग करें):
```bash
{
"admin_server": {
"listen_url": "127.0.0.1:3333",
"use_tls": true,
"cert_path": "gophish_admin.crt",
"key_path": "gophish_admin.key"
},
"phish_server": {
"listen_url": "0.0.0.0:443",
"use_tls": true,
"cert_path": "/opt/gophish/ssl_keys/key.crt",
"key_path": "/opt/gophish/ssl_keys/key.pem"
},
"db_name": "sqlite3",
"db_path": "gophish.db",
"migrations_prefix": "db/db_",
"contact_address": "",
"logging": {
"filename": "",
"level": ""
}
}
```
**गोफिश सेवा कॉन्फ़िगर करें**

गोफिश सेवा को स्वचालित रूप से शुरू किया जा सके और सेवा को प्रबंधित किया जा सके, आप निम्नलिखित सामग्री के साथ फ़ाइल `/etc/init.d/gophish` बना सकते हैं:
```bash
#!/bin/bash
# /etc/init.d/gophish
# initialization file for stop/start of gophish application server
#
# chkconfig: - 64 36
# description: stops/starts gophish application server
# processname:gophish
# config:/opt/gophish/config.json
# From https://github.com/gophish/gophish/issues/586

# define script variables

processName=Gophish
process=gophish
appDirectory=/opt/gophish
logfile=/var/log/gophish/gophish.log
errfile=/var/log/gophish/gophish.error

start() {
echo 'Starting '${processName}'...'
cd ${appDirectory}
nohup ./$process >>$logfile 2>>$errfile &
sleep 1
}

stop() {
echo 'Stopping '${processName}'...'
pid=$(/bin/pidof ${process})
kill ${pid}
sleep 1
}

status() {
pid=$(/bin/pidof ${process})
if [["$pid" != ""| "$pid" != "" ]]; then
echo ${processName}' is running...'
else
echo ${processName}' is not running...'
fi
}

case $1 in
start|stop|status) "$1" ;;
esac
```
सेवा को विन्यासित करने और इसे जांचने के लिए निम्न कार्रवाई करें:
```bash
mkdir /var/log/gophish
chmod +x /etc/init.d/gophish
update-rc.d gophish defaults
#Check the service
service gophish start
service gophish status
ss -l | grep "3333\|443"
service gophish stop
```
## कॉन्फ़िगरिंग मेल सर्वर और डोमेन

### इंतजार

जितना पुराना डोमेन होगा, उतना ही कम संभावना है कि यह स्पैम के रूप में पकड़ा जाएगा। इसलिए आपको फिशिंग मूल्यांकन से पहले जितना समय हो सके उसका इंतजार करना चाहिए (कम से कम 1 सप्ताह)।\
ध्यान दें कि यदि आपको एक सप्ताह इंतजार करना पड़ता है तो आप अब सभी चीजें कॉन्फ़िगर कर सकते हैं।

### रिवर्स DNS (rDNS) रिकॉर्ड कॉन्फ़िगर करें

एक rDNS (PTR) रिकॉर्ड सेट करें जो VPS का IP पता डोमेन नाम में विलिन करता है।

### भेजने वाली नीति फ्रेमवर्क (SPF) रिकॉर्ड

आपको **नए डोमेन के लिए एक SPF रिकॉर्ड कॉन्फ़िगर करना होगा**। अगर आपको नहीं पता कि SPF रिकॉर्ड क्या है तो [**इस पेज को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#spf)।

आप [https://www.spfwizard.net/](https://www.spfwizard.net) का उपयोग कर सकते हैं अपनी SPF नीति उत्पन्न करने के लिए (VPS मशीन का आईपी पता उपयोग करें)

![](<../../.gitbook/assets/image (388).png>)

यह वह सामग्री है जो डोमेन के भीतर एक TXT रिकॉर्ड में सेट की जानी चाहिए:
```bash
v=spf1 mx a ip4:ip.ip.ip.ip ?all
```
### डोमेन-आधारित संदेश प्रमाणीकरण, रिपोर्टिंग और अनुरूपण (DMARC) रिकॉर्ड

आपको **नए डोमेन के लिए एक DMARC रिकॉर्ड कॉन्फ़िगर करना होगा**। अगर आपको DMARC रिकॉर्ड क्या है पता नहीं है तो [**इस पेज को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#dmarc)।

आपको निम्नलिखित सामग्री के साथ नए DNS TXT रिकॉर्ड बनाना होगा जिसमें होस्टनाम `_dmarc.<डोमेन>` को पॉइंट करना होगा:
```bash
v=DMARC1; p=none
```
### डोमेन कुंजी निश्चित मेल (DKIM)

आपको **नए डोमेन के लिए एक DKIM कॉन्फ़िगर करना होगा**। अगर आपको DMARC रिकॉर्ड क्या है पता नहीं है तो [**इस पेज को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#dkim)।

यह ट्यूटोरियल इस पर आधारित है: [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)

{% hint style="info" %}
आपको DKIM कुंजी जेनरेट करते समय उत्पन्न दोनों B64 मानों को जोड़ने की आवश्यकता है:
```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0wPibdqPtzYk81njjQCrChIcHzxOp8a1wjbsoNtka2X9QXCZs+iXkvw++QsWDtdYu3q0Ofnr0Yd/TmG/Y2bBGoEgeE+YTUG2aEgw8Xx42NLJq2D1pB2lRQPW4IxefROnXu5HfKSm7dyzML1gZ1U0pR5X4IZCH0wOPhIq326QjxJZm79E1nTh3xj" "Y9N/Dt3+fVnIbMupzXE216TdFuifKM6Tl6O/axNsbswMS1TH812euno8xRpsdXJzFlB9q3VbMkVWig4P538mHolGzudEBg563vv66U8D7uuzGYxYT4WS8NVm3QBMg0QKPWZaKp+bADLkOSB9J2nUpk4Aj9KB5swIDAQAB
```
{% endhint %}

### अपने ईमेल कॉन्फ़िगरेशन स्कोर का परीक्षण करें

आप इसे [https://www.mail-tester.com/](https://www.mail-tester.com) का उपयोग करके कर सकते हैं।
बस पृष्ठ तक पहुंचें और उस पते पर एक ईमेल भेजें जो वे आपको देंगे:
```bash
echo "This is the body of the email" | mail -s "This is the subject line" test-iimosa79z@srv1.mail-tester.com
```
आप अपने ईमेल कॉन्फ़िगरेशन की जांच भी कर सकते हैं जब आप ईमेल भेजकर `check-auth@verifier.port25.com` पर भेजें और प्रतिक्रिया पढ़ें (इसके लिए आपको पोर्ट 25 खोलने की आवश्यकता है और यदि आप ईमेल को रूट के रूप में भेजते हैं तो फ़ाइल _/var/mail/root_ में प्रतिक्रिया देखें)।\
सुनिश्चित करें कि आप सभी परीक्षणों में सफलता प्राप्त कर रहे हैं:
```bash
==========================================================
Summary of Results
==========================================================
SPF check:          pass
DomainKeys check:   neutral
DKIM check:         pass
Sender-ID check:    pass
SpamAssassin check: ham
```
विकल्प स्वरूप में, आप एक **Gmail पते पर संदेश भेज सकते हैं जिसे आप नियंत्रण करते हैं**, अपने Gmail इनबॉक्स में प्राप्त **ईमेल के हेडर्स** को **देखें**, `Authentication-Results` हेडर फील्ड में `dkim=pass` मौजूद होना चाहिए।
```
Authentication-Results: mx.google.com;
spf=pass (google.com: domain of contact@example.com designates --- as permitted sender) smtp.mail=contact@example.com;
dkim=pass header.i=@example.com;
```
### हटाना स्पैमहाउस ब्लैकलिस्ट से

पृष्ठ www.mail-tester.com आपको बता सकता है कि क्या आपका डोमेन स्पैमहाउस द्वारा अवरुद्ध किया गया है। आप अपने डोमेन/आईपी को हटाने के लिए यहाँ अनुरोध कर सकते हैं: [https://www.spamhaus.org/lookup/](https://www.spamhaus.org/lookup/)

### माइक्रोसॉफ्ट ब्लैकलिस्ट से हटाना

आप अपने डोमेन/आईपी को हटाने के लिए यहाँ अनुरोध कर सकते हैं [https://sender.office.com/](https://sender.office.com).

## गोफिश अभियान बनाएं और लॉन्च करें

### भेजने का प्रोफ़ाइल

* किसी **नाम को पहचानने** के लिए सेंडर प्रोफ़ाइल सेट करें
* तय करें कि आप फिशिंग ईमेल भेजने के लिए किस खाते से भेजने जा रहे हैं। सुझाव: _noreply, support, servicedesk, salesforce..._
* आप उपयोगकर्ता नाम और पासवर्ड खाली छोड़ सकते हैं, लेकिन सुनिश्चित करें कि आपने अनग्रेजी प्रमाणपत्र त्रुटियों को जांचने का विकल्प चेक किया है

![](<../../.gitbook/assets/image (253) (1) (2) (1) (1) (2) (2) (3) (3) (5) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (17).png>)

{% hint style="info" %}
सुझाव दिया जाता है कि "**टेस्ट ईमेल भेजें**" कार्यक्षमता का उपयोग करें ताकि सब कुछ काम कर रहा है यह जांचने के लिए।\
मैं सुझाव दूंगा कि **टेस्ट ईमेल को 10 मिनट मेल पतों पर भेजें** ताकि परीक्षण करते समय ब्लैकलिस्ट होने से बचा जा सके।
{% endhint %}

### ईमेल टेम्पलेट

* कुछ **नाम सेट करें** टेम्पलेट को पहचानने के लिए
* फिर एक **विषय** लिखें (कुछ अजीब नहीं, बस कुछ जिसे आप एक साधारण ईमेल में पढ़ने की उम्मीद कर सकते हैं)
* सुनिश्चित करें कि आपने "**ट्रैकिंग छवि जोड़ें**" को चेक किया है
* ईमेल टेम्पलेट लिखें (आप चाहें तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे तो चाहे त
```markup
<html>
<head>
<title></title>
</head>
<body>
<p class="MsoNormal"><span style="font-size:10.0pt;font-family:&quot;Verdana&quot;,sans-serif;color:black">Dear {{.FirstName}} {{.LastName}},</span></p>

<p class="MsoNormal"><span style="font-size:10.0pt;font-family:&quot;Verdana&quot;,sans-serif;color:black">As you may be aware, due to the large number of employees working from home, the "PLATFORM NAME" platform is being migrated to a new domain with an improved and more secure version. To finalize account migration, please use the following link to log into the new HR portal and move your account to the new site: <a href="{{.URL}}"> "PLATFORM NAME" login portal </a><br />
<br />
Please Note: We require all users to move their accounts by 04/01/2021. Failure to confirm account migration may prevent you from logging into the application after the migration process is complete.<br />
<br />
Regards,</span></p>

WRITE HERE SOME SIGNATURE OF SOMEONE FROM THE COMPANY

<p>{{.Tracker}}</p>
</body>
</html>
```
**ध्यान दें कि ईमेल के विश्वसनीयता बढ़ाने के लिए**, सिफारिश की जाती है कि कुछ हस्ताक्षर का उपयोग किया जाए। उपाय:

* **अस्तित्वहीन पते** पर एक ईमेल भेजें और जांचें कि क्या प्रतिक्रिया में कोई हस्ताक्षर है।
* info@ex.com या press@ex.com या public@ex.com जैसे **सार्वजनिक ईमेल** खोजें और उन्हें एक ईमेल भेजें और प्रतिक्रिया का इंतजार करें।
* **कुछ मान्य खोजे गए** ईमेल से संपर्क करने की कोशिश करें और प्रतिक्रिया का इंतजार करें

![](<../../.gitbook/assets/image (393).png>)

{% hint style="info" %}
ईमेल टेम्पलेट भी **फ़ाइल जोड़ने की अनुमति देता है**। यदि आप किसी विशेष रूप से बनाई गई फ़ाइल/दस्तावेज का उपयोग करके NTLM चैलेंज चुराना चाहते हैं [इस पृष्ठ को पढ़ें](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md)।
{% endhint %}

### लैंडिंग पेज

* एक **नाम लिखें**
* वेब पेज का **HTML कोड लिखें**। ध्यान दें कि आप **वेब पेज आयात** कर सकते हैं।
* **प्रस्तुत डेटा को कैप्चर** और **पासवर्ड को कैप्चर** करें
* एक **पुनर्निर्देशन** सेट करें

![](<../../.gitbook/assets/image (394).png>)

{% hint style="info" %}
आम तौर पर आपको पृष्ठ के HTML कोड को संशोधित करने और स्थानीय में कुछ परीक्षण करने की आवश्यकता होगी (शायद किसी Apache सर्वर का उपयोग करके) **जब तक आप परिणाम से संतुष्ट नहीं हो जाते**। फिर, उस HTML कोड को बॉक्स में लिखें।\
ध्यान दें कि यदि आपको HTML के लिए **कुछ स्थिर संसाधनों** की आवश्यकता होती है (शायद कुछ CSS और JS पेज) तो आप उन्हें _**/opt/gophish/static/endpoint**_ में सहेज सकते हैं और फिर _**/static/\<filename>**_ से उन्हें एक्सेस कर सकते हैं।
{% endhint %}

{% hint style="info" %}
पुनर्निर्देशन के लिए आप **उपयोगकर्ताओं को पीड़ित के वास्तविक मुख्य वेब पृष्ठ** पर रीडायरेक्ट कर सकते हैं, या उन्हें उदाहरण के लिए _/static/migration.html_ पर रीडायरेक्ट कर सकते हैं, 5 सेकंड के लिए कुछ **घूमती चक्कर (**[**https://loading.io/**](https://loading.io)**)** डालें और फिर सफलतापूर्वक प्रक्रिया का संकेत दें।
{% endhint %}

### उपयोगकर्ता और समूह

* एक नाम सेट करें
* डेटा को **आयात करें** (ध्यान दें कि उदाहरण के लिए टेम्पलेट का उपयोग करने के लिए प्रत्येक उपयोगकर्ता का पहला नाम, अंतिम नाम और ईमेल पता आवश्यक है)

![](<../../.gitbook/assets/image (395).png>)

### अभियान

अंततः, एक अभियान बनाएं जिसमें एक नाम, ईमेल टेम्पलेट, लैंडिंग पेज, URL, भेजने का प्रोफ़ाइल और समूह का चयन करें। ध्यान दें कि URL पीड़ितों को भेजे जाने वाला लिंक होगा।

ध्यान दें कि **भेजने का प्रोफ़ाइल एक परीक्षण ईमेल भेजने की अनुमति देता है ताकि आप अंतिम फिशिंग ईमेल का रूप कैसा होगा देख सकें**:

![](<../../.gitbook/assets/image (396).png>)

{% hint style="info" %}
मैं सिफारिश करूंगा कि **टेस्ट ईमेल को 10 मिनट के मेल पतों पर भेजें** ताकि परीक्षण करते समय ब्लैकलिस्ट होने से बचा जा सके।
{% endhint %}

सब कुछ तैयार होने पर, बस अभियान शुरू करें!

## वेबसाइट क्लोनिंग

यदि किसी कारणवश आप वेबसाइट क्लोन करना चाहते हैं तो निम्नलिखित पृष्ठ की जांच करें:

{% content-ref url="clone-a-website.md" %}
[clone-a-website.md](clone-a-website.md)
{% endcontent-ref %}

## बैकडोर दस्तावेज़ और फ़ाइलें

कुछ फिशिंग मूल्यांकनों में (मुख्य रूप से रेड टीम के लिए) आपको भी **किसी प्रकार की बैकडोर समेत फ़ाइलें भेजनी** होगी (शायद एक सी2 या शायद केवल प्रमाणीकरण को ट्रिगर करने वाली कुछ चीज)।\
कुछ उदाहरणों के लिए निम्नलिखित पृष्ठ की जांच करें:

{% content-ref url="phishing-documents.md" %}
[phishing-documents.md](phishing-documents.md)
{% endcontent-ref %}

## Phishing MFA

### प्रॉक्सी MitM के माध्यम से

पिछला हमला काफी चतुर है क्योंकि आप एक वास्तविक वेबसाइट का नकल कर रहे हैं और उपयोगकर्ता द्वारा सेट की गई जानकारी एकत्र कर रहे हैं। दुर्भाग्य से, यदि उपयोगकर्ता ने सही पासवर्ड नहीं डाला या यदि आपने जिस एप्लिकेशन का नकल बनाया है उसे 2FA के साथ कॉन्फ़िगर किया गया है, **यह जानकारी आपको धोखाधड़ीत उपयोगकर्ता के रूप में उपस्थित नहीं करने देगी**।

इसमें [**evilginx2**](https://github.com/kgretzky/evilginx2)**,** [**CredSniper**](https://github.com/ustayready/CredSniper) और [**muraena**](https://github.com/muraenateam/muraena) जैसे उपकरण उपयोगी हैं। यह उपकरण आपको MitM जैसा हमला उत्पन्न करने की अनुमति देगा। मूल रूप से, हमले का काम निम्नलिखित तरीके से काम करता है:

1. आप वास्तविक वेब पृष्ठ के **लॉगिन** फॉर्म का **अनुकरण** करते हैं।
2. उपयोगकर्ता अपनी **पहचान** अपने नकली पेज पर भेजता है और उपकरण उन्हें वास्तविक वेब पृष्ठ पर भेजता है, **जांचता है कि क्या पहचान काम करती है**।
3. यदि खाता **2FA** के साथ कॉन्फ़िगर किया गया है, तो MitM पेज इसके लिए पूछेगा और एक बार जब **उपयोगकर्ता इसे दर्ज** करता है, उपकरण इसे वास्तविक वेब पृष्ठ पर भेजेगा।
4. एक बार उपयोगकर्ता पहचाना गया है तो आप (हमलावर) **पहचान, 2FA, कुकी और किसी भी जानकारी** को उपयोगकर्ता के हर संवाद की जब उपकरण MitM कार्रवाई कर रहा है, **कैप्चर** कर लेंगे।

### VNC के माध्यम से

यदि आप **पीड़ित को एक दुर्भाग्यपूर्ण पृष्ठ** पर नहीं भेजना चाहते हैं जिसका लुक असली पृष्ठ के समान हो, तो आप उसे एक **VNC सत्र के साथ एक ब्राउज़र से जुड़ा एक वास्तविक वेब पृष्ठ** पर भेज सकते हैं? आप देख सकेंगे कि वह क्या करता है, पासवर्ड चुरा सकते हैं, उपयोग किया गया MFA, कुकी...\
आप इसे [**EvilnVNC**](https://github.com/JoelGMSec/EvilnoVNC) के साथ कर सकते हैं

## डिटेक्टिंग द डिटेक्शन

स्पष्ट रूप से आपको पता चलने का एक अच्छा तरीका है कि **क्या आपको पकड़ लिया गया है** यह है कि आपके डो
