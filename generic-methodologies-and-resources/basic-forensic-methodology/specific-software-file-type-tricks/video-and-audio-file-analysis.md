<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Manipulacija audio i video fajlovima** je osnovna tehnika u **CTF forenzičkim izazovima**, koristeći **steganografiju** i analizu metapodataka za skrivanje ili otkrivanje tajnih poruka. Alati poput **[mediainfo](https://mediaarea.net/en/MediaInfo)** i **`exiftool`** su neophodni za pregled metapodataka fajlova i identifikaciju tipova sadržaja.

Za audio izazove, **[Audacity](http://www.audacityteam.org/)** se ističe kao premijeran alat za pregled talasnih oblika i analizu spektrograma, neophodnih za otkrivanje teksta kodiranog u audio formatu. **[Sonic Visualiser](http://www.sonicvisualiser.org/)** se visoko preporučuje za detaljnu analizu spektrograma. **Audacity** omogućava manipulaciju audio zapisa poput usporavanja ili obrtanja traka radi otkrivanja skrivenih poruka. **[Sox](http://sox.sourceforge.net/)**, alat za komandnu liniju, odličan je za konvertovanje i uređivanje audio fajlova.

**Manipulacija najmanje značajnim bitovima (LSB)** je česta tehnika u audio i video steganografiji, iskorišćavajući fiksne delove medijskih fajlova za skriveno ugrađivanje podataka. **[Multimon-ng](http://tools.kali.org/wireless-attacks/multimon-ng)** je koristan za dekodiranje poruka skrivenih kao **DTMF tonovi** ili **Morseov kod**.

Video izazovi često uključuju kontejnerske formate koji objedinjuju audio i video tokove. **[FFmpeg](http://ffmpeg.org/)** je osnovni alat za analizu i manipulaciju ovih formata, sposoban za demultipleksiranje i reprodukciju sadržaja. Za programere, **[ffmpy](http://ffmpy.readthedocs.io/en/latest/examples.html)** integriše mogućnosti FFmpeg-a u Python za napredne skriptabilne interakcije.

Ova paleta alata ističe potrebnu fleksibilnost u CTF izazovima, gde učesnici moraju primeniti širok spektar tehnika analize i manipulacije kako bi otkrili skrivene podatke unutar audio i video fajlova.

## Reference
* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)


<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
