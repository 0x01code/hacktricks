# Vifurushi vya Kernel vya macOS

<details>

<summary><strong>Jifunze AWS hacking kutoka sifuri hadi shujaa na</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Mtaalam wa Timu Nyekundu ya AWS ya HackTricks)</strong></a><strong>!</strong></summary>

* Je, unafanya kazi katika **kampuni ya usalama wa mtandao**? Je, ungependa kuona **kampuni yako ikionyeshwa kwenye HackTricks**? Au ungependa kupata ufikiaji wa **toleo la hivi karibuni la PEASS au kupakua HackTricks kwa PDF**? Angalia [**MIPANGO YA USAJILI**](https://github.com/sponsors/carlospolop)!
* Gundua [**Familia ya PEASS**](https://opensea.io/collection/the-peass-family), mkusanyiko wetu maalum wa [**NFTs**](https://opensea.io/collection/the-peass-family)
* Pata [**swag rasmi wa PEASS na HackTricks**](https://peass.creator-spring.com)
* **Jiunge na** [**💬**](https://emojipedia.org/speech-balloon/) **kikundi cha Discord** au kwenye [**kikundi cha telegram**](https://t.me/peass) au **nifuata** kwenye **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Shiriki mbinu zako za udukuzi kwa kutuma PR kwa** [**repo ya hacktricks**](https://github.com/carlospolop/hacktricks) **na** [**repo ya hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Taarifa Msingi

Vifurushi vya Kernel (Kexts) ni **vifurushi** vyenye kielezo cha **`.kext`** ambavyo **hupakiwa moja kwa moja katika nafasi ya kernel ya macOS**, kutoa utendaji wa ziada kwa mfumo wa uendeshaji kuu.

### Mahitaji

Kwa wazi, hii ni **nguvu sana hivyo ni vigumu kupakia kifurushi cha kernel**. Hizi ni **mahitaji** ambayo kifurushi cha kernel lazima kiyakidhi ili kipakiwe:

* Wakati wa **kuingia kwenye hali ya kupona**, vifurushi vya kernel **lazima viweze kupakiwa**:
  
<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Kifurushi cha kernel lazima kiwe **kimesainiwa na cheti cha usaini wa nambari ya kernel**, ambacho kinaweza kupewa tu na **Apple**. Ambayo itakagua kwa undani kampuni na sababu kwa nini inahitajika.
* Kifurushi cha kernel lazima pia kiwe **kimethibitishwa**, Apple itaweza kukagua kwa zisizo za programu hasidi.
* Kisha, mtumiaji wa **root** ndiye anayeweza **kupakia kifurushi cha kernel** na faili ndani ya kifurushi hicho lazima **ziwe mali ya root**.
* Wakati wa mchakato wa kupakia, kifurushi lazima kiwe tayari katika eneo la **ulinzi lisilo la root**: `/Library/StagedExtensions` (inahitaji idhini ya `com.apple.rootless.storage.KernelExtensionManagement`).
* Hatimaye, wakati wa kujaribu kupakia, mtumiaji atapokea [**ombi la uthibitisho**](https://developer.apple.com/library/archive/technotes/tn2459/\_index.html) na, ikiwa itakubaliwa, kompyuta lazima **izimishwe** ili kuipakia.

### Mchakato wa Upakiaji

Katika Catalina ilikuwa hivi: Ni muhimu kufahamu kuwa mchakato wa **uthibitisho** unatokea katika **userland**. Walakini, programu tu zenye idhini ya **`com.apple.private.security.kext-management`** ndizo zinaweza **kuomba kernel kupakia kifurushi**: `kextcache`, `kextload`, `kextutil`, `kextd`, `syspolicyd`

1. **`kextutil`** cli **inaanza** mchakato wa **uthibitisho** wa kupakia kifurushi
* Itazungumza na **`kextd`** kwa kutuma kutumia **huduma ya Mach**.
2. **`kextd`** itachunguza mambo kadhaa, kama vile **saini**
* Itazungumza na **`syspolicyd`** ili **kuthibitisha** ikiwa kifurushi kinaweza **kupakiwa**.
3. **`syspolicyd`** itamwomba **mtumiaji** ikiwa kifurushi hakijapakiwa hapo awali.
* **`syspolicyd`** itaripoti matokeo kwa **`kextd`**
4. **`kextd`** hatimaye itaweza **kuambia kernel kupakia** kifurushi

Ikiwa **`kextd`** haipatikani, **`kextutil`** inaweza kufanya ukaguzi sawa.

## Marejeo

* [https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/](https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/)
* [https://www.youtube.com/watch?v=hGKOskSiaQo](https://www.youtube.com/watch?v=hGKOskSiaQo)

<details>

<summary><strong>Jifunze AWS hacking kutoka sifuri hadi shujaa na</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Mtaalam wa Timu Nyekundu ya AWS ya HackTricks)</strong></a><strong>!</strong></summary>

* Je, unafanya kazi katika **kampuni ya usalama wa mtandao**? Je, ungependa kuona **kampuni yako ikionyeshwa kwenye HackTricks**? Au ungependa kupata ufikiaji wa **toleo la hivi karibuni la PEASS au kupakua HackTricks kwa PDF**? Angalia [**MIPANGO YA USAJILI**](https://github.com/sponsors/carlospolop)!
* Gundua [**Familia ya PEASS**](https://opensea.io/collection/the-peass-family), mkusanyiko wetu maalum wa [**NFTs**](https://opensea.io/collection/the-peass-family)
* Pata [**swag rasmi wa PEASS na HackTricks**](https://peass.creator-spring.com)
* **Jiunge na** [**💬**](https://emojipedia.org/speech-balloon/) **kikundi cha Discord** au kwenye [**kikundi cha telegram**](https://t.me/peass) au **nifuata** kwenye **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Shiriki mbinu zako za udukuzi kwa kutuma PR kwa** [**repo ya hacktricks**](https://github.com/carlospolop/hacktricks) **na** [**repo ya hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
