# बेसिक पायथन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## पायथन की मूल बातें

### उपयोगी जानकारी

list(xrange()) == range() --> पायथन3 में range पायथन2 का xrange है (यह एक सूचकांक नहीं है बल्कि एक जेनरेटर है)\
ट्यूपल और सूची के बीच अंतर यह है कि ट्यूपल में एक मान की स्थिति का मतलब होता है लेकिन सूचियों में केवल क्रमबद्ध मान होते हैं। ट्यूपल में संरचनाएं होती हैं लेकिन सूचियों में एक क्रम होता है।

### मुख्य आपरेशन

एक संख्या को उठाने के लिए आप इस्तेमाल करते हैं: 3\*\*2 (3^2 नहीं)\
यदि आप 2/3 करते हैं तो यह 1 लौटाता है क्योंकि आप दो पूर्णांकों (integers) को विभाजित कर रहे हैं। यदि आप दशमलव चाहते हैं तो आपको दशमलव विभाजित करना चाहिए (2.0/3.0)।\
i >= j\
i <= j\
i == j\
i != j\
a and b\
a or b\
not a\
float(a)\
int(a)\
str(d)\
ord("A") = 65\
chr(65) = 'A'\
hex(100) = '0x64'\
hex(100)\[2:] = '64'\
isinstance(1, int) = True\
"a b".split(" ") = \['a', 'b']\
" ".join(\['a', 'b']) = "a b"\
"abcdef".startswith("ab") = True\
"abcdef".contains("abc") = True\
"abc\n".strip() = "abc"\
"apbc".replace("p","") = "abc"\
dir(str) = सभी उपलब्ध विधियों की सूची\
help(str) = वर्ग str की परिभाषा\
"a".upper() = "A"\
"A".lower() = "a"\
"abc".capitalize() = "Abc"\
sum(\[1,2,3]) = 6\
sorted(\[1,43,5,3,21,4])

**अक्षरों को जोड़ें**\
3 \* ’a’ = ‘aaa’\
‘a’ + ‘b’ = ‘ab’\
‘a’ + str(3) = ‘a3’\
\[1,2,3]+\[4,5]=\[1,2,3,4,5]

**सूची के भाग**\
‘abc’\[0] = ‘a’\
'abc’\[-1] = ‘c’\
'abc’\[1:3] = ‘bc’ from \[1] to \[2]\
"qwertyuiop"\[:-1] = 'qwertyuio'

**टिप्पणियाँ**\
\# एक पंक्ति की टिप्पणी\
"""\
कई पंक्तियों की टिप्पणी\
एक और\
"""

**लूप**
```
if a:
#somethig
elif b:
#something
else:
#something

while(a):
#comething

for i in range(0,100):
#something from 0 to 99

for letter in "hola":
#something with a letter in "hola"
```
### टपल

t1 = (1, '2', 'तीन')\
t2 = (5, 6)\
t3 = t1 + t2 = (1, '2', 'तीन', 5, 6)\
(4,) = सिंगलटन\
d = () खाली टपल\
d += (4,) --> टपल में जोड़ना\
CANT! --> t1\[1] == 'नया मान'\
list(t2) = \[5, 6] --> टपल से सूची में

### सूची (एरे)

d = \[] खाली\
a = \[1, 2, 3]\
b = \[4, 5]\
a + b = \[1, 2, 3, 4, 5]\
b.append(6) = \[4, 5, 6]\
tuple(a) = (1, 2, 3) --> सूची से टपल में

### शब्दकोश

d = {} खाली\
monthNumbers = {1: 'जनवरी', 2: 'फ़रवरी', 'फ़रवरी': 2} --> monthNumbers -> {1: 'जनवरी', 2: 'फ़रवरी', 'फ़रवरी': 2}\
monthNumbers\[1] = 'जनवरी'\
monthNumbers\[‘फ़रवरी’] = 2\
list(monthNumbers) = \[1, 2, 'फ़रवरी']\
monthNumbers.values() = \['जनवरी', 'फ़रवरी', 2]\
keys = \[k for k in monthNumbers]\
a = {'9': 9}\
monthNumbers.update(a) = {'9': 9, 1: 'जनवरी', 2: 'फ़रवरी', 'फ़रवरी': 2}\
mN = monthNumbers.copy() #अपेक्षाकृत प्रतिलिपि\
monthNumbers.get('key', 0) #जांचें कि कुंजी मौजूद है, monthNumbers\["key"] का मान लौटाएं या 0 अगर यह मौजूद नहीं है

### सेट

