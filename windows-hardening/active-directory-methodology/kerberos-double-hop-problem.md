# केरबेरोस डबल हॉप समस्या

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में**।

</details>

## परिचय

केरबेरोस "डबल हॉप" समस्या उत्पन्न होती है जब एक हमलावर को **केरबेरोस प्रमाणीकरण का उपयोग करके दो** **हॉप** के बीच, उदाहरण के लिए **PowerShell**/**WinRM** का उपयोग करते हैं।

जब **केरबेरोस** के माध्यम से **प्रमाणीकरण** होता है, **क्रेडेंशियल्स** **मेमोरी में कैश नहीं होते** हैं। इसलिए, यदि आप mimikatz चलाते हैं तो आपको मशीन में उपयोगकर्ता के क्रेडेंशियल्स नहीं मिलेंगे भले ही वह प्रक्रियाएँ चला रहा हो।

यह इसलिए है क्योंकि केरबेरोस के साथ कनेक्ट करते समय ये कदम होते हैं:

1. उपयोगकर्ता1 क्रेडेंशियल्स प्रदान करता है और **डोमेन कंट्रोलर** उपयोगकर्ता1 को एक केरबेरोस **TGT** लौटाता है।
2. उपयोगकर्ता1 **TGT** का उपयोग करके **सेवा टिकट** का अनुरोध करता है **सर्वर1** से **कनेक्ट** होने के लिए।
3. उपयोगकर्ता1 **सर्वर1** से **कनेक्ट** करता है और **सेवा टिकट** प्रदान करता है।
4. **सर्वर1** में **क्रेडेंशियल्स** या उपयोगकर्ता1 का **TGT** कैश नहीं है। इसलिए, जब उपयोगकर्ता1 सर्वर1 से दूसरे सर्वर में लॉगिन करने का प्रयास करता है, तो वह **प्रमाणीकरण** नहीं कर पाता है।

### असीमित डेलीगेशन

यदि **असीमित डेलीगेशन** पीसी में सक्षम है, तो यह नहीं होगा क्योंकि **सर्वर** को उसे एक उपयोगकर्ता का **TGT** प्राप्त होगा जो उसका उपयोग कर रहा है। इसके अतिरिक्त, यदि असीमित डेलीगेशन का उपयोग किया जाता है तो आप शायद इससे **डोमेन कंट्रोलर** को कंप्रोमाइज कर सकते हैं।\
[**असीमित डेलीगेशन पृष्ठ में अधिक जानकारी**](unconstrained-delegation.md)।

### CredSSP

इस समस्या से बचने का एक और तरीका जो [**विशेष रूप से असुरक्षित है**](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/enable-wsmancredssp?view=powershell-7) है **क्रेडेंशियल सुरक्षा समर्थन प्रदाता**। माइक्रोसॉफ्ट से:

> CredSSP प्रमाणीकरण स्थानीय कंप्यूटर से दूरस्थ कंप्यूटर को उपयोगकर्ता क्रेडेंशियल्स को सौंपता है। यह अभ्यास दूरस्थ संचालन का सुरक्षा जोखिम बढ़ाता है। यदि दूरस्थ कंप्यूटर को कंप्रोमाइज़ किया जाता है, जब क्रेडेंशियल्स उसे पारित किए जाते हैं, तो क्रेडेंशियल्स का नेटवर्क सत्र को नियंत्रित करने के लिए उपयोग किया जा सकता है।

सुरक्षा संबंधित चिंताओं के कारण संयंत्रणीय प्रणालियों, संवेदनशील नेटवर्कों और समान परिवेशों पर **CredSSP** को प्रोडक्शन सिस्टम्स पर अक्षम करने की अधिक सिफारिश की जाती है। **CredSSP** सक्षम है या नहीं यह निर्धारित करने के लिए, `Get-WSManCredSSP` कमांड चलाया जा सकता है। यह कमांड **CredSSP स्थिति की जांच** करने की अनुमति देता है और यहाँ तक कि यह दूरस्थ से भी निष्पादित किया जा सकता है, प्राविधिक **WinRM** सक्षम हो।
```powershell
Invoke-Command -ComputerName bizintel -Credential ta\redsuit -ScriptBlock {
Get-WSManCredSSP
}
```
## उपाय

### आवाज़ कमांड

डबल हॉप समस्या का समाधान करने के लिए, एक nested `Invoke-Command` का एक तरीका प्रस्तुत किया गया है। यह समस्या को सीधे हल नहीं करता है लेकिन विशेष विन्यास की आवश्यकता न होने पर एक उपाय प्रदान करता है। यह दृष्टिकोण एक कमांड (`hostname`) को एक सेकेंडरी सर्वर पर एक्सीक्यूट करने की अनुमति देता है जो एक पहले हमले वाली मशीन से एक्सीक्यूट किया गया पावरशेल कमांड या पहले सर्वर के साथ पहले स्थापित PS-Session के माध्यम से किया जा सकता है। यहाँ देखें कि यह कैसे किया जाता है:
```powershell
$cred = Get-Credential ta\redsuit
Invoke-Command -ComputerName bizintel -Credential $cred -ScriptBlock {
Invoke-Command -ComputerName secdev -Credential $cred -ScriptBlock {hostname}
}
```
### पीएस-सत्र को स्थापित करना

डबल हॉप समस्या को अनदेखा करने का एक समाधान `Register-PSSessionConfiguration` का उपयोग `Enter-PSSession` के साथ करना है। यह विधि `evil-winrm` से भिन्न दृष्टिकोण आवश्यक करती है और डबल हॉप सीमा से प्रभावित नहीं होने वाला एक सत्र करने की अनुमति देती है।
```powershell
Register-PSSessionConfiguration -Name doublehopsess -RunAsCredential domain_name\username
Restart-Service WinRM
Enter-PSSession -ConfigurationName doublehopsess -ComputerName <pc_name> -Credential domain_name\username
klist
```
### पोर्ट फॉरवर्डिंग

एक बाध्य लक्ष्य पर स्थानीय प्रशासकों के लिए, पोर्ट फॉरवर्डिंग अंतिम सर्वर को भेजने की अनुमति देता है। `netsh` का उपयोग करके, पोर्ट फॉरवर्डिंग के लिए एक नियम जोड़ा जा सकता है, साथ ही एक Windows फ़ायरवॉल नियम भी जोड़ा जा सकता है जो फॉरवर्ड किए गए पोर्ट को अनुमति देता है।
```bash
netsh interface portproxy add v4tov4 listenport=5446 listenaddress=10.35.8.17 connectport=5985 connectaddress=10.35.8.23
netsh advfirewall firewall add rule name=fwd dir=in action=allow protocol=TCP localport=5446
```
#### winrs.exe

`winrs.exe` का उपयोग WinRM अनुरोधों को आगे भेजने के लिए किया जा सकता है, यदि PowerShell मॉनिटरिंग एक चिंता है तो यह एक कम जांचने योग्य विकल्प हो सकता है। नीचे दिए गए कमांड का उपयोग इसका प्रदर्शन करता है:
```bash
winrs -r:http://bizintel:5446 -u:ta\redsuit -p:2600leet hostname
```
### OpenSSH

पहले सर्वर पर OpenSSH इंस्टॉल करने से डबल हॉप समस्या का एक workaround सक्रिय हो जाता है, जो जंप बॉक्स स्थितियों के लिए विशेष रूप से उपयोगी है। इस विधि में CLI इंस्टॉलेशन और Windows के लिए OpenSSH की सेटअप की आवश्यकता होती है। जब Password Authentication के लिए कॉन्फ़िगर किया जाता है, तो इससे अंतरिक्ष सर्वर को उपयोगकर्ता के पक्ष में एक TGT प्राप्त करने की अनुमति होती है।

#### OpenSSH इंस्टॉलेशन कदम

1. नवीनतम OpenSSH रिलीज़ ज़िप डाउनलोड करें और टारगेट सर्वर पर ले जाएं।
2. ज़िप फ़ाइल को अनज़िप करें और `Install-sshd.ps1` स्क्रिप्ट चलाएं।
3. पोर्ट 22 खोलने के लिए फ़ायरवॉल नियम जोड़ें और SSH सेवाएँ चल रही हैं या नहीं यह सत्यापित करें।

`Connection reset` त्रुटियों को हल करने के लिए, अनुमतियों को अपडेट करने की आवश्यकता हो सकती है ताकि OpenSSH निर्देशिका पर सभी को पढ़ने और क्रियान्वित करने की पहुँच हो।
```bash
icacls.exe "C:\Users\redsuit\Documents\ssh\OpenSSH-Win64" /grant Everyone:RX /T
```
## संदर्भ

* [https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20)
* [https://posts.slayerlabs.com/double-hop/](https://posts.slayerlabs.com/double-hop/)
* [https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting](https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting)
* [https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/](https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को पीआर जमा करके।**

</details>
