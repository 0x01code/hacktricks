# कर्बेरोस डबल हॉप समस्या

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटीज़**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को PR जमा करके**.

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## परिचय

कर्बेरोस "डबल हॉप" समस्या उत्पन्न होती है जब एक हमलावर को **कर्बेरोस प्रमाणीकरण का उपयोग करके दो हॉप** के माध्यम से कोशिश करता है, उदाहरण के लिए **PowerShell**/**WinRM** का उपयोग करके।

जब **कर्बेरोस** के माध्यम से **प्रमाणीकरण** होता है, **क्रेडेंशियल** **मेमोरी में कैश नहीं होती** हैं। इसलिए, यदि आप mimikatz चलाते हैं तो आपको प्रयोक्ता के क्रेडेंशियल मशीन में नहीं मिलेंगे भले ही वह प्रक्रियाएँ चला रहा हो।

यह इसलिए है क्योंकि कर्बेरोस के साथ कनेक्ट करते समय ये कदम होते हैं:

1. प्रयोक्ता1 क्रेडेंशियल प्रदान करता है और **डोमेन कंट्रोलर** प्रयोक्ता1 को एक कर्बेरोस **TGT** लौटाता है।
2. प्रयोक्ता1 **TGT** का उपयोग करके **सेवा टिकट** का अनुरोध करता है **सर्वर1** से **कनेक्ट** करने के लिए।
3. प्रयोक्ता1 **सर्वर1** से **कनेक्ट** करता है और **सेवा टिकट** प्रदान करता है।
4. **सर्वर1** में प्रयोक्ता1 के **क्रेडेंशियल** या **TGT** कैश नहीं हैं। इसलिए, जब प्रयोक्ता1 सर्वर1 से दूसरे सर्वर में लॉगिन करने की कोशिश करता है, वह **प्रमाणीकरण करने में सक्षम नहीं होता** है।

### असीमित अधिकार

यदि PC में **असीमित अधिकार** सक्षम है, तो यह नहीं होगा क्योंकि **सर्वर** को उसे एक उपयोगकर्ता का **TGT** प्राप्त होगा जो इसे एक्सेस कर रहा है। इसके अतिरिक्त, यदि असीमित अधिकार का उपयोग किया जाता है तो आप शायद इससे **डोमेन कंट्रोलर को कंप्रोमाइज** कर सकते हैं।\
[**असीमित अधिकार पृष्ठ में अधिक जानकारी**](unconstrained-delegation.md).

### CredSSP

इस समस्या से बचने का एक और तरीका जो [**विशेष रूप से असुरक्षित है**](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/enable-wsmancredssp?view=powershell-7) है **Credential Security Support Provider**। माइक्रोसॉफ्ट से:

> CredSSP प्रमाणीकरण स्थानीय कंप्यूटर से रिमोट कंप्यूटर को उपयोगकर्ता क्रेडेंशियल अनुप्रेषित करता है। यह अभ्यास रिमोट संचालन का सुरक्षा जोखिम बढ़ाता है। यदि रिमोट कंप्यूटर को कंप्रोमाइज़ किया जाता है, जब क्रेडेंशियल उसे पास किए जाते हैं, तो उन क्रेडेंशियल का नेटवर्क सत्र नियंत्रण करने के लिए उपयोग किया जा सकता है।

उत्पादन सिस्टम, संवेदनशील नेटवर्क और समान परिवेशों में सुरक्षा संबंधित चिंताओं के कारण **CredSSP** को अक्षम करना अत्यंत सिफारिश किया जाता है। **CredSSP** सक्षम है या नहीं, यह निर्धारित करने के लिए, `Get-WSManCredSSP` कमांड चलाया जा सकता है। यह कमांड **CredSSP स्थिति की जांच** करने की अनुमति देता है और यहाँ तक कि यह दूरस्थ से निष्पादित किया जा सकता है, प्रावधान है कि **WinRM** सक्षम हो।
```powershell
Invoke-Command -ComputerName bizintel -Credential ta\redsuit -ScriptBlock {
Get-WSManCredSSP
}
```
## उपाय

### आवाहन कमांड

डबल हॉप समस्या का समाधान करने के लिए, एक nested `Invoke-Command` का एक तरीका प्रस्तुत किया गया है। यह समस्या सीधे हल नहीं करता है लेकिन विशेष विन्यास की आवश्यकता न होने पर एक उपाय प्रदान करता है। यह दृष्टिकोण एक कमांड (`hostname`) को एक द्वितीय सर्वर पर क्रियान्वित करने की अनुमति देता है जो एक पहले हमले वाली मशीन से या पहले सर्वर के साथ पहले स्थापित PS-Session के माध्यम से एक्जीक्यूट किया जा सकता है। यहाँ देखें कैसे किया जाता है:
```powershell
$cred = Get-Credential ta\redsuit
Invoke-Command -ComputerName bizintel -Credential $cred -ScriptBlock {
Invoke-Command -ComputerName secdev -Credential $cred -ScriptBlock {hostname}
}
```
### पीएससत्री कॉन्फ़िगरेशन पंजीकरण

डबल हॉप समस्या को अनदेखा करने का एक समाधान `Register-PSSessionConfiguration` का उपयोग `Enter-PSSession` के साथ करना है। यह विधि `evil-winrm` से भिन्न दृष्टिकोण आवश्यक करती है और डबल हॉप सीमा से पीड़ित नहीं होने वाली सत्र की अनुमति देती है।
```powershell
Register-PSSessionConfiguration -Name doublehopsess -RunAsCredential domain_name\username
Restart-Service WinRM
Enter-PSSession -ConfigurationName doublehopsess -ComputerName <pc_name> -Credential domain_name\username
klist
```
### पोर्ट फॉरवर्डिंग

एक बाध्य लक्ष्य पर स्थानीय प्रशासकों के लिए, पोर्ट फॉरवर्डिंग अंतिम सर्वर को भेजने की अनुरोधों को भेजने की अनुमति देता है। `netsh` का उपयोग करके, एक नियम पोर्ट फॉरवर्डिंग के लिए जोड़ा जा सकता है, साथ ही एक Windows फ़ायरवॉल नियम भी जोड़ा जा सकता है जिससे फॉरवर्ड किए गए पोर्ट को अनुमति दी जा सकती है।
```bash
netsh interface portproxy add v4tov4 listenport=5446 listenaddress=10.35.8.17 connectport=5985 connectaddress=10.35.8.23
netsh advfirewall firewall add rule name=fwd dir=in action=allow protocol=TCP localport=5446
```
#### winrs.exe

`winrs.exe` का उपयोग WinRM अनुरोधों को आगे भेजने के लिए किया जा सकता है, संभावित रूप से एक कम संवेदनशील विकल्प अगर PowerShell मॉनिटरिंग एक चिंता है। नीचे दिए गए कमांड का उपयोग दिखाता है:
```bash
winrs -r:http://bizintel:5446 -u:ta\redsuit -p:2600leet hostname
```
### OpenSSH

पहले सर्वर पर OpenSSH इंस्टॉल करने से डबल हॉप समस्या के लिए एक workaround सक्षम होता है, जो जंप बॉक्स स्थितियों के लिए विशेष रूप से उपयोगी है। इस विधि में CLI इंस्टॉलेशन और Windows के लिए OpenSSH की सेटअप की आवश्यकता होती है। जब पासवर्ड प्रमाणीकरण के लिए कॉन्फ़िगर किया जाता है, तो इससे अंतरिक्ष सर्वर को उपयोगकर्ता के पक्ष में एक TGT प्राप्त करने की अनुमति होती है।

#### OpenSSH इंस्टॉलेशन कदम

1. नवीनतम OpenSSH रिलीज़ ज़िप डाउनलोड करें और टारगेट सर्वर पर ले जाएं।
2. ज़िप फ़ाइल को अनज़िप करें और `Install-sshd.ps1` स्क्रिप्ट चलाएं।
3. पोर्ट 22 खोलने के लिए फ़ायरवॉल नियम जोड़ें और सत्यापित करें कि SSH सेवाएँ चल रही हैं।

`Connection reset` त्रुटियों को हल करने के लिए, अनुमतियों को अपडेट करने की आवश्यकता हो सकती है ताकि OpenSSH निर्देशिका पर सभी को पढ़ने और क्रियान्वित करने की पहुँच हो।
```bash
icacls.exe "C:\Users\redsuit\Documents\ssh\OpenSSH-Win64" /grant Everyone:RX /T
```
## संदर्भ

* [https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20)
* [https://posts.slayerlabs.com/double-hop/](https://posts.slayerlabs.com/double-hop/)
* [https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting](https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting)
* [https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/](https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/)

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**द पीएएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**एनएफटीज़**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक पीएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)
* **जुड़ें** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या मुझे **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें द्वारा पीआर जमा करके** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud).

</details>
