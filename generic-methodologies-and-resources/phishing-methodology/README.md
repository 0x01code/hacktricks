# फ़िशिंग पद्धति

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

## पद्धति

1. पीड़ित की पहचान करें
1. **पीड़ित डोमेन** का चयन करें।
2. पीड़ित द्वारा प्रयुक्त **लॉगिन पोर्टल्स की खोज करें** और **निर्णय लें** कि आप किसे **नकली बनाएंगे**।
3. **OSINT** का उपयोग करके **ईमेल खोजें**।
2. पर्यावरण तैयार करें
1. फ़िशिंग मूल्यांकन के लिए आप जो **डोमेन खरीदेंगे**
2. ईमेल सेवा से संबंधित रिकॉर्ड्स (SPF, DMARC, DKIM, rDNS) **कॉन्फ़िगर करें**
3. **gophish** के साथ VPS कॉन्फ़िगर करें
3. अभियान तैयार करें
1. **ईमेल टेम्पलेट** तैयार करें
2. क्रेडेंशियल्स चुराने के लिए **वेब पेज** तैयार करें
4. अभियान शुरू करें!

## समान डोमेन नाम उत्पन्न करें या विश्वसनीय डोमेन खरीदें

### डोमेन नाम विविधता तकनीकें

* **कीवर्ड**: डोमेन नाम में मूल डोमेन का एक महत्वपूर्ण **कीवर्ड शामिल** होता है (उदा., zelster.com-management.com)।
* **हाइफ़नेड सबडोमेन**: सबडोमेन के **डॉट को हाइफ़न से बदलें** (उदा., www-zelster.com)।
* **नया TLD**: एक **नए TLD** का उपयोग करते हुए समान डोमेन (उदा., zelster.org)।
* **होमोग्लिफ़**: डोमेन नाम में एक अक्षर को **समान दिखने वाले अक्षरों से बदलें** (उदा., zelfser.com)।
* **ट्रांसपोज़िशन**: डोमेन नाम के भीतर **दो अक्षरों को आदान-प्रदान करें** (उदा., zelster.com)।
* **सिंगुलराइज़ेशन/प्लुरलाइज़ेशन**: डोमेन नाम के अंत में "s" जोड़ें या हटाएं (उदा., zeltsers.com)।
* **ओमिशन**: डोमेन नाम से एक अक्षर **हटाएं** (उदा., zelser.com)।
* **रिपीटिशन**: डोमेन नाम में एक अक्षर को **दोहराएं** (उदा., zeltsser.com)।
* **रिप्लेसमेंट**: होमोग्लिफ़ की तरह लेकिन कम चुपके से। डोमेन नाम में एक अक्षर को बदलें, शायद कीबोर्ड पर मूल अक्षर के निकटता में एक अक्षर के साथ (उदा, zektser.com)।
* **सबडोमेनेड**: डोमेन नाम के अंदर एक **डॉट** डालें (उदा., ze.lster.com)।
* **इंसर्शन**: डोमेन नाम में एक अक्षर **डालें** (उदा., zerltser.com)।
* **मिसिंग डॉट**: TLD को डोमेन नाम के साथ जोड़ें। (उदा., zelstercom.com)

**ऑटोमैटिक टूल्स**

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy)

**वेबसाइट्स**

