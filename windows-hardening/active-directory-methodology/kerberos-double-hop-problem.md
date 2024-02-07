# केरबेरोस डबल हॉप समस्या

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन हैकट्रिक्स में** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड** करना चाहते हैं? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **ट्विटर** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **पालन** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud)।

</details>

## परिचय

केरबेरोस "डबल हॉप" समस्या उत्पन्न होती है जब एक हमलावर दो **हॉप** के माध्यम से **केरबेरोस प्रमाणीकरण का उपयोग** करने का प्रयास करता है, उदाहरण के लिए **PowerShell**/**WinRM** का उपयोग करके।

जब **केरबेरोस** के माध्यम से **प्रमाणीकरण** होता है, **क्रेडेंशियल** **मेमोरी में कैश नहीं** होती हैं। इसलिए, यदि आप mimikatz चलाते हैं तो आपको प्रयोक्ता के क्रेडेंशियल मशीन में नहीं मिलेंगे भले ही वह प्रक्रियाएँ चला रहा हो।

यह इसलिए है क्योंकि केरबेरोस के साथ कनेक्ट करते समय ये कदम होते हैं:

1. प्रयोक्ता1 क्रेडेंशियल प्रदान करता है और **डोमेन कंट्रोलर** प्रयोक्ता1 को एक केरबेरोस **TGT** लौटाता है।
2. प्रयोक्ता1 **TGT** का उपयोग करके **सेवा टिकट** का अनुरोध करता है **सर्वर1** से **कनेक्ट** होने के लिए।
3. प्रयोक्ता1 **सर्वर1** से **कनेक्ट** करता है और **सेवा टिकट** प्रदान करता है।
4. **सर्वर1** में प्रयोक्ता1 के **क्रेडेंशियल** या **TGT** नहीं होते हैं। इसलिए, जब प्रयोक्ता1 सर्वर1 से दूसरे सर्वर में लॉगिन करने का प्रयास करता है, वह **प्रमाणीकरण** नहीं कर पाता है।

### असीमित अनुबंधन

यदि PC में **असीमित अनुबंधन** सक्षम है, तो यह नहीं होगा क्योंकि **सर्वर** को उसे एक भी उपयोगकर्ता का **TGT** मिलेगा जो इसे एक्सेस कर रहा है। इसके अतिरिक्त, यदि असीमित अनुबंधन का उपयोग किया जाता है तो आप शायद इससे **डोमेन कंट्रोलर** को **कंप्रोमाइज** कर सकते हैं।\
[**असीमित अनुबंधन पृष्ठ में अधिक जानकारी**](unconstrained-delegation.md)।

### CredSSP

इस समस्या से बचने के लिए **सिस्टम व्यवस्थापकों** के लिए एक और सुझाव जो [**विशेष रूप से असुरक्षित है**](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/enable-wsmancredssp?view=powershell-7) है **क्रेडेंशियल सुरक्षा समर्थन प्रदाता**। CredSSP को सक्षम करना वर्षों से विभिन्न फोरमों में उल्लेखित समाधान रहा है। माइक्रोसॉफ्ट से:

_“CredSSP प्रमाणीकरण स्थानीय कंप्यूटर से दूरस्थ कंप्यूटर को उपयोगकर्ता क्रेडेंशियल अनुप्रेषित करता है। यह अभ्यास दूरस्थ संचालन का सुरक्षा जोखिम बढ़ाता है। यदि दूरस्थ कंप्यूटर को कंप्रोमाइज़ किया जाता है, जब क्रेडेंशियल उसे पारित किए जाते हैं, तो क्रेडेंशियल का उपयोग नेटवर्क सत्र को नियंत्रित करने के लिए किया जा सकता है।”_

यदि उत्पादन सिस्टमों, संवेदनशील नेटवर्क आदि पर **CredSSP सक्षम** पाए जाते हैं, तो सुझाव दिया जाता है कि उन्हें अक्षम किया जाए। CredSSP स्थिति की जांच करने का एक त्वरित तरीका है `Get-WSManCredSSP` चलाकर। जो दूरस्थ से निष्क्रिय किया जा सकता है यदि WinRM सक्षम है।
```powershell
Invoke-Command -ComputerName bizintel -Credential ta\redsuit -ScriptBlock {
Get-WSManCredSSP
}
```
## कार्यारूपण

### आवाज़ कमांड <a href="#invoke-command" id="invoke-command"></a>

यह विधि दोहरी हॉप समस्या के साथ काम करने की तरह है, जिसे आवश्यकता नहीं है। इसमें कोई विन्यास नहीं है, और आप इसे आसानी से अपने हमले वाले बॉक्स से चला सकते हैं। यह मूल रूप से एक **नेस्टेड `Invoke-Command`** है।

यह **दूसरे सर्वर पर `hostname`** चलाएगा:
```powershell
$cred = Get-Credential ta\redsuit
Invoke-Command -ComputerName bizintel -Credential $cred -ScriptBlock {
Invoke-Command -ComputerName secdev -Credential $cred -ScriptBlock {hostname}
}
```
आप एक **PS-Session** को **पहले सर्वर** के साथ स्थापित कर सकते हैं और बस वहां से **`Invoke-Command`** को `$cred` के साथ **चला** सकते हैं इसे नेस्ट करने की बजाय। हालांकि, अपने हमलावार बॉक्स से इसे चलाना कार्यक्षमता को केंद्रीकृत करता है:
```powershell
# From the WinRM connection
$pwd = ConvertTo-SecureString 'uiefgyvef$/E3' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('DOMAIN\username', $pwd)
# Use "-Credential $cred" option in Powerview commands
```
### रजिस्टर PSSession Configuration

यदि **`evil-winrm`** का उपयोग न करके आप **`Enter-PSSession`** cmdlet का उपयोग कर सकते हैं तो आप फिर **`Register-PSSessionConfiguration`** का उपयोग करके डबल हॉप समस्या को अनदेखा करने के लिए पुनः कनेक्ट कर सकते हैं:
```powershell
# Register a new PS Session configuration
Register-PSSessionConfiguration -Name doublehopsess -RunAsCredential domain_name\username
# Restar WinRM
Restart-Service WinRM
# Get a PSSession
Enter-PSSession -ConfigurationName doublehopsess -ComputerName <pc_name> -Credential domain_name\username
# Check that in this case the TGT was sent and is in memory of the PSSession
klist
# In this session you won't have the double hop problem anymore
```
### पोर्ट फॉरवर्डिंग <a href="#portproxy" id="portproxy"></a>

क्योंकि हमारे पास इंटरमीडिएट टारगेट **bizintel: 10.35.8.17** पर स्थानीय प्रशासक है, आप एक पोर्ट फॉरवर्डिंग नियम जोड़ सकते हैं ताकि आपके अनुरोध अंतिम/तीसरे सर्वर **secdev: 10.35.8.23** पर भेजे जा सकें।

**netsh** का उपयोग करके आप एक वन-लाइनर निकालकर नियम जोड़ सकते हैं।
```bash
netsh interface portproxy add v4tov4 listenport=5446 listenaddress=10.35.8.17 connectport=5985 connectaddress=10.35.8.23
```
इसलिए **पहला सर्वर** पोर्ट 5446 पर सुन रहा है और 5446 पर पहुंचने वाले अनुरोधों को **दूसरे सर्वर** पोर्ट 5985 (यानी WinRM) पर फॉरवर्ड करेगा।

फिर Windows फ़ायरवॉल में एक होल बनाएं, जिसे एक तेज netsh one-liner के साथ भी किया जा सकता है।
```bash
netsh advfirewall firewall add rule name=fwd dir=in action=allow protocol=TCP localport=5446
```
अब सत्र स्थापित करें, जो हमें **पहले सर्वर** की ओर अग्रेषित करेगा।

<figure><img src="../../.gitbook/assets/image (3) (5) (1).png" alt=""><figcaption></figcaption></figure>

#### winrs.exe <a href="#winrsexe" id="winrsexe"></a>

**Portforwarding WinRM** अनुरोधों को भी लगता है कि यह काम करता है जब **`winrs.exe`** का उपयोग किया जाता है। यदि आपको यह पता है कि PowerShell को मॉनिटर किया जा रहा है, तो यह एक बेहतर विकल्प हो सकता है। नीचे दिए गए कमांड से `hostname` के रूप में “**secdev**” लौटाता है।
```bash
winrs -r:http://bizintel:5446 -u:ta\redsuit -p:2600leet hostname
```
### OpenSSH <a href="#openssh" id="openssh"></a>

इस तकनीक के लिए [OpenSSH इंस्टॉल करना आवश्यक है](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH) पहले सर्वर बॉक्स पर। Windows के लिए OpenSSH इंस्टॉल करना **पूरी तरह से CLI के माध्यम से** किया जा सकता है और यह बहुत कम समय लेता है - और यह मैलवेयर के रूप में फ्लैग नहीं करता!

यह स्थितियों में संभावना है कि यह संभव नहीं हो सकता है, बहुत ज्यादा कठिन हो सकता है या सामान्य OpSec जोखिम हो सकता है।

यह तकनीक एक जंप बॉक्स सेटअप पर विशेष रूप से उपयोगी हो सकती है - जिसमें एक अन्यथा पहुंच योग्य नेटवर्क तक पहुंच हो। एक बार SSH कनेक्शन स्थापित हो जाता है, उपयोगकर्ता/हमलावर डबल-हॉप समस्या में बिना धमाके के सेगमेंटेड नेटवर्क के खिलाफ जितने भी `New-PSSession` की आवश्यकता हो, उन्हें आग लगा सकता है।

जब OpenSSH में **पासवर्ड प्रमाणीकरण** का उपयोग किया जाता है (कुंजियों या कर्बेरोस नहीं), तो **लॉगऑन प्रकार 8** यानी _नेटवर्क क्लियर पाठ लॉगऑन_ होता है। यह यह नहीं मानना चाहिए कि आपका पासवर्ड साफ़ पाठ में भेजा जाता है - यह वास्तव में SSH द्वारा एन्क्रिप्ट किया जाता है। आपके सत्र के लिए यह आगमन के समय अपने [प्रमाणीकरण पैकेज](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-logonusera?redirectedfrom=MSDN) के माध्यम से स्पष्ट पाठ में अनएन्क्रिप्ट हो जाता है ताकि आपका सत्र और अधिक सर्वरों के लिए जूसी TGT का अनुरोध कर सके!

इससे अंतरवर्ती सर्वर आपके पक्ष में एक TGT का अनुरोध कर सकता है और स्थानीय रूप से इसे स्टोर कर सकता है। आपका सत्र फिर इस TGT का उपयोग कर सकता है अधिक सर्वरों के प्रमाणीकरण(PS रिमोट) के लिए।

#### OpenSSH इंस्टॉल स्थिति

नवीनतम [OpenSSH रिलीज़ ज़िप डाउनलोड करें github से](https://github.com/PowerShell/Win32-OpenSSH/releases) और इसे अपने हमलावर बॉक्स पर ले जाएं और इसे वहाँ ले जाएं (या सीधे जंप बॉक्स पर डाउनलोड करें)।

जहाँ चाहें वहाँ ज़िप को अनज़िप करें। फिर, इंस्टॉल स्क्रिप्ट चलाएं - `Install-sshd.ps1`

अंततः, बस फायरवॉल नियम जोड़ें **पोर्ट 22 खोलने** के लिए। सुनिश्चित करें कि SSH सेवाएं इंस्टॉल की गई हैं, और उन्हें शुरू करें। ये दोनों सेवाएं SSH काम करने के लिए चालू होनी चाहिए।

यदि आपको `Connection reset` त्रुटि मिलती है, तो अनुमतियाँ अपडेट करें ताकि **सभी: पढ़ें और क्रियान्वित करें** रूट OpenSSH निर्देशिका पर।
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
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को PR जमा करके।**

</details>