सेट में पुनरावृत्ति नहीं होती है\
myset = set(\['ए', 'बी']) = {'ए', 'बी'}\
myset.add('सी') = {'ए', 'बी', 'सी'}\
myset.add('ए') = {'ए', 'बी', 'सी'} #पुनरावृत्ति नहीं होती है\
myset.update(\[1, 2, 3]) = set(\['ए', 1, 2, 'बी', 'सी', 3])\
myset.discard(10) #यदि मौजूद है, तो हटाएं, यदि नहीं है, तो कुछ नहीं\
myset.remove(10) #यदि मौजूद है, तो हटाएं, यदि नहीं है, तो अपवाद उठाएं\
myset2 = set(\[1, 2, 3, 4])\
myset.union(myset2) #myset या myset2 में मौजूद मान\
myset.intersection(myset2) #myset और myset2 में मौजूद मान\
myset.difference(myset2) #myset में मौजूद मान, लेकिन myset2 में नहीं\
myset.symmetric\_difference(myset2) #myset और myset2 में मौजूद नहीं होने वाले मान (दोनों में नहीं)\
myset.pop() #सेट का पहला तत्व प्राप्त करें और हटाएं\
myset.intersection\_update(myset2) #myset = myset और myset2 दोनों में मौजूद तत्व\
myset.difference\_update(myset2) #myset = myset में मौजूद तत्व, लेकिन myset2 में नहीं\
myset.symmetric\_difference\_update(myset2) #myset = दोनों में मौजूद नहीं होने वाले तत्व

### कक्षाएँ

\_\_It\_\_ में विधि उपयोग की जाएगी जो सॉर्ट करने के लिए इस कक्षा के एक ऑब्जेक्ट से बड़ा हैं या नहीं
```python
class Person(name):
def __init__(self,name):
self.name= name
self.lastName = name.split(‘ ‘)[-1]
self.birthday = None
def __It__(self, other):
if self.lastName == other.lastName:
return self.name < other.name
return self.lastName < other.lastName #Return True if the lastname is smaller

def setBirthday(self, month, day. year):
self.birthday = date tame.date(year,month,day)
def getAge(self):
return (date time.date.today() - self.birthday).days


class MITPerson(Person):
nextIdNum = 0	# Attribute of the Class
def __init__(self, name):
Person.__init__(self,name)
self.idNum = MITPerson.nextIdNum  —> Accedemos al atributo de la clase
MITPerson.nextIdNum += 1 #Attribute of the class +1

def __it__(self, other):
return self.idNum < other.idNum
```
### map, zip, filter, lambda, sorted और one-liners

**Map** की तरह है: \[f(x) for x in iterable] --> map(tutple,\[a,b]) = \[(1,2,3),(4,5)]\
m = map(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) --> \[False, False, True, False, False, True, False, False, True]

**zip** तब रुकता है जब foo या bar में से छोटा हो जाता है:
```
for f, b in zip(foo, bar):
print(f, b)
```
**लैम्बडा** का उपयोग एक फ़ंक्शन को परिभाषित करने के लिए किया जाता है\
(lambda x,y: x+y)(5,3) = 8 --> लैम्बडा का उपयोग एक साधारण **फ़ंक्शन** के रूप में करें\
**sorted**(range(-5,6), key=lambda x: x\*\* 2) = \[0, -1, 1, -2, 2, -3, 3, -4, 4, -5, 5] --> लैम्बडा का उपयोग एक सूची को क्रमबद्ध करने के लिए करें\
m = **filter**(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) = \[3, 6, 9] --> लैम्बडा का उपयोग फ़िल्टर करने के लिए करें\
**reduce** (lambda x,y: x\*y, \[1,2,3,4]) = 24
```
def make_adder(n):
return lambda x: x+n
plus3 = make_adder(3)
plus3(4) = 7 # 3 + 4 = 7

class Car:
crash = lambda self: print('Boom!')
my_car = Car(); my_car.crash() = 'Boom!'
```
mult1 = \[x for x in \[1, 2, 3, 4, 5, 6, 7, 8, 9] if x%3 == 0 ]

### अपवाद
```
def divide(x,y):
try:
result = x/y
except ZeroDivisionError, e:
print “division by zero!” + str(e)
except TypeError:
divide(int(x),int(y))
else:
print “result i”, result
finally
print “executing finally clause in any case”
```
### Assert()

यदि शर्त गलत होती है तो स्क्रीन पर स्ट्रिंग प्रिंट होगी।
```
def avg(grades, weights):
assert not len(grades) == 0, 'no grades data'
assert len(grades) == 'wrong number grades'
```
### जेनरेटर, yield

