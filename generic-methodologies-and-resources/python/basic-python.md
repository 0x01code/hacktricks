# 基本的なPython

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>を通じて、ゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションをご覧ください
* **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter** 🐦で**フォロー**する：[**@hacktricks_live**](https://twitter.com/hacktricks_live)**。**
* **ハッキングトリックを共有するには、PRを** [**HackTricks**](https://github.com/carlospolop/hacktricks) **および** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **のGitHubリポジトリに提出してください。**

</details>

## Pythonの基礎

### 便利な情報

list(xrange()) == range() --> Python3ではrangeはPython2のxrangeと同じ（リストではなくジェネレーター）\
タプルとリストの違いは、タプル内の値の位置が意味を持つが、リストは単なる順序付けられた値であることです。タプルには構造がありますが、リストには順序があります。

### 主な操作

数を累乗するには：3\*\*2（3^2ではありません）\
2/3を行うと1が返されます。これは2つの整数（integers）を割っているためです。小数点以下を取得するには、浮動小数点数で割る必要があります（2.0/3.0）。\
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
dir(str) = 利用可能なすべてのメソッドのリスト\
help(str) = クラスstrの定義\
"a".upper() = "A"\
"A".lower() = "a"\
"abc".capitalize() = "Abc"\
sum(\[1,2,3]) = 6\
sorted(\[1,43,5,3,21,4])

**文字を結合する**\
3 \* ’a’ = ‘aaa’\
‘a’ + ‘b’ = ‘ab’\
‘a’ + str(3) = ‘a3’\
\[1,2,3]+\[4,5]=\[1,2,3,4,5]

**リストの部分**\
‘abc’\[0] = ‘a’\
'abc’\[-1] = ‘c’\
'abc’\[1:3] = ‘bc’ from \[1] to \[2]\
"qwertyuiop"\[:-1] = 'qwertyuio'

**コメント**\
\# 1行コメント\
"""\
複数行コメント\
別のコメント\
"""

**ループ**
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
### タプル

t1 = (1, '2', 'three')\
t2 = (5, 6)\
t3 = t1 + t2 = (1, '2', 'three', 5, 6)\
(4,) = シングルトン\
d = () 空のタプル\
d += (4,) --> タプルに追加\
CANT! --> t1\[1] == 'New value'\
list(t2) = \[5, 6] --> タプルからリストへ

### リスト（配列）

d = \[] 空\
a = \[1, 2, 3]\
b = \[4, 5]\
a + b = \[1, 2, 3, 4, 5]\
b.append(6) = \[4, 5, 6]\
tuple(a) = (1, 2, 3) --> リストからタプルへ

### 辞書

d = {} 空\
monthNumbers = {1: 'Jan', 2: 'feb', 'feb': 2} --> monthNumbers -> {1: 'Jan', 2: 'feb', 'feb': 2}\
monthNumbers\[1] = 'Jan'\
monthNumbers\[‘feb’] = 2\
list(monthNumbers) = \[1, 2, 'feb']\
monthNumbers.values() = \['Jan', 'feb', 2]\
keys = \[k for k in monthNumbers]\
a = {'9': 9}\
monthNumbers.update(a) = {'9': 9, 1: 'Jan', 2: 'feb', 'feb': 2}\
mN = monthNumbers.copy() #独立したコピー\
monthNumbers.get('key', 0) #キーが存在するか確認し、存在する場合はmonthNumbers\["key"]の値を返し、存在しない場合は0を返す

### 集合

集合には重複がない\
myset = set(\['a', 'b']) = {'a', 'b'}\
myset.add('c') = {'a', 'b', 'c'}\
myset.add('a') = {'a', 'b', 'c'} #重複なし\
myset.update(\[1, 2, 3]) = set(\['a', 1, 2, 'b', 'c', 3])\
myset.discard(10) #存在する場合は削除、存在しない場合は何もしない\
myset.remove(10) #存在する場合は削除、存在しない場合は例外を発生\
myset2 = set(\[1, 2, 3, 4])\
myset.union(myset2) #mysetまたはmyset2の値\
myset.intersection(myset2) #mysetとmyset2の値\
myset.difference(myset2) #mysetにあるがmyset2にない値\
myset.symmetric\_difference(myset2) #mysetとmyset2の両方にない値\
myset.pop() #集合の最初の要素を取得して削除\
myset.intersection\_update(myset2) #myset = mysetとmyset2の両方にある要素\
myset.difference\_update(myset2) #myset = mysetにあってmyset2にない要素\
myset.symmetric\_difference\_update(myset2) #myset = 両方にない要素
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
### map, zip, filter, lambda, sorted and one-liners

**Map** is like: \[f(x) for x in iterable] --> map(tutple,\[a,b]) = \[(1,2,3),(4,5)]\
m = map(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) --> \[False, False, True, False, False, True, False, False, True]

**zip** stops when the shorter of foo or bar stops:
```
for f, b in zip(foo, bar):
print(f, b)
```
**Lambda**は関数を定義するために使用されます\
(lambda x,y: x+y)(5,3) = 8 --> Lambdaを単純な**関数**として使用する\
**sorted**(range(-5,6), key=lambda x: x\*\* 2) = \[0, -1, 1, -2, 2, -3, 3, -4, 4, -5, 5] --> リストをソートするためにLambdaを使用する\
m = **filter**(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) = \[3, 6, 9] --> フィルタリングするためにLambdaを使用する\
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
```python
mult1 = [x for x in [1, 2, 3, 4, 5, 6, 7, 8, 9] if x%3 == 0 ]
```

### 例外
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

条件がfalseの場合、文字列が画面に表示されます
```
def avg(grades, weights):
assert not len(grades) == 0, 'no grades data'
assert len(grades) == 'wrong number grades'
```
### ジェネレータ、yield

ジェネレータは、何かを返す代わりに、何かを「yield（生成）」します。アクセスすると、最初に生成された値を「返し」、その後、再度アクセスすると次に生成された値を返します。つまり、すべての値が同時に生成されるのではなく、すべての値を持つリストよりもこれを使用することで多くのメモリを節約できます。
```
def myGen(n):
yield n
yield n + 1
```
```markdown
g = myGen(6) --> 6\
next(g) --> 7\
next(g) --> エラー

### 正規表現

import re\
re.search("\w","hola").group() = "h"\
re.findall("\w","hola") = \['h', 'o', 'l', 'a']\
re.findall("\w+(la)","hola caracola") = \['la', 'la']

**特殊な意味:**\
. --> すべて\
\w --> \[a-zA-Z0-9\_]\
\d --> 数字\
\s --> 空白文字\[ \n\r\t\f]\
\S --> 空白以外の文字\
^ --> で始まる\
$ --> で終わる\
\+ --> 1つ以上\
\* --> 0個以上\
? --> 0または1回

**オプション:**\
re.search(pat,str,re.IGNORECASE)\
IGNORECASE\
DOTALL --> ドットが改行に一致するようにする\
MULTILINE --> ^ および $ が異なる行で一致するようにする

re.findall("<.\*>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>foo\</b>and\<i>so on\</i>']\
re.findall("<.\*?>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>', '\</b>', '\<i>', '\</i>']

IterTools\
**product**\
from **itertools** import product --> 1つ以上のリスト間の組み合わせを生成し、値を繰り返すことがある、直積（分配特性）\
print list(**product**(\[1,2,3],\[3,4])) = \[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]\
print list(**product**(\[1,2,3],repeat = 2)) = \[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

**permutations**\
from **itertools** import **permutations** --> 各位置のすべての文字の組み合わせを生成する\
print list(permutations(\['1','2','3'])) = \[('1', '2', '3'), ('1', '3', '2'), ('2', '1', '3'),... すべての可能な組み合わせ\
print(list(permutations('123',2))) = \[('1', '2'), ('1', '3'), ('2', '1'), ('2', '3'), ('3', '1'), ('3', '2')] 長さ2のすべての可能な組み合わせ

**combinations**\
from itertools import **combinations** --> 文字を繰り返さずにすべての可能な組み合わせを生成する（"ab"が存在する場合、"ba"は生成されない）\
print(list(**combinations**('123',2))) --> \[('1', '2'), ('1', '3'), ('2', '3')]

**combinations\_with\_replacement**\
from itertools import **combinations\_with\_replacement** --> 文字以降のすべての可能な組み合わせを生成する（たとえば、3番目は3番目以降から混合されますが、2番目や最初とは混合されません）\
print(list(**combinations\_with\_replacement**('1133',2))) = \[('1', '1'), ('1', '1'), ('1', '3'), ('1', '3'), ('1', '1'), ('1', '3'), ('1', '3'), ('3', '3'), ('3', '3'), ('3', '3')]

### デコレータ

関数の実行にかかる時間を測定するデコレータ（[こちら](https://towardsdatascience.com/decorating-functions-in-python-619cbbe82c74)から）
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
もし実行すると、以下のようなものが表示されます:
```
Let's call our decorated function
Decorated func!
Execution time: 4.792213439941406e-05 seconds
```
<details>

<summary><strong>AWSハッキングをゼロからヒーローまで学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**をフォローする**
* **ハッキングトリックを共有するには、PRを** [**HackTricks**](https://github.com/carlospolop/hacktricks) **および** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **のGitHubリポジトリに提出してください。**

</details>
