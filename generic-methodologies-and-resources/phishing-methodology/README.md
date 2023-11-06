# फिशिंग मेथडोलॉजी

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) को PR जमा करके।

</details>

## मेथडोलॉजी

1. पीडीएन विचार करें
1. **पीडीएन डोमेन** का चयन करें।
2. पीडीएन द्वारा उपयोग किए जाने वाले कुछ मूलभूत वेब जांच करें **लॉगिन पोर्टल की खोज** करें और निर्धारित करें कि आप किसे **अनुकरण** करेंगे।
3. कुछ **OSINT** का उपयोग करें ईमेल्स की **खोज** करने के लिए।
2. पर्यावरण की तैयारी करें
1. फिशिंग मूल्यांकन के लिए उपयोग करने वाले डोमेन को **खरीदें**
2. ईमेल सेवा संबंधित रिकॉर्ड (SPF, DMARC, DKIM, rDNS) को **कॉन्फ़िगर** करें
3. **गोफिश** के साथ VPS को कॉन्फ़िगर करें
3. अभियान की तैयारी करें
1. **ईमेल टेम्पलेट** की तैयारी करें
2. प्रमाणपत्रों को चुराने के लिए **वेब पेज** की तैयारी करें
4. अभियान शुरू करें!

## समान डोमेन नाम उत्पन्न करें या विश्वसनीय डोमेन खरीदें

### डोमेन नाम विविधता तकनीकें

* **कीवर्ड**: डोमेन नाम में मूल डोमेन का महत्वपूर्ण **कीवर्ड होता है** (उदाहरण के लिए, zelster.com-management.com).
* **हाइफ़न सबडोमेन**: सबडोमेन के लिए **डॉट को हाइफ़न में बदलें** (उदाहरण के लिए, www-zelster.com).
* **नया TLD**: एक **नया TLD** का उपयोग करके समान डोमेन (उदाहरण के लिए, zelster.org)
* **होमोग्लिफ**: डोमेन नाम में एक अक्षर को बदलकर उसके **समान दिखने वाले अक्षरों** से बदलता है (उदाहरण के लिए, zelfser.com).
* **ट्रांसपोज़िशन:** डोमेन नाम में दो अक्षरों को **आपस में बदल देता है** (उदाहरण के लिए, zelster.com).
* **एकलीकरण/बहुवचनीकरण**: डोमेन नाम के अंत में "s" जोड़ता है या हटाता है (उदाहरण के लिए, zeltsers.com).
* **छूट**: डोमेन नाम से एक अक्षर को **हटा देता है** (उदाहरण के लिए, zelser.com).
* **दोहराना**: डोमेन नाम में एक अक्षर को **दोहराता है** (उदाहरण के लिए, zeltsser.com).
* **प्रतिस्थापन**: होमोग्लिफ की तरह है लेकिन कम छलांगी। डोमेन नाम में एक अक्षर को बदलता है, शायद मूल अक्षर के पास के अक्षर के साथ (उदाहरण के लिए, zektser.com).
* **सबडोमेन्ड**: डोमेन नाम में एक **डॉट** डालें (उदाहरण के लिए, ze.lster.com).
* **इंजेक्शन**: डोमेन नाम में एक अक्षर **डालता है** (उदाहरण के लिए, zerltser.com).
* **गुम हुआ डॉट**: डोमेन नाम के बाद TLD जोड़ें। (उदाहरण के लिए, zelstercom.com)

**स्वचालित उपकरण**

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://
### एक विश्वसनीय डोमेन खरीदें

