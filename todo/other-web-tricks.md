# Ostale Web Trikove

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks merch**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### Host header

Često se back-end oslanja na **Host header** da bi izvršio određene akcije. Na primer, može koristiti njegovu vrednost kao **domen za slanje resetovanja lozinke**. Dakle, kada primite email sa linkom za resetovanje lozinke, domen koji se koristi je onaj koji ste uneli u Host header. Zatim, možete zatražiti resetovanje lozinke drugih korisnika i promeniti domen u jedan koji kontrolišete kako biste ukrali njihove kodove za resetovanje lozinke. [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
Imajte na umu da možda čak i ne morate čekati da korisnik klikne na link za resetovanje lozinke da biste dobili token, jer možda čak i **spam filteri ili drugi posrednički uređaji/botovi će kliknuti na njega da ga analiziraju**.
{% endhint %}

### Session booleans

Ponekad kada uspešno završite neku verifikaciju, back-end će **samo dodati boolean sa vrednošću "True" atributu vaše sesije**. Zatim, drugi endpoint će znati da li ste uspešno prošli tu proveru.\
Međutim, ako **prođete proveru** i vašoj sesiji je dodeljena vrednost "True" u sigurnosnom atributu, možete pokušati **pristupiti drugim resursima** koji **zavise od istog atributa** ali na koje **ne biste trebali imati dozvole** za pristup. [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### Funkcionalnost registracije

Pokušajte da se registrujete kao već postojeći korisnik. Pokušajte takođe koristeći ekvivalentne karaktere (tačke, mnogo razmaka i Unicode).

### Preuzimanje emailova

Registrujte email, pre nego što ga potvrdite promenite email, zatim, ako se nova potvrda emaila šalje na prvi registrovani email, možete preuzeti bilo koji email. Ili ako možete omogućiti drugi email potvrđujući prvi, takođe možete preuzeti bilo koji nalog.

### Pristup internom servisnom stolu kompanija koje koriste atlassian

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### TRACE metoda

Programeri ponekad zaborave da onemoguće različite opcije za debagovanje u produkcionom okruženju. Na primer, HTTP `TRACE` metoda je dizajnirana za dijagnostičke svrhe. Ako je omogućena, web server će odgovoriti na zahteve koji koriste `TRACE` metodu tako što će u odgovoru odjeknuti tačan zahtev koji je primljen. Ovo ponašanje često nije štetno, ali ponekad dovodi do otkrivanja informacija, kao što su naziv internih autentikacionih zaglavlja koja mogu biti dodata zahtevima od strane reverznih proxy-ja.![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)