एक जेनरेटर, कुछ वापस नहीं करता है, बल्कि वह कुछ "यील्ड" करता है। जब आप इसे एक्सेस करते हैं, तो यह पहले उत्पन्न हुए मान को "वापस" करेगा, फिर आप इसे फिर से एक्सेस कर सकते हैं और यह अगले उत्पन्न हुए मान को वापस करेगा। इसलिए, सभी मान एक साथ उत्पन्न नहीं होते हैं और इसके बजाय सभी मानों की एक सूची का उपयोग करके इसके बजाय इसका उपयोग करके बहुत सारी मेमोरी बचाई जा सकती है।
```
def myGen(n):
yield n
yield n + 1
```
g = myGen(6) --> 6\
next(g) --> 7\
next(g) --> त्रुटि

### नियमित अभिव्यक्तियाँ

import re\
re.search("\w","hola").group() = "h"\
re.findall("\w","hola") = \['h', 'o', 'l', 'a']\
re.findall("\w+(la)","hola caracola") = \['la', 'la']

**विशेष अर्थ:**\
. --> सब कुछ\
\w --> \[a-zA-Z0-9\_]\
\d --> संख्या\
\s --> श्वेतस्थान वर्ण\[ \n\r\t\f]\
\S --> श्वेतस्थान वर्ण के बाहर\
^ --> से शुरू होता है\
$ --> से समाप्त होता है\
\+ --> एक या अधिक\
\* --> 0 या अधिक\
? --> 0 या 1 बार होने की संभावना

**विकल्प:**\
re.search(pat,str,re.IGNORECASE)\
IGNORECASE\
DOTALL --> डॉट को नया पंक्ति मिलाने के लिए अनुमति देता है\
MULTILINE --> ^ और $ को अलग-अलग पंक्तियों में मिलाने की अनुमति देता है

re.findall("<.\*>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>foo\</b>and\<i>so on\</i>']\
re.findall("<.\*?>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>', '\</b>', '\<i>', '\</i>']

IterTools\
**उत्पाद**\
from **itertools** import product --> 1 या अधिक सूचियों के बीच संयोजन उत्पन्न करता है, संभवतः मानों को दोहराता है, कार्टेशियन उत्पाद (वितर्जन गुणधर्म)\
print list(**product**(\[1,2,3],\[3,4])) = \[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]\
print list(**product**(\[1,2,3],repeat = 2)) = \[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

**अनुक्रमणिका**\
from **itertools** import **permutations** --> हर स्थान में सभी वर्णों के संयोजन उत्पन्न करता है\
print list(permutations(\['1','2','3'])) = \[('1', '2', '3'), ('1', '3', '2'), ('2', '1', '3'),... हर संभव संयोजन\
print(list(permutations('123',2))) = \[('1', '2'), ('1', '3'), ('2', '1'), ('2', '3'), ('3', '1'), ('3', '2')] लंबाई 2 के हर संभव संयोजन

**संयोजन**\
from itertools import **combinations** --> बिना वर्णों को दोहराए बिना संभव सभी संयोजन उत्पन्न करता है (यदि "ab" मौजूद है, तो "ba" नहीं उत्पन्न करता है)\
print(list(**combinations**('123',2))) --> \[('1', '2'), ('1', '3'), ('2', '3')]

**combinations\_with\_replacement**\
from itertools import **combinations\_with\_replacement** --> वर्ण के बाद से संभव सभी संयोजन उत्पन्न करता है (उदाहरण के लिए, तीसरा तीसरे के साथ मिश्रित होता है लेकिन दूसरे या पहले के साथ नहीं)\
print(list(**combinations\_with\_replacement**('1133',2))) = \[('1', '1'), ('1', '1'), ('1', '3'), ('1', '3'), ('1', '1'), ('1', '3'), ('1', '3'), ('3', '3'), ('3', '3'), ('3', '3')]

### डेकोरेटर्स

एक फ़ंक्शन को निष्पादित करने के लिए आवश्यक समय का आकार लेने वाला डेकोरेटर (यहां से): [यहां](https://towardsdatascience.com/decorating-functions-in-python-619cbbe82c74)
```python
from functools import wraps
import time
def timeme(func):
@wraps(func)
def wrapper(*args, **kwargs):
print("Let's call our decorated function")
start = time.time()
result = func(*args, **kwargs)
print('Execution time: {} seconds'.format(time.time() - start))
return result
return wrapper

@timeme
def decorated_func():
print("Decorated func!")
```
यदि आप इसे चलाते हैं, तो आप कुछ इस तरह का कुछ देखेंगे:
```
Let's call our decorated function
Decorated func!
Execution time: 4.792213439941406e-05 seconds
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
