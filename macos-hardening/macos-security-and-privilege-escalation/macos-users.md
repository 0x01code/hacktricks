# macOS उपयोगकर्ता

{% hint style="success" %}
**AWS हैकिंग सीखें और अभ्यास करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और अभ्यास करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** को [**पीआर जमा करके**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** से प्रेरित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि जांच सकें कि क्या कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर्स** द्वारा **क्षति** पहुंचाई गई है।

WhiteIntel का मुख्य उद्देश्य खाता हासिल करने और रैंसमवेयर हमलों से लड़ना है जो जानकारी चोरी करने वाले मैलवेयर से होते हैं।

आप उनकी वेबसाइट चेक कर सकते हैं और **मुफ्त** में उनका इंजन प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

***

### सामान्य उपयोगकर्ता

*   **डेमन**: सिस्टम डेमन्स के लिए रिजर्व उपयोगकर्ता। डिफ़ॉल्ट डेमन खाता नाम आम तौर पर "\_" से शुरू होते हैं:

```bash
_amavisd, _analyticsd, _appinstalld, _appleevents, _applepay, _appowner, _appserver, _appstore, _ard, _assetcache, _astris, _atsserver, _avbdeviced, _calendar, _captiveagent, _ces, _clamav, _cmiodalassistants, _coreaudiod, _coremediaiod, _coreml, _ctkd, _cvmsroot, _cvs, _cyrus, _datadetectors, _demod, _devdocs, _devicemgr, _diskimagesiod, _displaypolicyd, _distnote, _dovecot, _dovenull, _dpaudio, _driverkit, _eppc, _findmydevice, _fpsd, _ftp, _fud, _gamecontrollerd, _geod, _hidd, _iconservices, _installassistant, _installcoordinationd, _installer, _jabber, _kadmin_admin, _kadmin_changepw, _knowledgegraphd, _krb_anonymous, _krb_changepw, _krb_kadmin, _krb_kerberos, _krb_krbtgt, _krbfast, _krbtgt, _launchservicesd, _lda, _locationd, _logd, _lp, _mailman, _mbsetupuser, _mcxalr, _mdnsresponder, _mobileasset, _mysql, _nearbyd, _netbios, _netstatistics, _networkd, _nsurlsessiond, _nsurlstoraged, _oahd, _ondemand, _postfix, _postgres, _qtss, _reportmemoryexception, _rmd, _sandbox, _screensaver, _scsd, _securityagent, _softwareupdate, _spotlight, _sshd, _svn, _taskgated, _teamsserver, _timed, _timezone, _tokend, _trustd, _trustevaluationagent, _unknown, _update_sharing, _usbmuxd, _uucp, _warmd, _webauthserver, _windowserver, _www, _wwwproxy, _xserverdocs
```
* **मेहमान**: बहुत सख्त अनुमतियों वाले मेहमानों के लिए खाता

{% code overflow="wrap" %}
```bash
state=("automaticTime" "afpGuestAccess" "filesystem" "guestAccount" "smbGuestAccess")
for i in "${state[@]}"; do sysadminctl -"${i}" status; done;
```
{% endcode %}

* **कोई नहीं**: प्रक्रियाएँ इस उपयोगकर्ता के साथ कार्यान्वित होती हैं जब न्यूनतम अनुमतियाँ आवश्यक होती हैं
* **रूट**

### उपयोगकर्ता विशेषाधिकार

* **मानक उपयोगकर्ता:** सबसे मौलिक उपयोगकर्ता। इस उपयोगकर्ता को सॉफ़्टवेयर इंस्टॉल करने या अन्य उन्नत कार्यों को करने की कोशिश करते समय एडमिन उपयोगकर्ता से अनुमतियाँ प्रदान की जाती हैं। वे इसे अपने आप नहीं कर सकते।
* **एडमिन उपयोगकर्ता**: एक उपयोगकर्ता जो अधिकांश समय मानक उपयोगकर्ता के रूप में कार्य करता है लेकिन साथ ही रूट कार्रवाई जैसे सॉफ़्टवेयर इंस्टॉल करने और अन्य प्रशासनिक कार्य करने की अनुमति होती है। सभी उपयोगकर्ता जो एडमिन समूह में शामिल हैं, **सुडोएर्स फ़ाइल के माध्यम से रूट तक पहुँच दी जाती हैं**।
* **रूट**: रूट एक उपयोगकर्ता है जिसे लगभग किसी भी कार्रवाई करने की अनुमति है (सिस्टम इंटेग्रिटी संरक्षण जैसे सुरक्षा द्वारा प्रतिबंध लगाए गए हो सकते हैं)।
* उदाहरण के लिए रूट `/System` के अंदर एक फ़ाइल नहीं रख सकेगा।

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