आप [https://www.expireddomains.net/](https://www.expireddomains.net) में एक समाप्त हो चुके डोमेन की खोज कर सकते हैं जिसे आप उपयोग कर सकते हैं।\
यह सुनिश्चित करने के लिए कि आप खरीदने जा रहे समाप्त डोमेन में **पहले से ही अच्छा SEO है**, आप यह देख सकते हैं कि यह कैसे श्रेणीबद्ध है:

* [http://www.fortiguard.com/webfilter](http://www.fortiguard.com/webfilter)
* [https://urlfiltering.paloaltonetworks.com/query/](https://urlfiltering.paloaltonetworks.com/query/)

## ईमेल खोज

* [https://github.com/laramies/theHarvester](https://github.com/laramies/theHarvester) (100% मुफ्त)
* [https://phonebook.cz/](https://phonebook.cz) (100% मुफ्त)
* [https://maildb.io/](https://maildb.io)
* [https://hunter.io/](https://hunter.io)
* [https://anymailfinder.com/](https://anymailfinder.com)

अधिक मान्य ईमेल पते खोजने या पहले से पता लगाए गए पतों की **पुष्टि करने** के लिए आप पीडीएमटीपी सर्वर को ब्रूट-फोर्स कर सकते हैं। [यहां ईमेल पते की पुष्टि/खोजने का तरीका सीखें](../../network-services-pentesting/pentesting-smtp/#username-bruteforce-enumeration)।\
इसके अलावा, यदि उपयोगकर्ता अपने मेल तक पहुंचने के लिए **किसी वेब पोर्टल का उपयोग करते हैं**, तो आप यह देख सकते हैं कि क्या यह **उपयोगकर्ता नाम ब्रूट-फोर्स** के लिए संवेदनशील है, और यदि संभव हो तो उस संवेदनशीलता का उपयोग करें।

## GoPhish को कॉन्फ़िगर करना

### स्थापना

आप इसे [https://github.com/gophish/gophish/releases/tag/v0.11.0](https://github.com/gophish/gophish/releases/tag/v0.11.0) से डाउनलोड कर सकते हैं।

इसे `/opt/gophish` में डाउनलोड और डीकंप्रेस करें और `/opt/gophish/gophish` को चलाएं।\
आपको पोर्ट 3333 में एडमिन उपयोगकर्ता के लिए एक पासवर्ड दिया जाएगा आउटपुट में। इसलिए, उस पोर्ट तक पहुंचें और उन क्रेडेंशियल का उपयोग करके एडमिन पासवर्ड बदलें। आपको शायद इस पोर्ट को स्थानिक: पर टनल करने की आवश्यकता हो सकती है।
```bash
ssh -L 3333:127.0.0.1:3333 <user>@<ip>
```
### कॉन्फ़िगरेशन

**TLS प्रमाणपत्र कॉन्फ़िगरेशन**

इस स्टेप से पहले आपको **पहले से ही खरीद लिया होना चाहिए डोमेन** जिसे आप उपयोग करने जा रहे हैं और यह **इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट पर इंटरनेट प
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

**/etc/postfix/main.cf** के अंदर निम्नलिखित चरों की मानें भी बदलें

`myhostname = <domain>`\
`mydestination = $myhostname, <domain>, localhost.com, localhost`

अंत में, फ़ाइलें **`/etc/hostname`** और **`/etc/mailname`** को अपने डोमेन नाम पर संशोधित करें और **अपने VPS को रीस्टार्ट करें।**

अब, `mail.<domain>` के लिए **DNS A रिकॉर्ड** बनाएं जो VPS के **IP पते** को पॉइंट करता है और `mail.<domain>` के लिए **DNS MX रिकॉर्ड** बनाएं।

अब हम एक ईमेल भेजने का परीक्षण करें:
```bash
apt install mailutils
echo "This is the body of the email" | mail -s "This is the subject line" test@email.com
```
**Gophish विन्यास**

Gophish के निष्पादन को रोकें और इसे विन्यासित करें।\
`/opt/gophish/config.json` को निम्नलिखित रूप में संशोधित करें (https का उपयोग करने का ध्यान दें):
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

गोफिश सेवा को स्वचालित रूप से शुरू करने और सेवा को प्रबंधित करने के लिए आप निम्नलिखित सामग्री के साथ फ़ाइल `/etc/init.d/gophish` बना सकते हैं:
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
सेवा को विन्यासित करने और इसे जांचने के लिए निम्न कार्रवाई को पूरा करें:
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

जितना पुराना एक डोमेन होगा, उतना ही कम संभावित है कि यह स्पैम के रूप में पकड़ा जाएगा। इसलिए आपको फिशिंग मूल्यांकन से पहले जितना समय हो सके (कम से कम 1 सप्ताह) प्रतीक्षा करनी चाहिए।\
ध्यान दें कि यदि आपको एक सप्ताह इंतजार करना होता है, तो आप अभी सब कुछ कॉन्फ़िगर कर सकते हैं।

### Reverse DNS (rDNS) रिकॉर्ड कॉन्फ़िगर करें

एक rDNS (PTR) रिकॉर्ड कॉन्फ़िगर करें जो VPS के IP पते को डोमेन नाम में विलोपित करता है।

### Sender Policy Framework (SPF) रिकॉर्ड

आपको **नए डोमेन के लिए एक SPF रिकॉर्ड कॉन्फ़िगर करना होगा**। यदि आपको नहीं पता कि SPF रिकॉर्ड क्या है, तो [**इस पृष्ठ को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#spf)।

आप [https://www.spfwizard.net/](https://www.spfwizard.net) का उपयोग करके अपनी SPF नीति उत्पन्न कर सकते हैं (VPS मशीन का IP पता उपयोग करें)

![](<../../.gitbook/assets/image (388).png>)

यह विषय जो डोमेन के भीतर एक TXT रिकॉर्ड के रूप में सेट किया जाना चाहिए, यहां दिया गया है:
```bash
v=spf1 mx a ip4:ip.ip.ip.ip ?all
```
### डोमेन-आधारित संदेश प्रमाणीकरण, रिपोर्टिंग और अनुरूपता (DMARC) रिकॉर्ड

आपको **नए डोमेन के लिए एक DMARC रिकॉर्ड कॉन्फ़िगर करना होगा**। यदि आपको पता नहीं है कि DMARC रिकॉर्ड क्या है, तो [**इस पेज को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#dmarc)।

आपको निम्नलिखित सामग्री के साथ एक नया DNS TXT रिकॉर्ड बनाना होगा, जिसमें होस्टनाम `_dmarc.<डोमेन>` को इंगित करना होगा:
```bash
v=DMARC1; p=none
```
### डोमेनकीज आईडेंटिफाइड मेल (DKIM)

आपको **नए डोमेन के लिए डीकेआईएम कॉन्फ़िगर करना होगा**। अगर आपको यह पता नहीं है कि डीमार्क रिकॉर्ड क्या होता है, तो [**इस पेज को पढ़ें**](../../network-services-pentesting/pentesting-smtp/#dkim)।

यह ट्यूटोरियल इस पर आधारित है: [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)

{% hint style="info" %}
आपको डीकेआईएम की कुंजी द्वारा उत्पन्न दो B64 मानों को जोड़ना होगा:
```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0wPibdqPtzYk81njjQCrChIcHzxOp8a1wjbsoNtka2X9QXCZs+iXkvw++QsWDtdYu3q0Ofnr0Yd/TmG/Y2bBGoEgeE+YTUG2aEgw8Xx42NLJq2D1pB2lRQPW4IxefROnXu5HfKSm7dyzML1gZ1U0pR5X4IZCH0wOPhIq326QjxJZm79E1nTh3xj" "Y9N/Dt3+fVnIbMupzXE216TdFuifKM6Tl6O/axNsbswMS1TH812euno8xRpsdXJzFlB9q3VbMkVWig4P538mHolGzudEBg563vv66U8D7uuzGYxYT4WS8NVm3QBMg0QKPWZaKp+bADLkOSB9J2nUpk4Aj9KB5swIDAQAB
```
{% endhint %}

### अपने ईमेल कॉन्फ़िगरेशन स्कोर की जांच करें

आप इसका उपयोग करके कर सकते हैं [https://www.mail-tester.com/](https://www.mail-tester.com)\
बस पृष्ठ तक पहुंचें और उन्हें दिए गए पते पर एक ईमेल भेजें:
```bash
echo "This is the body of the email" | mail -s "This is the subject line" test-iimosa79z@srv1.mail-tester.com
```
आप अपने ईमेल कॉन्फ़िगरेशन की जांच भी कर सकते हैं, `check-auth@verifier.port25.com` पर एक ईमेल भेजकर और प्रतिक्रिया पढ़कर (इसके लिए आपको पोर्ट 25 खोलने की आवश्यकता होगी और यदि आप ईमेल को रूट के रूप में भेजते हैं तो फ़ाइल _/var/mail/root_ में प्रतिक्रिया देखें)।\
सुनिश्चित करें कि आप सभी परीक्षणों में सफलता प्राप्त करते हैं:
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
वैकल्पिक रूप से, आप एक Gmail पते को नियंत्रण करने वाले एक संदेश को भेज सकते हैं, अपने Gmail इनबॉक्स में प्राप्त किए गए ईमेल के हेडर्स को देखें, `Authentication-Results` हेडर फ़ील्ड में `dkim=pass` मौजूद होना चाहिए।
```
Authentication-Results: mx.google.com;
spf=pass (google.com: domain of contact@example.com designates --- as permitted sender) smtp.mail=contact@example.com;
dkim=pass header.i=@example.com;
```
### स्पैमहाउस ब्लैकलिस्ट से हटाना

पृष्ठ www.mail-tester.com आपको बता सकता है कि क्या आपका डोमेन स्पैमहाउस द्वारा अवरुद्ध हो रहा है। आप अपने डोमेन/आईपी को हटाने के लिए यहां अनुरोध कर सकते हैं: [https://www.spamhaus.org/lookup/](https://www.spamhaus.org/lookup/)

### माइक्रोसॉफ्ट ब्लैकलिस्ट से हटाना

आप अपने डोमेन/आईपी को यहां हटाने के लिए अनुरोध कर सकते हैं: [https://sender.office.com/](https://sender.office.com).

## गोफिश अभियान बनाएं और लॉन्च करें

### भेजने वाले प्रोफ़ाइल

* भेजने वाले प्रोफ़ाइल को पहचानने के लिए कोई **नाम सेट करें**
* फ़िशिंग ईमेल भेजने के लिए किस खाते से भेजने जा रहे हैं यह तय करें। सुझाव: _noreply, support, servicedesk, salesforce..._
* आप उपयोगकर्ता नाम और पासवर्ड खाली छोड़ सकते हैं, लेकिन सुनिश्चित करें कि आपने "सर्टिफिकेट त्रुटियों को अनदेखा करें" की जांच की है

![](<../../.gitbook/assets/image (253) (1) (2) (1) (1) (2) (2) (3) (3) (5) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (
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
ध्यान दें कि **ईमेल के विश्वसनीयता को बढ़ाने के लिए**, सलाह दी जाती है कि आप कुछ साइनेचर का उपयोग करें जो क्लाइंट के ईमेल से हो। सुझाव:

* एक **अस्तित्व नहीं रखने वाले पते** पर एक ईमेल भेजें और देखें कि क्या प्रतिक्रिया में कोई साइनेचर है।
* info@ex.com या press@ex.com या public@ex.com जैसे **सार्वजनिक ईमेल** खोजें और उन्हें एक ईमेल भेजें और प्रतिक्रिया की प्रतीक्षा करें।
* कुछ मान्य खोजे गए ईमेल से संपर्क करने की कोशिश करें और प्रतिक्रिया की प्रतीक्षा करें।

![](<../../.gitbook/assets/image (393).png>)

{% hint style="info" %}
ईमेल टेम्पलेट भी **भेजने के लिए फ़ाइलें संलग्न करने** की अनुमति देता है। यदि आप किसी विशेष रूप से तैयार की गई फ़ाइलें / दस्तावेज़ [इस पृष्ठ](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md) को चुरा लेने के लिए NTLM चुनौतियों को चुराना चाहते हैं।
{% endhint %}

### लैंडिंग पेज

* एक **नाम** लिखें
* वेब पेज का **HTML कोड लिखें**। ध्यान दें कि आप **वेब पेज को आयात** कर सकते हैं।
* **जमा हुए डेटा को कैप्चर** करें और **पासवर्ड को कैप्चर** करें
* एक **पुनर्निर्देशन सेट** करें

![](<../../.gitbook/assets/image (394).png>)

{% hint style="info" %}
आमतौर पर आपको पृष्ठ के HTML कोड को संशोधित करने की आवश्यकता होगी और कुछ परीक्षण स्थानीय में करने की आवश्यकता होगी (शायद किसी Apache सर्वर का उपयोग करके) **जब तक आप परिणामों से संतुष्ट नहीं हो जाते हैं।** फिर, उस HTML कोड को बॉक्स में लिखें।\
ध्यान दें कि यदि आपको HTML के लिए **कुछ स्थिर संसाधनों** का उपयोग करना होता है (शायद कुछ CSS और JS पेजेज) तो आप उन्हें _**/opt/gophish/static/endpoint**_ में सहेज सकते हैं और फिर से उन्हें _**/static/\<filename>**_ से एक्सेस कर सकते हैं।
{% endhint %}

{% hint style="info" %}
पुनर्निर्देशन के लिए आप **उपयोगकर्ताओं को पीडित की वास्तविक मुख्य वेब पृष्ठ** पर रीडायरेक्ट कर सकते हैं, या उन्हें उदाहरण के लिए _/static/migration.html_ पर रीडायरेक्ट करें, 5 सेकंड के लिए कुछ **घूमती चक्कर वाली पहिया** ([**https://loading.io/**](https://loading.io)) लगाएं और फिर इंद्रधनुष की प्रक्रिया सफल रही है इसकी सूचना दें।
{% endhint %}

### उपयोगकर्ता और समूह

* एक नाम सेट करें
* डेटा को **आयात करें** (ध्यान दें कि उदाहरण के लिए आपको प्रत्येक उपयोगकर्ता के पहला नाम, अंतिम नाम और ईमेल पता की आवश्यकता होगी)

![](<../../.gitbook/assets/image (395).png>)

### अभियान

अंत में, एक अभियान बनाएं जिसमें एक नाम, ईमेल टेम्पलेट, लैंडिंग पेज, URL, भेजने का प्रोफ़ाइल और समूह का चयन करें। ध्यान दें कि URL पीडितों को भेजा जाने वाला लिंक होगा

ध्यान दें कि **भेजने का प्रोफ़ाइल आपको अंतिम फ़िशिंग ईमेल की तरह कैसा दिखेगा यह देखने के लिए एक परीक्षण ईमेल भेजने की अनुमति देता है**:

![](<../../.gitbook/assets/image (396).png>)

{% hint style="info" %}
मैं सलाह दूंगा कि आप **टेस्ट ईमेल्स को 10 मिनट मेल पतों पर भेजें** ताकि आपको परीक्षण करने से काले सूची में नहीं डाला जाए।
{% endhint %}

सब कुछ तैयार हो जाने पर, अभियान को चालू करें!

## वेबसाइट क्लोनिंग

यदि किसी कारण से आप वेबसाइट क्लोन करना चाहते हैं, तो निम्नलिखित पृष्ठ की जांच करें:

{% content-ref url="clone-a-website.md" %}
[clone-a-website.md](clone-a-website.md)
{% endcontent-ref %}

## बैकडोर दस्तावेज़ और फ़ाइलें

कुछ फ़िशिंग मूल्यांकनों में (मुख्य रूप से लाल टीमों के लिए) आपको भी **कुछ प्रकार के बैकडोर संबंधी फ़ाइलें भेजने** की आ
## संदर्भ

* [https://zeltser.com/domain-name-variations-in-phishing/](https://zeltser.com/domain-name-variations-in-phishing/)
* [https://0xpatrik.com/phishing-domains/](https://0xpatrik.com/phishing-domains/)
* [https://darkbyte.net/robando-sesiones-y-bypasseando-2fa-con-evilnovnc/](https://darkbyte.net/robando-sesiones-y-bypasseando-2fa-con-evilnovnc/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह,
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>
