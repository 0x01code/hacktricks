# macOS Geheue Dumping

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>

## WhiteIntel

<figure><img src=".gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is 'n **donkerweb**-aangedrewe soekenjin wat **gratis** funksies bied om te kontroleer of 'n maatskappy of sy kliënte deur **steel-malware** gekompromitteer is.

Die primêre doel van WhiteIntel is om rekening-oorneemings en lospryse-aanvalle te beveg wat voortspruit uit inligtingsteel-malware.

Jy kan hul webwerf besoek en hul enjin **gratis** probeer by:

{% embed url="https://whiteintel.io" %}

---

## Geheue Artefakte

### Ruil lêers

Ruil lêers, soos `/private/var/vm/swapfile0`, dien as **kasgeheue wanneer die fisiese geheue vol is**. Wanneer daar geen spasie meer in fisiese geheue is nie, word die data na 'n ruil lêer oorgedra en dan weer na fisiese geheue gebring as dit nodig is. Verskeie ruil lêers kan teenwoordig wees, met name soos swapfile0, swapfile1, ensovoorts.

### Hiberneerbeeld

Die lêer wat geleë is by `/private/var/vm/sleepimage` is noodsaaklik tydens **hibernasiemodus**. **Data vanaf geheue word in hierdie lêer gestoor wanneer OS X hiberneer**. Met die ontwaking van die rekenaar haal die stelsel geheuedata uit hierdie lêer, wat die gebruiker in staat stel om voort te gaan waar hulle opgehou het.

Dit is die moeite werd om op te let dat op moderne MacOS-stelsels hierdie lêer tipies vir sekuriteitsredes versleutel is, wat herwinning moeilik maak.

* Om te kontroleer of versleuteling geaktiveer is vir die sleepimage, kan die opdrag `sysctl vm.swapusage` uitgevoer word. Dit sal wys of die lêer versleutel is.

### Geheuedruklogs

'n Ander belangrike geheue-verwante lêer in MacOS-stelsels is die **geheuedruklog**. Hierdie logboeke is geleë in `/var/log` en bevat gedetailleerde inligting oor die stelsel se geheuegebruik en drukgebeure. Dit kan veral nuttig wees vir die diagnose van geheue-verwante probleme of om te verstaan hoe die stelsel oor tyd geheue bestuur.

## Geheue dump met osxpmem

Om die geheue in 'n MacOS-rekenaar te dump, kan jy [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip) gebruik.

**Nota**: Die volgende instruksies sal slegs werk vir Macs met Intel-argitektuur. Hierdie instrument is nou gearchiveer en die laaste vrystelling was in 2017. Die binêre lêer wat met die instruksies hieronder afgelaai is, teiken Intel-skyfies omdat Apple Silicon nie in 2017 bestaan het nie. Dit mag moontlik wees om die binêre lêer vir arm64-argitektuur te kompileer, maar jy sal dit self moet probeer.
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
Indien jy hierdie fout vind: `osxpmem.app/MacPmem.kext kon nie laai nie - (libkern/kext) verifikasie mislukking (lêer eienaarskap/permisies); kyk na die stelsel/kernel logboeke vir foute of probeer kextutil(8)` Jy kan dit regmaak deur:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**Ander foute** kan dalk reggestel word deur **die laai van die kext** in "Sekuriteit & Privaatheid --> Algemeen", net **staan** dit toe.

Jy kan ook hierdie **eenlynige kode** gebruik om die aansoek af te laai, die kext te laai en die geheue te dump:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

## WhiteIntel

<figure><img src=".gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is 'n **dark-web** aangedrewe soekenjin wat **gratis** funksies bied om te kontroleer of 'n maatskappy of sy kliënte **gekompromiteer** is deur **stealer malware**.

Hul primêre doel van WhiteIntel is om rekening-oorneemings en lospryse-aanvalle te beveg wat voortspruit uit inligtingsteel-malware.

Jy kan hul webwerf besoek en hul enjin **gratis** probeer by:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>
