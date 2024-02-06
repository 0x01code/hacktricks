# मूल Python

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## Python Basics

### उपयोगी जानकारी

list(xrange()) == range() --> Python3 में range Python2 का xrange है (यह एक सूचकक है न कि एक सूची)\
एक Tuple और एक List के बीच अंतर यह है कि एक Tuple में एक मान की स्थिति का मतलब होता है लेकिन सूचियों में केवल क्रमबद्ध मान होते हैं। Tuples में संरचनाएँ होती हैं लेकिन सूचियों में क्रम होता है।

### मुख्य ऑपरेशन

एक संख्या को उठाने के लिए आप इस्तेमाल करते हैं: 3\*\*2 (नहीं 3^2)\
यदि आप 2/3 करते हैं तो यह 1 लौटाता है क्योंकि आप दो ints (पूर्णांक) को विभाजित कर रहे हैं। यदि आप दशमलव चाहते हैं तो आपको floats से विभाजित करना चाहिए (2.0/3.0)।\
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

**वर्णों को जोड़ें**\
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
### टपल्स

t1 = (1,'2,'three')\
t2 = (5,6)\
t3 = t1 + t2 = (1, '2', 'three', 5, 6)\
(4,) = Singelton\
d = () empty tuple\
d += (4,) --> एक टपल में जोड़ें\
CANT! --> t1\[1] == 'New value'\
list(t2) = \[5,6] --> टपल से सूची में

### सूची (एरे)

d = \[] empty\
a = \[1,2,3]\
b = \[4,5]\
a + b = \[1,2,3,4,5]\
b.append(6) = \[4,5,6]\
tuple(a) = (1,2,3) --> सूची से टपल में

### शब्दकोश

d = {} empty\
monthNumbers={1:’Jan’, 2: ‘feb’,’feb’:2}—> monthNumbers ->{1:’Jan’, 2: ‘feb’,’feb’:2}\
monthNumbers\[1] = ‘Jan’\
monthNumbers\[‘feb’] = 2\
list(monthNumbers) = \[1,2,’feb’]\
monthNumbers.values() = \[‘Jan’,’feb’,2]\
keys = \[k for k in monthNumbers]\
a={'9':9}\
monthNumbers.update(a) = {'9':9, 1:’Jan’, 2: ‘feb’,’feb’:2}\
mN = monthNumbers.copy() #Independent copy\
monthNumbers.get('key',0) #Check if key exists, Return value of monthNumbers\["key"] or 0 if it does not exists

### सेट

सेट में पुनरावृत्तियाँ नहीं होतीं हैं\
myset = set(\['a', 'b']) = {'a', 'b'}\
myset.add('c') = {'a', 'b', 'c'}\
myset.add('a') = {'a', 'b', 'c'} #कोई पुनरावृत्ति नहीं\
myset.update(\[1,2,3]) = set(\['a', 1, 2, 'b', 'c', 3])\
myset.discard(10) #यदि मौजूद है, तो हटा दें, अगर नहीं, तो कुछ नहीं\
myset.remove(10) #यदि मौजूद है, तो हटा दें, अगर नहीं, तो अपशिष्ट उठाएं\
myset2 = set(\[1, 2, 3, 4])\
myset.union(myset2) #मेरे सेट या myset2 में मान\
myset.intersection(myset2) #मेरे सेट और myset2 में मान\
myset.difference(myset2) #मेरे सेट में मान लेकिन myset2 में नहीं\
myset.symmetric\_difference(myset2) #वह मान जो मेरे सेट और myset2 में नहीं हैं (दोनों में नहीं)\
myset.pop() #सेट का पहला तत्व प्राप्त करें और हटा दें\
myset.intersection\_update(myset2) #myset = मेरे सेट और myset2 में तत्व\
myset.difference\_update(myset2) #myset = मेरे सेट में तत्व लेकिन myset2 में नहीं\
myset.symmetric\_difference\_update(myset2) #myset = उन तत्वों को जो दोनों में नहीं हैं

### कक्षाएँ

\_\_It\_\_ में वाला विधि उसे तुलना करने के लिए उपयोग किया जाएगा यदि इस कक्षा के एक ऑब्जेक्ट दूसरे से बड़ा है
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

**Map** इसके जैसा है: \[f(x) for x in iterable] --> map(tutple,\[a,b]) = \[(1,2,3),(4,5)]\
m = map(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) --> \[False, False, True, False, False, True, False, False, True]

**zip** उस समय रुकता है जब foo या bar में से छोटा हो जाता है:
```
for f, b in zip(foo, bar):
print(f, b)
```
**लैम्बडा** का उपयोग एक फ़ंक्शन को परिभाषित करने के लिए किया जाता है\
(lambda x,y: x+y)(5,3) = 8 --> लैम्बडा का उपयोग सरल **फ़ंक्शन** के रूप में करें\
**sorted**(range(-5,6), key=lambda x: x\*\* 2) = \[0, -1, 1, -2, 2, -3, 3, -4, 4, -5, 5] --> लैम्बडा का उपयोग सूची को क्रमबद्ध करने के लिए करें\
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
mult1 = \[x जिसके लिए x \[1, 2, 3, 4, 5, 6, 7, 8, 9\] में हैं अगर x%3 == 0 ]

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

यदि स्थिति गलत है तो स्क्रीन पर स्ट्रिंग प्रिंट होगी
```
def avg(grades, weights):
assert not len(grades) == 0, 'no grades data'
assert len(grades) == 'wrong number grades'
```
### जेनरेटर, yield

एक जेनरेटर, कुछ वापस नहीं देता, बल्कि यह कुछ "यील्ड" करता है। जब आप इसे एक्सेस करते हैं, तो यह पहले जेनरेट किया गया पहला मान "वापस" करेगा, फिर, आप इसे फिर से एक्सेस कर सकते हैं और यह अगला जेनरेट किया गया मान वापस करेगा। इसलिए, सभी मान एक साथ नहीं उत्पन्न होते हैं और इसका उपयोग करके सभी मानों वाली एक सूची की बजाय बहुत सारी मेमोरी बचाई जा सकती है।
```
def myGen(n):
yield n
yield n + 1
```
```markdown
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
\s --> सफेद जगह वाला वर्ण\[ \n\r\t\f]\
\S --> गैर-सफेद जगह वाला वर्ण\
^ --> से शुरू होता है\
$ --> से समाप्त होता है\
\+ --> एक या अधिक\
\* --> 0 या अधिक\
? --> 0 या 1 बार होने की संभावना

**विकल्प:**\
re.search(pat,str,re.IGNORECASE)\
IGNORECASE\
DOTALL --> डॉट को न्यूलाइन से मिलाने दें\
MULTILINE --> ^ और $ को विभिन्न पंक्तियों में मिलाने दें

re.findall("<.\*>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>foo\</b>and\<i>so on\</i>']\
re.findall("<.\*?>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>', '\</b>', '\<i>', '\</i>']

IterTools\
**product**\
from **itertools** import product --> 1 या अधिक सूचियों के बीच संयोजन उत्पन्न करता है, शायद मानों को दोहराता है, कार्तेसियन उत्पाद (वितरणीय गुणाकार)\
print list(**product**(\[1,2,3],\[3,4])) = \[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]\
print list(**product**(\[1,2,3],repeat = 2)) = \[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

**permutations**\
from **itertools** import **permutations** --> हर स्थान में सभी वर्णों के संयोजन उत्पन्न करता है\
print list(permutations(\['1','2','3'])) = \[('1', '2', '3'), ('1', '3', '2'), ('2', '1', '3'),... हर संभावित संयोजन\
print(list(permutations('123',2))) = \[('1', '2'), ('1', '3'), ('2', '1'), ('2', '3'), ('3', '1'), ('3', '2')] हर संभावित संयोजन जिनकी लंबाई 2 है

**combinations**\
from itertools import **combinations** --> विभिन्न वर्णों के बिना दोहराए गए सभी संभावित संयोजन उत्पन्न करता है ("ab" मौजूद है, "ba" उत्पन्न नहीं करता)\
print(list(**combinations**('123',2))) --> \[('1', '2'), ('1', '3'), ('2', '3')]

**combinations\_with\_replacement**\
from itertools import **combinations\_with\_replacement** --> वर्ण से आगे सभी संभावित संयोजन उत्पन्न करता है (उदाहरण के लिए, तीसरा तीसरे के बाद मिश्रित है लेकिन दूसरे या पहले के साथ नहीं)\
print(list(**combinations\_with\_replacement**('1133',2))) = \[('1', '1'), ('1', '1'), ('1', '3'), ('1', '3'), ('1', '1'), ('1', '3'), ('1', '3'), ('3', '3'), ('3', '3'), ('3', '3')]

### डेकोरेटर्स

किसी फ़ंक्शन को निष्पादित करने के लिए जरूरी समय का आकार बढ़ाने वाला डेकोरेटर (यहाँ से): [यहाँ](https://towardsdatascience.com/decorating-functions-in-python-619cbbe82c74):
```
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
यदि आप इसे चलाते हैं, तो आप कुछ इस प्रकार का कुछ देखेंगे:
```
Let's call our decorated function
Decorated func!
Execution time: 4.792213439941406e-05 seconds
```
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** को PRs जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
