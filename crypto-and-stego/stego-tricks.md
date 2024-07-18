# 隐写术技巧

{% hint style="success" %}
学习并练习 AWS 黑客技能：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习 GCP 黑客技能：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 检查[**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注**我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向 [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}

**Try Hard 安全团队**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## **从文件中提取数据**

### **Binwalk**

用于搜索二进制文件中嵌入的隐藏文件和数据的工具。通过 `apt` 安装，源代码可在 [GitHub](https://github.com/ReFirmLabs/binwalk) 上找到。
```bash
binwalk file # Displays the embedded data
binwalk -e file # Extracts the data
binwalk --dd ".*" file # Extracts all data
```
### **Foremost**

根据文件的头部和尾部恢复文件，对于 png 图像非常有用。通过 `apt` 安装，源代码位于 [GitHub](https://github.com/korczis/foremost)。
```bash
foremost -i file # Extracts data
```
### **Exiftool**

帮助查看文件元数据，可在[此处](https://www.sno.phy.queensu.ca/\~phil/exiftool/)找到。
```bash
exiftool file # Shows the metadata
```
### **Exiv2**

类似于exiftool，用于查看元数据。可通过`apt`安装，在[GitHub](https://github.com/Exiv2/exiv2)上获取源代码，并有一个[官方网站](http://www.exiv2.org/)。
```bash
exiv2 file # Shows the metadata
```
### **文件**

确定你正在处理的文件类型。

### **字符串**

提取文件中的可读字符串，使用各种编码设置来过滤输出。
```bash
strings -n 6 file # Extracts strings with a minimum length of 6
strings -n 6 file | head -n 20 # First 20 strings
strings -n 6 file | tail -n 20 # Last 20 strings
strings -e s -n 6 file # 7bit strings
strings -e S -n 6 file # 8bit strings
strings -e l -n 6 file # 16bit strings (little-endian)
strings -e b -n 6 file # 16bit strings (big-endian)
strings -e L -n 6 file # 32bit strings (little-endian)
strings -e B -n 6 file # 32bit strings (big-endian)
```
### **比较 (cmp)**

用于将修改后的文件与在线找到的原始版本进行比较。
```bash
cmp original.jpg stego.jpg -b -l
```
## **提取文本中的隐藏数据**

### **空格中的隐藏数据**

看似空白的空格中可能隐藏着不可见字符。要提取这些数据，请访问 [https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder](https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder)。

## **从图像中提取数据**

### **使用GraphicMagick识别图像细节**

[GraphicMagick](https://imagemagick.org/script/download.php) 用于确定图像文件类型并识别潜在的损坏。执行以下命令来检查一个图像：
```bash
./magick identify -verbose stego.jpg
```
尝试修复损坏的图像时，添加元数据注释可能会有所帮助：
```bash
./magick mogrify -set comment 'Extraneous bytes removed' stego.jpg
```
### **Steghide用于数据隐藏**

Steghide可用于在`JPEG、BMP、WAV和AU`文件中隐藏数据，能够嵌入和提取加密数据。使用`apt`可以轻松安装，其[源代码可在GitHub上找到](https://github.com/StefanoDeVuono/steghide)。

**命令:**

* `steghide info file` 用于查看文件是否包含隐藏数据。
* `steghide extract -sf file [--passphrase password]` 用于提取隐藏数据，密码可选。

要进行基于Web的提取，请访问[此网站](https://futureboy.us/stegano/decinput.html)。

**使用Stegcracker进行暴力破解攻击:**

* 要在Steghide上尝试密码破解，请使用[stegcracker](https://github.com/Paradoxis/StegCracker.git)，如下所示:
```bash
stegcracker <file> [<wordlist>]
```
### **zsteg 用于 PNG 和 BMP 文件**

zsteg 专门用于揭示 PNG 和 BMP 文件中的隐藏数据。安装可通过 `gem install zsteg` 完成，其[源代码在 GitHub 上](https://github.com/zed-0xff/zsteg)。

**命令:**

* `zsteg -a file` 在文件上应用所有检测方法。
* `zsteg -E file` 指定用于数据提取的有效载荷。

### **StegoVeritas 和 Stegsolve**

**stegoVeritas** 检查元数据，执行图像转换，并应用 LSB 强制破解等其他功能。使用 `stegoveritas.py -h` 查看所有选项的完整列表，使用 `stegoveritas.py stego.jpg` 执行所有检查。

**Stegsolve** 应用各种颜色滤镜以显示图像中隐藏的文本或消息。可在[GitHub](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve)上找到。

### **FFT 用于隐藏内容检测**

快速傅里叶变换 (FFT) 技术可揭示图像中的隐藏内容。有用的资源包括:

* [EPFL 演示](http://bigwww.epfl.ch/demo/ip/demos/FFT/)
* [Ejectamenta](https://www.ejectamenta.com/Fourifier-fullscreen/)
* [GitHub 上的 FFTStegPic](https://github.com/0xcomposure/FFTStegPic)

### **Stegpy 用于音频和图像文件**

Stegpy 允许将信息嵌入图像和音频文件中，支持 PNG、BMP、GIF、WebP 和 WAV 等格式。可在[GitHub](https://github.com/dhsdshdhk/stegpy)上找到。

### **Pngcheck 用于 PNG 文件分析**

要分析 PNG 文件或验证其真实性，请使用:
```bash
apt-get install pngcheck
pngcheck stego.png
```
### **图像分析的附加工具**

要进行进一步探索，请考虑访问：

* [Magic Eye Solver](http://magiceye.ecksdee.co.uk/)
* [Image Error Level Analysis](https://29a.ch/sandbox/2012/imageerrorlevelanalysis/)
* [Outguess](https://github.com/resurrecting-open-source-projects/outguess)
* [OpenStego](https://www.openstego.com/)
* [DIIT](https://diit.sourceforge.net/)

## **从音频中提取数据**

**音频隐写术** 提供了一种在声音文件中隐藏信息的独特方法。不同的工具被用于嵌入或检索隐藏内容。

### **Steghide (JPEG, BMP, WAV, AU)**

Steghide 是一个多功能工具，旨在将数据隐藏在 JPEG、BMP、WAV 和 AU 文件中。详细说明请参阅[隐写技巧文档](stego-tricks.md#steghide)。

### **Stegpy (PNG, BMP, GIF, WebP, WAV)**

该工具兼容多种格式，包括 PNG、BMP、GIF、WebP 和 WAV。有关更多信息，请参阅[Stegpy 的部分](stego-tricks.md#stegpy-png-bmp-gif-webp-wav)。

### **ffmpeg**

ffmpeg 对于评估音频文件的完整性至关重要，突出显示详细信息并指出任何差异。
```bash
ffmpeg -v info -i stego.mp3 -f null -
```
### **WavSteg (WAV)**

WavSteg在使用最低有效位策略在WAV文件中隐藏和提取数据方面表现出色。它可在[GitHub](https://github.com/ragibson/Steganography#WavSteg)上找到。命令包括：
```bash
python3 WavSteg.py -r -b 1 -s soundfile -o outputfile

python3 WavSteg.py -r -b 2 -s soundfile -o outputfile
```
### **Deepsound**

Deepsound允许使用AES-256在声音文件中加密和检测信息。可从[官方页面](http://jpinsoft.net/deepsound/download.aspx)下载。

### **Sonic Visualizer**

Sonic Visualizer是一款无价的工具，用于对音频文件进行视觉和分析检查，可以揭示其他方法无法检测到的隐藏元素。访问[官方网站](https://www.sonicvisualiser.org/)了解更多信息。

### **DTMF Tones - 拨号音**

可以通过在线工具如[此DTMF检测器](https://unframework.github.io/dtmf-detect/)和[DialABC](http://dialabc.com/sound/detect/index.html)来检测音频文件中的DTMF音调。

## **其他技术**

### **二进制长度平方根 - QR码**

平方为整数的二进制数据可能代表一个QR码。使用以下代码片段进行检查：
```python
import math
math.sqrt(2500) #50
```
### **盲文翻译**

要进行盲文翻译，请使用[Branah盲文翻译器](https://www.branah.com/braille-translator)这一优秀资源。

## **参考资料**

* [**https://0xrick.github.io/lists/stego/**](https://0xrick.github.io/lists/stego/)
* [**https://github.com/DominicBreuker/stego-toolkit**](https://github.com/DominicBreuker/stego-toolkit)

**Try Hard安全团队**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

{% hint style="success" %}
学习并练习AWS黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks培训AWS红队专家（ARTE）**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习GCP黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks培训GCP红队专家（GRTE）**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持HackTricks</summary>

* 查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享黑客技巧。

</details>
{% endhint %}
