# Stego Tricks

{% hint style="success" %}
AWSハッキングの学習と練習:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と練習: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **ハッキングトリックを共有するために、** [**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。

</details>
{% endhint %}

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## **ファイルからデータを抽出する**

### **Binwalk**

埋め込まれた隠しファイルやデータを検索するためのツール。`apt`を介してインストールされ、そのソースは[GitHub](https://github.com/ReFirmLabs/binwalk)で入手できます。
```bash
binwalk file # Displays the embedded data
binwalk -e file # Extracts the data
binwalk --dd ".*" file # Extracts all data
```
### **最も重要な**

ファイルをヘッダーとフッターに基づいて回復し、png画像に便利です。[GitHub](https://github.com/korczis/foremost)でソースを使用して`apt`を介してインストールします。
```bash
foremost -i file # Extracts data
```
### **Exiftool**

ファイルのメタデータを表示するのに役立ちます。[こちら](https://www.sno.phy.queensu.ca/\~phil/exiftool/)で入手可能です。
```bash
exiftool file # Shows the metadata
```
### **Exiv2**

Exiftoolに類似し、メタデータを表示します。`apt`を使用してインストール可能で、[GitHub](https://github.com/Exiv2/exiv2)でソースを入手でき、公式ウェブサイトは[こちら](http://www.exiv2.org/)です。
```bash
exiv2 file # Shows the metadata
```
### **ファイル**

取り扱っているファイルの種類を特定します。

### **文字列**

さまざまなエンコーディング設定を使用して、ファイルから読み取れる文字列を抽出します。
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
### **比較（cmp）**

オンラインで見つかった元のバージョンと変更されたファイルを比較するのに便利です。
```bash
cmp original.jpg stego.jpg -b -l
```
## **テキスト内の隠しデータの抽出**

### **スペース内の隠しデータ**

見かけ上空白のスペースに不可視文字が情報を隠している可能性があります。このデータを抽出するには、[https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder](https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder) を訪れてください。

## **画像からデータの抽出**

### **GraphicMagickを使用して画像の詳細を特定する**

[GraphicMagick](https://imagemagick.org/script/download.php) は画像ファイルの種類を特定し、潜在的な破損を特定するために使用されます。以下のコマンドを実行して画像を検査します：
```bash
./magick identify -verbose stego.jpg
```
修復を試みるために、損傷した画像にメタデータコメントを追加すると役立つかもしれません:
```bash
./magick mogrify -set comment 'Extraneous bytes removed' stego.jpg
```
### **データの隠蔽にSteghideを使用**

Steghideは、`JPEG、BMP、WAV、およびAU`ファイル内にデータを隠すことを容易にし、暗号化されたデータを埋め込んだり抽出したりすることができます。`apt`を使用して簡単にインストールでき、[GitHubでソースコードが利用可能です](https://github.com/StefanoDeVuono/steghide)。

**コマンド:**

* `steghide info file`はファイルに隠されたデータが含まれているかどうかを示します。
* `steghide extract -sf file [--passphrase password]`は隠されたデータを抽出し、パスワードはオプションです。

Webベースの抽出を行う場合は、[このウェブサイト](https://futureboy.us/stegano/decinput.html)を訪れてください。

**Stegcrackerを使用したブルートフォース攻撃:**

* Steghideでパスワードクラッキングを試みるには、[stegcracker](https://github.com/Paradoxis/StegCracker.git)を以下のように使用します:
```bash
stegcracker <file> [<wordlist>]
```
### **PNG および BMP ファイル用の zsteg**

zsteg は、PNG および BMP ファイル内の隠されたデータを特定するのに特化しています。インストールは `gem install zsteg` を使用し、[GitHub でソースを入手](https://github.com/zed-0xff/zsteg)します。

**コマンド:**

- `zsteg -a file` はファイルにすべての検出方法を適用します。
- `zsteg -E file` はデータ抽出用のペイロードを指定します。

### **StegoVeritas および Stegsolve**

**stegoVeritas** はメタデータをチェックし、画像変換を実行し、LSB ブルートフォースなどを適用します。すべてのオプションの完全なリストについては `stegoveritas.py -h` を使用し、すべてのチェックを実行するには `stegoveritas.py stego.jpg` を使用します。

**Stegsolve** はさまざまなカラーフィルタを適用して画像内の隠されたテキストやメッセージを表示します。[GitHub](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve)で入手できます。

### **隠されたコンテンツの検出のための FFT**

高速フーリエ変換（FFT）技術を使用すると、画像内の隠されたコンテンツを明らかにすることができます。有用なリソースは次のとおりです:

- [EPFL Demo](http://bigwww.epfl.ch/demo/ip/demos/FFT/)
- [Ejectamenta](https://www.ejectamenta.com/Fourifier-fullscreen/)
- [GitHub 上の FFTStegPic](https://github.com/0xcomposure/FFTStegPic)

### **オーディオおよび画像ファイル用の Stegpy**

Stegpy は、PNG、BMP、GIF、WebP、WAV などの形式をサポートし、画像およびオーディオファイルに情報を埋め込むことができます。[GitHub](https://github.com/dhsdshdhk/stegpy)で入手できます。

### **PNG ファイルの解析用 Pngcheck**

PNG ファイルを解析したり、その信頼性を検証するには:
```bash
apt-get install pngcheck
pngcheck stego.png
```
### **画像解析のための追加ツール**

さらなる探求のために、以下を訪れてみてください：

* [Magic Eye Solver](http://magiceye.ecksdee.co.uk/)
* [Image Error Level Analysis](https://29a.ch/sandbox/2012/imageerrorlevelanalysis/)
* [Outguess](https://github.com/resurrecting-open-source-projects/outguess)
* [OpenStego](https://www.openstego.com/)
* [DIIT](https://diit.sourceforge.net/)

## **オーディオからデータを抽出する**

**オーディオステガノグラフィ**は、音声ファイル内に情報を隠すためのユニークな方法を提供します。異なるツールが埋め込みや隠されたコンテンツの取り出しに使用されます。

### **Steghide (JPEG、BMP、WAV、AU)**

Steghideは、JPEG、BMP、WAV、およびAUファイルにデータを隠すために設計された多目的なツールです。詳細な手順については、[stego tricks documentation](stego-tricks.md#steghide)を参照してください。

### **Stegpy (PNG、BMP、GIF、WebP、WAV)**

このツールは、PNG、BMP、GIF、WebP、およびWAVなど、さまざまな形式と互換性があります。詳細については、[Stegpy's section](stego-tricks.md#stegpy-png-bmp-gif-webp-wav)を参照してください。

### **ffmpeg**

ffmpegは、オーディオファイルの整合性を評価し、詳細な情報を強調し、不一致を特定するために重要です。
```bash
ffmpeg -v info -i stego.mp3 -f null -
```
### **WavSteg (WAV)**

WavStegは、最も重要でないビット戦略を使用してWAVファイル内にデータを隠したり抽出したりするのに優れています。[GitHub](https://github.com/ragibson/Steganography#WavSteg)で利用可能です。コマンドには次のものがあります：
```bash
python3 WavSteg.py -r -b 1 -s soundfile -o outputfile

python3 WavSteg.py -r -b 2 -s soundfile -o outputfile
```
### **Deepsound**

DeepsoundはAES-256を使用して音声ファイル内の情報の暗号化と検出を可能にします。[公式ページ](http://jpinsoft.net/deepsound/download.aspx)からダウンロードできます。

### **Sonic Visualizer**

オーディオファイルの視覚的および分析的検査には貴重なツールであるSonic Visualizerを使用すると、他の手段では検出できない隠された要素を明らかにすることができます。詳細は[公式ウェブサイト](https://www.sonicvisualiser.org/)をご覧ください。

### **DTMF Tones - Dial Tones**

オーディオファイル内のDTMFトーンを検出することは、[このDTMF検出器](https://unframework.github.io/dtmf-detect/)や[DialABC](http://dialabc.com/sound/detect/index.html)などのオンラインツールを使用して達成できます。

## **その他のテクニック**

### **Binary Length SQRT - QR Code**

平方数になるバイナリデータはQRコードを表す可能性があります。次のスニペットを使用して確認してください：
```python
import math
math.sqrt(2500) #50
```
### **点字翻訳**

点字を翻訳するには、[Branah点字翻訳](https://www.branah.com/braille-translator)が優れたリソースです。

## **参考文献**

* [**https://0xrick.github.io/lists/stego/**](https://0xrick.github.io/lists/stego/)
* [**https://github.com/DominicBreuker/stego-toolkit**](https://github.com/DominicBreuker/stego-toolkit)

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

{% hint style="success" %}
AWSハッキングの学習と実践：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**](https://training.hacktricks.xyz/courses/grte)<img src="/.gitbook/assets/grte.png" alt="" data-size="line">

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)をフォローしてください。
* ハッキングトリックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。

</details>
{% endhint %}