* [https://dnstwist.it/](https://dnstwist.it)
* [https://dnstwister.report/](https://dnstwister.report)
* [https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/](https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/)

### बिटफ्लिपिंग

कंप्यूटिंग की दुनिया में, सब कुछ पीछे के दृश्यों में मेमोरी में बिट्स (शून्य और एक) में संग्रहीत होता है।\
यह डोमेन्स पर भी लागू होता है। उदाहरण के लिए, _windows.com_ आपके कंप्यूटिंग डिवाइस की अस्थायी मेमोरी में _01110111..._ बन जाता है।\
हालांकि, अगर इन बिट्स में से एक सौर फ्लेयर, कॉस्मिक किरणों, या हार्डवेयर त्रुटि के कारण स्वतः फ्लिप हो जाता है? यानी एक 0 एक 1 में बदल जाता है और इसके विपरीत।\
DNS अनुरोध की इस अवधारणा को लागू करते हुए, संभव है कि DNS सर्वर को **प्राप्त होने वाला अनुरोधित डोमेन** मूल रूप से अनुरोधित डोमेन के समान नहीं है।

उदाहरण के लिए, windows.com डोमेन में 1 बिट का परिवर्तन इसे _windnws.com_ में बदल सकता है।\
**हमलावर वैध उपयोगकर्ताओं को अपने इंफ्रास्ट्रक्चर की ओर मोड़ने के लिए पीड़ित से संबंधित जितने संभव हो सके बिट-फ्लिपिंग डोमेन्स को रजिस्टर कर सकते हैं**।

अधिक जानकारी के लिए [https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/](https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/) पढ़ें।

### विश्वसनीय डोमेन खरीदें

आप [https://www.expireddomains.net/](https://www.expireddomains.net) में एक समाप्त डोमेन की खोज कर सकते हैं जिसे आप उपयोग कर सकते हैं।\
यह सुनिश्चित करने के लिए कि आप जो समाप्त डोमेन खरीदने जा रहे हैं **पहले से ही एक अच्छा SEO है**, आप इसे कैसे वर्गीकृत किया गया है, इसकी खोज कर सकते हैं:

* [http://www.fortiguard.com/webfilter](http://www.fortiguard.com/webfilter)
* [https://urlfiltering.paloaltonetworks.com/query/](https://urlfiltering.paloaltonetworks.com/query/)

## ईमेल्स की खोज

* [https://github.com/laramies/theHarvester](https://github.com/laramies/theHarvester) (100% मुफ्त)
* [https://phonebook.cz/](https://phonebook.cz) (100% मुफ्त)
* [https://maildb.io/](https://maildb.io)
* [https://hunter.io/](https://hunter.io)
* [https://anymailfinder.com/](https://anymailfinder.com)

अधिक मान्य ईमेल पते **खोजने के लिए** या आपके द्वारा पहले ही खोजे गए ईमेल पतों की **पुष्टि करने के लिए** आप जांच सकते हैं कि क्या आप पीड़ित के smtp सर्वरों को ब्रूट-फोर्स कर सकते हैं। [यहां जानें कि ईमेल पते की पुष्टि/खोज कैसे करें](../../network-services-pentesting/pentesting-smtp/#username-bruteforce-enumeration).\
इसके अलावा, यह मत भूलें कि यदि उपयोगकर्ता **किसी वेब पोर्टल का उपयोग अपने मेल्स तक पहुंचने के लिए करते हैं**, तो आप जांच सकते हैं कि क्या यह **यूजरनेम ब्रूट फोर्स** के लिए संवेदनशील है, और यदि संभव हो तो इस कमजोरी का शोषण करें।

## GoPhish कॉन्फ़िगर करना

### इंस्टालेशन

आप इसे [https://github.com/gophish/gophish/releases/tag/v0.11.0](https://github.com/gophish/gophish/releases/tag/v0.11.0) से ड
```bash
ssh -L 3333:127.0.0.1:3333 <user>@<ip>
```
### कॉन्फ़िगरेशन

**TLS प्रमाणपत्र कॉन्फ़िगरेशन**

इस चरण से पहले आपको **पहले से ही डोमेन खरीदना चाहिए** जिसका उपयोग आप करने वाले हैं और यह **IP पर इंगित कर रहा होना चाहिए** जो **VPS का है** जहाँ आप **gophish** कॉन्फ़िगर कर रहे हैं।
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

फिर निम्नलिखित फाइलों में डोमेन जोड़ें:

* **/etc/postfix/virtual\_domains**
* **/etc/postfix/transport**
* **/etc/postfix/virtual\_regexp**

**/etc/postfix/main.cf में निम्नलिखित वेरिएबल्स के मान भी बदलें**

`myhostname = <domain>`\
`mydestination = $myhostname, <domain>, localhost.com, localhost`

अंत में **`/etc/hostname`** और **`/etc/mailname`** फाइलों को आपके डोमेन नाम से मॉडिफाई करें और **अपने VPS को रीस्टार्ट करें।**

अब, **DNS A रिकॉर्ड** `mail.<domain>` बनाएं जो VPS के **आईपी पते** की ओर इंगित करता है और `mail.<domain>` की ओर इंगित करने वाला एक **DNS MX** रिकॉर्ड बनाएं।

अब चलिए एक ईमेल भेजने का परीक्षण करते हैं:
```bash
apt install mailutils
echo "This is the body of the email" | mail -s "This is the subject line" test@email.com
```
**Gophish समाकृति**

Gophish का निष्पादन रोकें और इसे समाकृत करें।\
`/opt/gophish/config.json` को निम्नलिखित में परिवर्तित करें (https के प्रयोग का ध्यान रखें):
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
**gophish सेवा को कॉन्फ़िगर करें**

gophish सेवा को स्वचालित रूप से शुरू किया जा सके और एक सेवा के रूप में प्रबंधित किया जा सके, आप निम्नलिखित सामग्री के साथ `/etc/init.d/gophish` फ़ाइल बना सकते हैं:
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
सेवा को कॉन्फ़िगर करना समाप्त करें और इसे जांचें:
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
## मेल सर्वर और डोमेन को कॉन्फ़िगर करना

### प्रतीक्षा करें

एक डोमेन जितना पुराना होता है, उसे स्पैम के रूप में पकड़े जाने की संभावना उतनी ही कम होती है। इसलिए आपको फ़िशिंग मूल्यांकन से पहले जितना संभव हो सके उतना समय (कम से कम 1 सप्ताह) प्रतीक्षा करनी चाहिए।\
ध्यान दें कि भले ही आपको एक सप्ताह प्रतीक्षा करनी पड़े, आप अभी सब कुछ कॉन्फ़िगर कर सकते हैं।

### रिवर्स DNS (rDNS) रिकॉर्ड कॉन्फ़िगर करें

VPS के IP पते को डोमेन नाम से हल करने के लिए एक rDNS (PTR) रिकॉर्ड सेट करें।

### सेंडर पॉलिसी फ्रेमवर्क (SPF) रिकॉर्ड

आपको **नए डोमेन के लिए एक SPF रिकॉर्ड कॉन्फ़िगर करना चाहिए**। यदि आपको नहीं पता कि SPF रिकॉर्ड क्या है [**इस पृष्ठ को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#spf)।

आप [https://www.spfwizard.net/](https://www.spfwizard.net) का उपयोग करके अपनी SPF नीति उत्पन्न कर सकते हैं (VPS मशीन के IP का उपयोग करें)

![](<../../.gitbook/assets/image (388).png>)

यह वह सामग्री है जिसे डोमेन के अंदर एक TXT रिकॉर्ड के भीतर सेट किया जाना चाहिए:
```bash
v=spf1 mx a ip4:ip.ip.ip.ip ?all
```
### डोमेन-आधारित संदेश प्रमाणीकरण, रिपोर्टिंग और संगतता (DMARC) रिकॉर्ड

आपको **नए डोमेन के लिए DMARC रिकॉर्ड कॉन्फ़िगर करना चाहिए**। यदि आपको नहीं पता कि DMARC रिकॉर्ड क्या है [**इस पृष्ठ को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#dmarc)।

आपको नया DNS TXT रिकॉर्ड बनाना होगा जो कि होस्टनेम `_dmarc.<domain>` को निम्नलिखित सामग्री के साथ पॉइंट करे:
```bash
v=DMARC1; p=none
```
### DomainKeys Identified Mail (DKIM)

आपको **नए डोमेन के लिए DKIM कॉन्फ़िगर करना चाहिए**। यदि आपको नहीं पता कि DMARC रिकॉर्ड क्या है [**इस पृष्ठ को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#dkim).

यह ट्यूटोरियल आधारित है: [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)

{% hint style="info" %}
आपको DKIM कुंजी द्वारा उत्पन्न दोनों B64 मानों को जोड़ना होगा:
```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0wPibdqPtzYk81njjQCrChIcHzxOp8a1wjbsoNtka2X9QXCZs+iXkvw++QsWDtdYu3q0Ofnr0Yd/TmG/Y2bBGoEgeE+YTUG2aEgw8Xx42NLJq2D1pB2lRQPW4IxefROnXu5HfKSm7dyzML1gZ1U0pR5X4IZCH0wOPhIq326QjxJZm79E1nTh3xj" "Y9N/Dt3+fVnIbMupzXE216TdFuifKM6Tl6O/axNsbswMS1TH812euno8xRpsdXJzFlB9q3VbMkVWig4P538mHolGzudEBg563vv66U8D7uuzGYxYT4WS8NVm3QBMg0QKPWZaKp+bADLkOSB9J2nUpk4Aj9KB5swIDAQAB
```
{% endhint %}

### अपने ईमेल कॉन्फ़िगरेशन स्कोर का परीक्षण करें

आप यह [https://www.mail-tester.com/](https://www.mail-tester.com) का उपयोग करके कर सकते हैं।\
बस पेज पर जाएँ और उनके द्वारा दिए गए पते पर एक ईमेल भेजें:
```bash
echo "This is the body of the email" | mail -s "This is the subject line" test-iimosa79z@srv1.mail-tester.com
```
आप अपने ईमेल कॉन्फ़िगरेशन की जांच भी कर सकते हैं, `check-auth@verifier.port25.com` पर एक ईमेल भेजकर और प्रतिक्रिया पढ़कर (इसके लिए आपको पोर्ट **25** को **खोलना** होगा और फ़ाइल _/var/mail/root_ में प्रतिक्रिया देखनी होगी अगर आपने ईमेल रूट के रूप में भेजा है)।\
जांचें कि आप सभी परीक्षणों में पास होते हैं:
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
वैकल्पिक रूप से, आप एक **Gmail पते पर संदेश भेज सकते हैं जिस पर आपका नियंत्रण हो**, **देखें** अपने Gmail इनबॉक्स में प्राप्त **ईमेल के हेडर्स** को, `dkim=pass` को `Authentication-Results` हेडर फील्ड में मौजूद होना चाहिए।
```
Authentication-Results: mx.google.com;
spf=pass (google.com: domain of contact@example.com designates --- as permitted sender) smtp.mail=contact@example.com;
dkim=pass header.i=@example.com;
```
### Spamhouse Blacklist से हटाना

पेज www.mail-tester.com आपको बता सकता है अगर आपका डोमेन Spamhouse द्वारा ब्लॉक किया जा रहा है। आप अपने डोमेन/IP को हटाने के लिए अनुरोध कर सकते हैं: [https://www.spamhaus.org/lookup/](https://www.spamhaus.org/lookup/)

### Microsoft Blacklist से हटाना

आप अपने डोमेन/IP को हटाने के लिए अनुरोध कर सकते हैं [https://sender.office.com/](https://sender.office.com).

## GoPhish अभियान बनाएं और लॉन्च करें

### Sending Profile

* कोई **नाम चुनें** जिससे सेंडर प्रोफाइल की पहचान हो सके
* तय करें कि आप किस अकाउंट से फ़िशिंग ईमेल भेजने वाले हैं। सुझाव: _noreply, support, servicedesk, salesforce..._
* आप यूजरनेम और पासवर्ड खाली छोड़ सकते हैं, लेकिन सुनिश्चित करें कि आपने Ignore Certificate Errors को चेक किया है

![](<../../.gitbook/assets/image (253) (1) (2) (1) (1) (2) (2) (3) (3) (5) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (17).png>)

{% hint style="info" %}
"**Send Test Email**" कार्यक्षमता का उपयोग करना सिफारिश की जाती है ताकि यह जांचा जा सके कि सब कुछ काम कर रहा है।\
मैं सिफारिश करूंगा कि **परीक्षण ईमेल 10min मेल पतों पर भेजें** ताकि परीक्षण करते समय ब्लैकलिस्टेड होने से बचा जा सके।
{% endhint %}

### Email Template

* कोई **नाम चुनें** जिससे टेम्पलेट की पहचान हो सके
* फिर एक **विषय** लिखें (कुछ अजीब नहीं, बस कुछ ऐसा जो आप एक सामान्य ईमेल में पढ़ने की उम्मीद कर सकते हैं)
* सुनिश्चित करें कि आपने "**Add Tracking Image**" को चेक किया है
* **ईमेल टेम्पलेट** लिखें (आप निम्नलिखित उदाहरण की तरह वेरिएबल्स का उपयोग कर सकते हैं):
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
ध्यान दें कि **ईमेल की विश्वसनीयता बढ़ाने के लिए**, ग्राहक के ईमेल से किसी हस्ताक्षर का उपयोग करना सुझावित है। सुझाव:

* **अस्तित्वहीन पते** पर एक ईमेल भेजें और जांचें कि क्या प्रतिक्रिया में कोई हस्ताक्षर है।
* **सार्वजनिक ईमेल** जैसे info@ex.com या press@ex.com या public@ex.com पर खोजें और उन्हें एक ईमेल भेजें और प्रतिक्रिया की प्रतीक्षा करें।
* **कुछ मान्य पता लगाए गए** ईमेल से संपर्क करने का प्रयास करें और प्रतिक्रिया की प्रतीक्षा करें।

![](<../../.gitbook/assets/image (393).png>)

{% hint style="info" %}
ईमेल टेम्पलेट आपको **फाइलें भेजने के लिए संलग्न करने की भी अनुमति देता है**। यदि आप कुछ विशेष रूप से तैयार की गई फाइलों/दस्तावेजों का उपयोग करके NTLM चुनौतियों को चुराना चाहते हैं [इस पृष्ठ को पढ़ें](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md)।
{% endhint %}

### लैंडिंग पेज

* एक **नाम** लिखें
* वेब पेज का **HTML कोड लिखें**। ध्यान दें कि आप वेब पेजों को **आयात** कर सकते हैं।
* **कैप्चर सबमिटेड डेटा** और **कैप्चर पासवर्ड** को चिह्नित करें
* एक **पुनर्निर्देशन** सेट करें

![](<../../.gitbook/assets/image (394).png>)

{% hint style="info" %}
आमतौर पर आपको पृष्ठ के HTML कोड को संशोधित करने और कुछ परीक्षण स्थानीय रूप से करने की आवश्यकता होगी (शायद कुछ Apache सर्वर का उपयोग करके) **जब तक आप परिणामों से संतुष्ट न हों।** फिर, उस HTML कोड को बॉक्स में लिखें।\
ध्यान दें कि यदि आपको HTML के लिए कुछ स्थिर संसाधनों का **उपयोग करने की आवश्यकता है** (शायद कुछ CSS और JS पेज) तो आप उन्हें _**/opt/gophish/static/endpoint**_ में सहेज सकते हैं और फिर _**/static/\<filename>**_ से उन तक पहुंच सकते हैं।
{% endhint %}

{% hint style="info" %}
पुनर्निर्देशन के लिए आप **उपयोगकर्ताओं को पीड़ित के वैध मुख्य वेब पेज पर पुनर्निर्देशित कर सकते हैं**, या उन्हें _/static/migration.html_ पर पुनर्निर्देशित कर सकते हैं, उदाहरण के लिए, कुछ **स्पिनिंग व्हील (**[**https://loading.io/**](https://loading.io)**) को 5 सेकंड के लिए डालें और फिर संकेत दें कि प्रक्रिया सफल रही**।
{% endhint %}

### उपयोगकर्ता और समूह

* एक नाम सेट करें
* **डेटा आयात करें** (ध्यान दें कि उदाहरण के लिए टेम्पलेट का उपयोग करने के लिए आपको प्रत्येक उपयोगकर्ता का पहला नाम, अंतिम नाम और ईमेल पता चाहिए)

![](<../../.gitbook/assets/image (395).png>)

### अभियान

अंत में, एक नाम, ईमेल टेम्पलेट, लैंडिंग पेज, URL, प्रोफाइल भेजना और समूह का चयन करके एक अभियान बनाएं। ध्यान दें कि URL पीड़ितों को भेजा जाने वाला लिंक होगा।

ध्यान दें कि **भेजने वाला प्रोफाइल आपको अंतिम फ़िशिंग ईमेल कैसा दिखेगा यह देखने के लिए एक परीक्षण ईमेल भेजने की अनुमति देता है**:

![](<../../.gitbook/assets/image (396).png>)

{% hint style="info" %}
मैं सुझाव दूंगा कि **परीक्षण ईमेल 10min मेल पतों पर भेजें** ताकि परीक्षण करते समय ब्लैकलिस्टेड होने से बचा जा सके।
{% endhint %}

सब कुछ तैयार होने के बाद, बस अभियान शुरू करें!

## वेबसाइट क्लोनिंग

यदि किसी कारण से आप वेबसाइट को क्लोन करना चाहते हैं तो निम्नलिखित पृष्ठ देखें:

{% content-ref url="clone-a-website.md" %}
[clone-a-website.md](clone-a-website.md)
{% endcontent-ref %}

## बैकडोर्ड दस्तावेज और फाइलें

कुछ फ़िशिंग मूल्यांकनों में (मुख्य रूप से रेड टीमों के लिए) आप कुछ प्रकार के बैकडोर वाली फाइलें भी **भेजना चाहेंगे** (शायद एक C2 या शायद बस कुछ ऐसा जो प्रमाणीकरण को ट्रिगर करेगा)।\
कुछ उदाहरणों के लिए निम्नलिखित पृष्ठ देखें:

{% content-ref url="phishing-documents.md" %}
[phishing-documents.md](phishing-documents.md)
{% endcontent-ref %}

## फ़िशिंग MFA

### प्रॉक्सी MitM के माध्यम से

पिछला हमला काफी चतुर है क्योंकि आप एक वास्तविक वेबसाइट का नकली बना रहे हैं और उपयोगकर्ता द्वारा सेट की गई जानकारी एकत्र कर रहे हैं। दुर्भाग्य से, अगर उपयोगकर्ता ने सही पासवर्ड नहीं डाला है या आपके द्वारा नकली बनाई गई एप्लिकेशन 2FA के साथ कॉन्फ़िगर की गई है, **तो यह जानकारी आपको धोखा दिए गए उपयोगकर्ता की नकल करने की अनुमति नहीं देगी**।

यहां [**evilginx2**](https://github.com/kgretzky/evilginx2)**,** [**CredSniper**](https://github.com/ustayready/CredSniper) और [**muraena**](https://github.com/muraenateam/muraena) जैसे उपकरण उपयोगी हैं। यह उपकरण आपको MitM जैसे हमले उत्पन्न करने की अनुमति देगा। मूल रूप से, हमले निम्नलिखित तरीके से काम करते हैं:

1. आप **लॉगिन फॉर्म की नकल करते हैं** वास्तविक वेबपेज की।
2. उपयोगकर्ता अपने **प्रमाण-पत्र** आपके नकली पृष्ठ पर **भेजता है** और उपकरण उन्हें वास्तविक वेबपेज पर भेजता है, **जांचता है कि प्रमाण-पत्र काम करते हैं या नहीं**।
3. अगर खाता **2FA** के साथ कॉन्फ़िगर किया गया है, तो MitM पृष्ठ इसके लिए पूछेगा और एक बार **उपयोगकर्ता इसे पेश करता है** उपकरण इसे वास्तविक वेब पेज पर भेज देगा।
4. एक बार उपयोगकर्ता प्रमाणित हो जाता है तो आप (हमलावर के रूप में) **प्रमाण-पत्र, 2FA, कुकी और हर इंटरैक्शन की कोई भी जानकारी कैप्चर कर लेंगे** जबकि उपकरण MitM का प्रदर्शन कर रहा है।

### VNC के माध्यम से

क्या होगा अगर बजाय **पीड़ित को एक दुर्भावनापूर्ण पृष्ठ पर भेजने के** जो मूल की तरह दिखता है, आप उसे एक **VNC सत्र में भेजते हैं जिसमें एक ब्राउज़र वास्तविक वेब पेज से जुड़ा होता है**? आप देख पाएंगे कि वह क्या करता है, पासवर्ड चुरा सकते हैं, MFA का उपयोग करते हैं, कुकीज़...\
आप इसे [**EvilnVNC**](https://github.com/JoelGMSec/EvilnoVNC) के साथ कर सकते हैं।

## पता लगाने का पता लगाना

स्वाभाविक रूप से यह जानने का सबसे अच्छा तरीका कि आप पकड़े गए हैं या नहीं,
