# Trucos para CTFs de Criptografía

<details>

<summary><strong>Aprende a hackear en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver a tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** revisa los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Consigue el [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de GitHub de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Bases de Datos de Hashes en Línea

* _**Búscalo en Google**_
* [http://hashtoolkit.com/reverse-hash?hash=4d186321c1a7f0f354b297e8914ab240](http://hashtoolkit.com/reverse-hash?hash=4d186321c1a7f0f354b297e8914ab240)
* [https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com)
* [https://crackstation.net/](https://crackstation.net)
* [https://md5decrypt.net/](https://md5decrypt.net)
* [https://www.onlinehashcrack.com](https://www.onlinehashcrack.com)
* [https://gpuhash.me/](https://gpuhash.me)
* [https://hashes.org/search.php](https://hashes.org/search.php)
* [https://www.cmd5.org/](https://www.cmd5.org)
* [https://hashkiller.co.uk/Cracker/MD5](https://hashkiller.co.uk/Cracker/MD5)
* [https://www.md5online.org/md5-decrypt.html](https://www.md5online.org/md5-decrypt.html)

## Autosolvers Mágicos

* [**https://github.com/Ciphey/Ciphey**](https://github.com/Ciphey/Ciphey)
* [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/) (Módulo Mágico)
* [https://github.com/dhondta/python-codext](https://github.com/dhondta/python-codext)
* [https://www.boxentriq.com/code-breaking](https://www.boxentriq.com/code-breaking)

## Codificadores

La mayoría de los datos codificados se pueden decodificar con estos 2 recursos:

* [https://www.dcode.fr/tools-list](https://www.dcode.fr/tools-list)
* [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)

### Autosolvers de Sustitución

* [https://www.boxentriq.com/code-breaking/cryptogram](https://www.boxentriq.com/code-breaking/cryptogram)
* [https://quipqiup.com/](https://quipqiup.com) - ¡Muy bueno!

#### César - ROTx Autosolvers

* [https://www.nayuki.io/page/automatic-caesar-cipher-breaker-javascript](https://www.nayuki.io/page/automatic-caesar-cipher-breaker-javascript)

#### Cifrado Atbash

* [http://rumkin.com/tools/cipher/atbash.php](http://rumkin.com/tools/cipher/atbash.php)

### Autosolver de Codificaciones Base

Revisa todas estas bases con: [https://github.com/dhondta/python-codext](https://github.com/dhondta/python-codext)

* **Ascii85**
* `BQ%]q@psCd@rH0l`
* **Base26** \[_A-Z_]
* `BQEKGAHRJKHQMVZGKUXNT`
* **Base32** \[_A-Z2-7=_]
* `NBXWYYLDMFZGCY3PNRQQ====`
* **Zbase32** \[_ybndrfg8ejkmcpqxot1uwisza345h769_]
* `pbzsaamdcf3gna5xptoo====`
* **Base32 Geohash** \[_0-9b-hjkmnp-z_]
* `e1rqssc3d5t62svgejhh====`
* **Base32 Crockford** \[_0-9A-HJKMNP-TV-Z_]
* `D1QPRRB3C5S62RVFDHGG====`
* **Base32 Extended Hexadecimal** \[_0-9A-V_]
* `D1NMOOB3C5P62ORFDHGG====`
* **Base45** \[_0-9A-Z $%\*+-./:_]
* `59DPVDGPCVKEUPCPVD`
* **Base58 (bitcoin)** \[_1-9A-HJ-NP-Za-km-z_]
* `2yJiRg5BF9gmsU6AC`
* **Base58 (flickr)** \[_1-9a-km-zA-HJ-NP-Z_]
* `2YiHqF5bf9FLSt6ac`
* **Base58 (ripple)** \[_rpshnaf39wBUDNEGHJKLM4PQ-T7V-Z2b-eCg65jkm8oFqi1tuvAxyz_]
* `pyJ5RgnBE9gm17awU`
* **Base62** \[_0-9A-Za-z_]
* `g2AextRZpBKRBzQ9`
* **Base64** \[_A-Za-z0-9+/=_]
* `aG9sYWNhcmFjb2xh`
* **Base67** \[_A-Za-z0-9-_.!\~\_]
* `NI9JKX0cSUdqhr!p`
* **Base85 (Ascii85)** \[_!"#$%&'()\*+,-./0-9:;<=>?@A-Z\[\\]^\_\`a-u_]
* `BQ%]q@psCd@rH0l`
* **Base85 (Adobe)** \[_!"#$%&'()\*+,-./0-9:;<=>?@A-Z\[\\]^\_\`a-u_]
* `<~BQ%]q@psCd@rH0l~>`
* **Base85 (IPv6 o RFC1924)** \[_0-9A-Za-z!#$%&()\*+-;<=>?@^_\`{|}\~\_]
* `Xm4y`V\_|Y(V{dF>\`
* **Base85 (xbtoa)** \[_!"#$%&'()\*+,-./0-9:;<=>?@A-Z\[\\]^\_\`a-u_]
* `xbtoa Begin\nBQ%]q@psCd@rH0l\nxbtoa End N 12 c E 1a S 4e6 R 6991d`
* **Base85 (XML)** \[_0-9A-Za-y!#$()\*+,-./:;=?@^\`{|}\~z\__]
* `Xm4y|V{~Y+V}dF?`
* **Base91** \[_A-Za-z0-9!#$%&()\*+,./:;<=>?@\[]^\_\`{|}\~"_]
* `frDg[*jNN!7&BQM`
* **Base100** \[]
* `👟👦👣👘👚👘👩👘👚👦👣👘`
* **Base122** \[]
* `4F ˂r0Xmvc`
* **ATOM-128** \[_/128GhIoPQROSTeUbADfgHijKLM+n0pFWXY456xyzB7=39VaqrstJklmNuZvwcdEC_]
* `MIc3KiXa+Ihz+lrXMIc3KbCC`
* **HAZZ15** \[_HNO4klm6ij9n+J2hyf0gzA8uvwDEq3X1Q7ZKeFrWcVTts/MRGYbdxSo=ILaUpPBC5_]
* `DmPsv8J7qrlKEoY7`
* **MEGAN35** \[_3G-Ub=c-pW-Z/12+406-9Vaq-zA-F5_]
* `kLD8iwKsigSalLJ5`
* **ZONG22** \[_ZKj9n+yf0wDVX1s/5YbdxSo=ILaUpPBCHg8uvNO4klm6iJGhQ7eFrWczAMEq3RTt2_]
* `ayRiIo1gpO+uUc7g`
* **ESAB46** \[]
* `3sHcL2NR8WrT7mhR`
* **MEGAN45** \[]
* `kLD8igSXm2KZlwrX`
* **TIGO3FX** \[]
* `7AP9mIzdmltYmIP9mWXX`
* **TRIPO5** \[]
* `UE9vSbnBW6psVzxB`
* **FERON74** \[]
* `PbGkNudxCzaKBm0x`
* **GILA7** \[]
* `D+nkv8C1qIKMErY1`
* **Citrix CTX1** \[]
* `MNGIKCAHMOGLKPAKMMGJKNAINPHKLOBLNNHILCBHNOHLLPBK`

[http://k4.cba.pl/dw/crypo/tools/eng\_atom128c.html](http://k4.cba.pl/dw/crypo/tools/eng\_atom128c.html) - 404 Muerto: [https://web.archive.org/web/20190228181208/http://k4.cba.pl/dw/crypo/tools/eng\_hackerize.html](https://web.archive.org/web/20190228181208/http://k4.cba.pl/dw/crypo/tools/eng\_hackerize.html)

### HackerizeXS \[_╫Λ↻├☰┏_]
```
╫☐↑Λ↻Λ┏Λ↻☐↑Λ
```
### Morse

[http://k4.cba.pl/dw/crypo/tools/eng_hackerize.html](http://k4.cba.pl/dw/crypo/tools/eng_hackerize.html) - 404 Muerto: [https://web.archive.org/web/20190228181208/http://k4.cba.pl/dw/crypo/tools/eng_hackerize.html](https://web.archive.org/web/20190228181208/http://k4.cba.pl/dw/crypo/tools/eng_hackerize.html)
```
.... --- .-.. -.-. .- .-. .- -.-. --- .-.. .-
```
### UUencoder

No hay texto para traducir aparte del título, que se mantendría igual ya que es un nombre propio de una técnica de codificación.
```
begin 644 webutils_pl
M2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(
M3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/
F3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$$`
`
end
```
* [http://www.webutils.pl/index.php?idx=uu](http://www.webutils.pl/index.php?idx=uu)

### XXEncoder
```
begin 644 webutils_pl
hG2xAEIVDH236Hol-G2xAEIVDH236Hol-G2xAEIVDH236Hol-G2xAEIVDH236
5Hol-G2xAEE++
end
```
### YEncoder
```
=ybegin line=128 size=28 name=webutils_pl
ryvkryvkryvkryvkryvkryvkryvk
=yend size=28 crc32=35834c86
```
* [http://www.webutils.pl/index.php?idx=yenc](http://www.webutils.pl/index.php?idx=yenc)

### BinHex
```
(This file must be converted with BinHex 4.0)
:#hGPBR9dD@acAh"X!$mr2cmr2cmr!!!!!!!8!!!!!-ka5%p-38K26%&)6da"5%p
-38K26%'d9J!!:
```
* [http://www.webutils.pl/index.php?idx=binhex](http://www.webutils.pl/index.php?idx=binhex)

### ASCII85
```
<~85DoF85DoF85DoF85DoF85DoF85DoF~>
```
* [http://www.webutils.pl/index.php?idx=ascii85](http://www.webutils.pl/index.php?idx=ascii85)

### Teclado Dvorak
```
drnajapajrna
```
* [https://www.geocachingtoolbox.com/index.php?lang=en\&page=dvorakKeyboard](https://www.geocachingtoolbox.com/index.php?lang=en\&page=dvorakKeyboard)

### A1Z26

Letras a su valor numérico
```
8 15 12 1 3 1 18 1 3 15 12 1
```
### Cifrado Afin Codificar

Letra a número `(ax+b)%26` (_a_ y _b_ son las claves y _x_ es la letra) y el resultado de vuelta a letra
```
krodfdudfrod
```
### Código SMS

**Multitap** [reemplaza una letra](https://www.dcode.fr/word-letter-change) por dígitos repetidos definidos por el código de tecla correspondiente en un [teclado de teléfono móvil](https://www.dcode.fr/phone-keypad-cipher) (Este modo se utiliza al escribir SMS).\
Por ejemplo: 2=A, 22=B, 222=C, 3=D...\
Puedes identificar este código porque verás **varios números repetidos**.

Puedes decodificar este código en: [https://www.dcode.fr/multitap-abc-cipher](https://www.dcode.fr/multitap-abc-cipher)

### Código Bacon

Sustituye cada letra por 4 As o Bs (o 1s y 0s)
```
00111 01101 01010 00000 00010 00000 10000 00000 00010 01101 01010 00000
AABBB ABBAB ABABA AAAAA AAABA AAAAA BAAAA AAAAA AAABA ABBAB ABABA AAAAA
```
### Runas

![](../.gitbook/assets/runes.jpg)

## Compresión

**Raw Deflate** y **Raw Inflate** (puedes encontrar ambos en Cyberchef) pueden comprimir y descomprimir datos sin encabezados.

## Criptografía Fácil

### XOR - Autosolver

* [https://wiremask.eu/tools/xor-cracker/](https://wiremask.eu/tools/xor-cracker/)

### Bifid

Se necesita una palabra clave
```
fgaargaamnlunesuneoa
```
### Vigenere

Se necesita una palabra clave
```
wodsyoidrods
```
## Criptografía Fuerte

### Fernet

2 cadenas base64 (token y clave)
```
Token:
gAAAAABWC9P7-9RsxTz_dwxh9-O2VUB7Ih8UCQL1_Zk4suxnkCvb26Ie4i8HSUJ4caHZuiNtjLl3qfmCv_fS3_VpjL7HxCz7_Q==

Key:
-s6eI5hyNh8liH7Gq0urPC-vzPgNnxauKvRO4g03oYI=
```
* [https://asecuritysite.com/encryption/ferdecode](https://asecuritysite.com/encryption/ferdecode)

### Compartición de Secretos de Samir

Un secreto se divide en X partes y para recuperarlo necesitas Y partes (_Y <= X_).
```
8019f8fa5879aa3e07858d08308dc1a8b45
80223035713295bddf0b0bd1b10a5340b89
803bc8cf294b3f83d88e86d9818792e80cd
```
### Fuerza bruta con OpenSSL

## Herramientas

<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver a tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** revisa los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Consigue el [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de github de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
