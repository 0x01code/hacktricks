# Phishing-Methodologie

<details>

<summary><strong>Lernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks im PDF-Format herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegramm-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) Github-Repositorys einreichen.

</details>

## Methodologie

1. Erkunden Sie das Opfer
1. Wählen Sie die **Opferdomäne** aus.
2. Führen Sie eine grundlegende Web-Enumeration durch, **suchen Sie nach Login-Portalen**, die vom Opfer verwendet werden, und **entscheiden Sie**, welches Sie **imitieren** werden.
3. Verwenden Sie etwas **OSINT**, um **E-Mails zu finden**.
2. Bereiten Sie die Umgebung vor
1. **Kaufen Sie die Domain**, die Sie für die Phishing-Bewertung verwenden werden
2. **Konfigurieren Sie die E-Mail-Dienst-bezogenen Einträge** (SPF, DMARC, DKIM, rDNS)
3. Konfigurieren Sie den VPS mit **gophish**
3. Bereiten Sie die Kampagne vor
1. Bereiten Sie die **E-Mail-Vorlage** vor
2. Bereiten Sie die **Webseite** vor, um die Anmeldedaten zu stehlen
4. Starten Sie die Kampagne!

## Generieren Sie ähnliche Domainnamen oder kaufen Sie eine vertrauenswürdige Domain

### Techniken zur Variation von Domainnamen

* **Schlüsselwort**: Der Domainname enthält ein wichtiges Schlüsselwort der Originaldomain (z. B. zelster.com-management.com).
* **Bindestrich-Subdomäne**: Ändern Sie den **Punkt gegen einen Bindestrich** einer Subdomäne (z. B. www-zelster.com).
* **Neue TLD**: Gleiche Domain mit einer **neuen TLD** (z. B. zelster.org)
* **Homoglyph**: Es ersetzt einen Buchstaben im Domainnamen durch **Buchstaben, die ähnlich aussehen** (z. B. zelfser.com).
* **Transposition:** Es **tauscht zwei Buchstaben** innerhalb des Domainnamens aus (z. B. zelsetr.com).
* **Singularisierung/Pluralisierung**: Fügt am Ende des Domainnamens ein "s" hinzu oder entfernt es (z. B. zeltsers.com).
* **Auslassung**: Es **entfernt einen** der Buchstaben aus dem Domainnamen (z. B. zelser.com).
* **Wiederholung**: Es **wiederholt einen** der Buchstaben im Domainnamen (z. B. zeltsser.com).
* **Ersetzung**: Ähnlich wie Homoglyph, aber weniger unauffällig. Es ersetzt einen der Buchstaben im Domainnamen, möglicherweise durch einen Buchstaben in der Nähe des Originalbuchstabens auf der Tastatur (z. B. zektser.com).
* **Subdomained**: Fügen Sie einen **Punkt** innerhalb des Domainnamens ein (z. B. ze.lster.com).
* **Einfügung**: Es **fügt einen Buchstaben** in den Domainnamen ein (z. B. zerltser.com).
* **Fehlender Punkt**: Hängen Sie die TLD an den Domainnamen an. (z. B. zelstercom.com)

**Automatische Tools**

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy)

**Websites**

* [https://dnstwist.it/](https://dnstwist.it)
* [https://dnstwister.report/](https://dnstwister.report)
* [https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/](https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/)

### Bitflipping

Es besteht die **Möglichkeit, dass eines von einigen gespeicherten oder übermittelten Bits automatisch umgedreht wird**, aufgrund verschiedener Faktoren wie Sonneneruptionen, kosmische Strahlen oder Hardwarefehler.

Wenn dieses Konzept auf DNS-Anfragen angewendet wird, ist es möglich, dass die vom DNS-Server empfangene **Domain nicht mit der ursprünglich angeforderten Domain übereinstimmt**.

Beispielsweise kann eine einzelne Bit-Änderung in der Domain "windows.com" diese in "windnws.com" ändern.

Angreifer können dies ausnutzen, indem sie mehrere Bit-Flipping-Domains registrieren, die der Domain des Opfers ähnlich sind. Ihr Ziel ist es, legitime Benutzer auf ihre eigene Infrastruktur umzuleiten.

Für weitere Informationen lesen Sie [https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/](https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/)

### Kauf einer vertrauenswürdigen Domain

Sie können auf [https://www.expireddomains.net/](https://www.expireddomains.net) nach einer abgelaufenen Domain suchen, die Sie verwenden könnten.\
Um sicherzustellen, dass die abgelaufene Domain, die Sie kaufen möchten, **bereits ein gutes SEO hat**, können Sie überprüfen, wie sie kategorisiert ist in:

* [http://www.fortiguard.com/webfilter](http://www.fortiguard.com/webfilter)
* [https://urlfiltering.paloaltonetworks.com/query/](https://urlfiltering.paloaltonetworks.com/query/)

## E-Mails entdecken

* [https://github.com/laramies/theHarvester](https://github.com/laramies/theHarvester) (100% kostenlos)
* [https://phonebook.cz/](https://phonebook.cz) (100% kostenlos)
* [https://maildb.io/](https://maildb.io)
* [https://hunter.io/](https://hunter.io)
* [https://anymailfinder.com/](https://anymailfinder.com)

Um weitere gültige E-Mail-Adressen zu entdecken oder diejenigen zu überprüfen, die Sie bereits entdeckt haben, können Sie versuchen, die SMTP-Server des Opfers per Brute-Force zu überprüfen. [Erfahren Sie hier, wie Sie E-Mail-Adressen verifizieren/entdecken können](../../network-services-pentesting/pentesting-smtp/#username-bruteforce-enumeration).\
Vergessen Sie außerdem nicht, dass Sie, wenn die Benutzer ein **beliebiges Webportal zum Zugriff auf ihre E-Mails verwenden**, überprüfen können, ob es anfällig für **Benutzernamen-Brute-Force** ist, und die Schwachstelle bei Bedarf ausnutzen können.

## Konfigurieren von GoPhish

### Installation

Sie können es von [https://github.com/gophish/gophish/releases/tag/v0.11.0](https://github.com/gophish/gophish/releases/tag/v0.11.0) herunterladen

Laden Sie es herunter und entpacken Sie es in `/opt/gophish` und führen Sie `/opt/gophish/gophish` aus\
Sie erhalten ein Passwort für den Admin-Benutzer auf Port 3333 in der Ausgabe. Greifen Sie daher auf diesen Port zu und verwenden Sie diese Anmeldeinformationen, um das Admin-Passwort zu ändern. Möglicherweise müssen Sie diesen Port lokal tunneln:
```bash
ssh -L 3333:127.0.0.1:3333 <user>@<ip>
```
### Konfiguration

**TLS-Zertifikatkonfiguration**

Vor diesem Schritt sollten Sie bereits die **Domain gekauft** haben, die Sie verwenden werden, und sie muss auf die **IP des VPS** zeigen, auf dem Sie **gophish** konfigurieren.
```bash
DOMAIN="<domain>"
wget https://dl.eff.org/certbot-auto
chmod +x certbot-auto
sudo apt install snapd
sudo snap install core
sudo snap refresh core
sudo apt-get remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
certbot certonly --standalone -d "$DOMAIN"
mkdir /opt/gophish/ssl_keys
cp "/etc/letsencrypt/live/$DOMAIN/privkey.pem" /opt/gophish/ssl_keys/key.pem
cp "/etc/letsencrypt/live/$DOMAIN/fullchain.pem" /opt/gophish/ssl_keys/key.crt​
```
**E-Mail-Konfiguration**

Beginnen Sie mit der Installation: `apt-get install postfix`

Fügen Sie dann die Domain zu den folgenden Dateien hinzu:

* **/etc/postfix/virtual\_domains**
* **/etc/postfix/transport**
* **/etc/postfix/virtual\_regexp**

**Ändern Sie auch die Werte der folgenden Variablen in /etc/postfix/main.cf**

`myhostname = <domain>`\
`mydestination = $myhostname, <domain>, localhost.com, localhost`

Ändern Sie schließlich die Dateien **`/etc/hostname`** und **`/etc/mailname`** auf Ihren Domainnamen und **starten Sie Ihren VPS neu.**

Erstellen Sie nun einen **DNS-A-Record** von `mail.<domain>`, der auf die **IP-Adresse** des VPS zeigt, und einen **DNS-MX-Record**, der auf `mail.<domain>` zeigt.

Lassen Sie uns nun testen, ob eine E-Mail gesendet werden kann:
```bash
apt install mailutils
echo "This is the body of the email" | mail -s "This is the subject line" test@email.com
```
**Gophish Konfiguration**

Stoppen Sie die Ausführung von Gophish und konfigurieren Sie es.\
Ändern Sie `/opt/gophish/config.json` wie folgt (beachten Sie die Verwendung von https):
```bash
{
"admin_server": {
"listen_url": "127.0.0.1:3333",
"use_tls": true,
"cert_path": "gophish_admin.crt",
"key_path": "gophish_admin.key"
},
"phish_server": {
"listen_url": "0.0.0.0:443",
"use_tls": true,
"cert_path": "/opt/gophish/ssl_keys/key.crt",
"key_path": "/opt/gophish/ssl_keys/key.pem"
},
"db_name": "sqlite3",
"db_path": "gophish.db",
"migrations_prefix": "db/db_",
"contact_address": "",
"logging": {
"filename": "",
"level": ""
}
}
```
**Konfigurieren des Gophish-Dienstes**

Um den Gophish-Dienst zu erstellen, damit er automatisch gestartet und als Dienst verwaltet werden kann, können Sie die Datei `/etc/init.d/gophish` mit folgendem Inhalt erstellen:
```bash
#!/bin/bash
# /etc/init.d/gophish
# initialization file for stop/start of gophish application server
#
# chkconfig: - 64 36
# description: stops/starts gophish application server
# processname:gophish
# config:/opt/gophish/config.json
# From https://github.com/gophish/gophish/issues/586

# define script variables

processName=Gophish
process=gophish
appDirectory=/opt/gophish
logfile=/var/log/gophish/gophish.log
errfile=/var/log/gophish/gophish.error

start() {
echo 'Starting '${processName}'...'
cd ${appDirectory}
nohup ./$process >>$logfile 2>>$errfile &
sleep 1
}

stop() {
echo 'Stopping '${processName}'...'
pid=$(/bin/pidof ${process})
kill ${pid}
sleep 1
}

status() {
pid=$(/bin/pidof ${process})
if [["$pid" != ""| "$pid" != "" ]]; then
echo ${processName}' is running...'
else
echo ${processName}' is not running...'
fi
}

case $1 in
start|stop|status) "$1" ;;
esac
```
Beenden Sie die Konfiguration des Dienstes und überprüfen Sie diese durch:
```bash
mkdir /var/log/gophish
chmod +x /etc/init.d/gophish
update-rc.d gophish defaults
#Check the service
service gophish start
service gophish status
ss -l | grep "3333\|443"
service gophish stop
```
## Konfiguration des Mail-Servers und der Domain

### Warten und legitim sein

Je älter eine Domain ist, desto unwahrscheinlicher ist es, dass sie als Spam erkannt wird. Daher sollten Sie so lange wie möglich warten (mindestens 1 Woche), bevor Sie die Phishing-Bewertung durchführen. Darüber hinaus wird die Reputation besser, wenn Sie eine Seite über einen reputablen Sektor erstellen.

Beachten Sie, dass Sie trotz einer Wartezeit von einer Woche jetzt alles konfigurieren können.

### Reverse DNS (rDNS) Eintrag konfigurieren

Legen Sie einen rDNS (PTR) Eintrag fest, der die IP-Adresse des VPS in den Domainnamen auflöst.

### Sender Policy Framework (SPF) Eintrag

Sie müssen **einen SPF-Eintrag für die neue Domain konfigurieren**. Wenn Sie nicht wissen, was ein SPF-Eintrag ist, [**lesen Sie diese Seite**](../../network-services-pentesting/pentesting-smtp/#spf).

Sie können [https://www.spfwizard.net/](https://www.spfwizard.net) verwenden, um Ihre SPF-Richtlinie zu generieren (verwenden Sie die IP des VPS).

![](<../../.gitbook/assets/image (388).png>)

Dies ist der Inhalt, der innerhalb eines TXT-Eintrags in der Domain festgelegt werden muss:
```bash
v=spf1 mx a ip4:ip.ip.ip.ip ?all
```
### Domain-based Message Authentication, Reporting & Conformance (DMARC) Record

Sie müssen **einen DMARC-Eintrag für die neue Domain konfigurieren**. Wenn Sie nicht wissen, was ein DMARC-Eintrag ist, [**lesen Sie diese Seite**](../../network-services-pentesting/pentesting-smtp/#dmarc).

Sie müssen einen neuen DNS-TXT-Eintrag erstellen, der den Hostnamen `_dmarc.<domain>` auf den folgenden Inhalt verweist:
```bash
v=DMARC1; p=none
```
### DomainKeys Identified Mail (DKIM)

Sie müssen **einen DKIM für die neue Domain konfigurieren**. Wenn Sie nicht wissen, was ein DMARC-Eintrag ist, [**lesen Sie diese Seite**](../../network-services-pentesting/pentesting-smtp/#dkim).

Dieses Tutorial basiert auf: [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)

{% hint style="info" %}
Sie müssen beide B64-Werte, die der DKIM-Schlüssel generiert, verketten:
```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0wPibdqPtzYk81njjQCrChIcHzxOp8a1wjbsoNtka2X9QXCZs+iXkvw++QsWDtdYu3q0Ofnr0Yd/TmG/Y2bBGoEgeE+YTUG2aEgw8Xx42NLJq2D1pB2lRQPW4IxefROnXu5HfKSm7dyzML1gZ1U0pR5X4IZCH0wOPhIq326QjxJZm79E1nTh3xj" "Y9N/Dt3+fVnIbMupzXE216TdFuifKM6Tl6O/axNsbswMS1TH812euno8xRpsdXJzFlB9q3VbMkVWig4P538mHolGzudEBg563vv66U8D7uuzGYxYT4WS8NVm3QBMg0QKPWZaKp+bADLkOSB9J2nUpk4Aj9KB5swIDAQAB
```
{% endhint %}

### Testen Sie Ihren E-Mail-Konfigurationsscore

Sie können das mit [https://www.mail-tester.com/](https://www.mail-tester.com) tun.\
Greifen Sie einfach auf die Seite zu und senden Sie eine E-Mail an die Ihnen gegebene Adresse:
```bash
echo "This is the body of the email" | mail -s "This is the subject line" test-iimosa79z@srv1.mail-tester.com
```
Sie können auch **Ihre E-Mail-Konfiguration überprüfen**, indem Sie eine E-Mail an `check-auth@verifier.port25.com` senden und **die Antwort lesen** (dafür müssen Sie den Port **25 öffnen** und die Antwort in der Datei _/var/mail/root_ sehen, wenn Sie die E-Mail als root senden).\
Überprüfen Sie, dass Sie alle Tests bestehen:
```bash
==========================================================
Summary of Results
==========================================================
SPF check:          pass
DomainKeys check:   neutral
DKIM check:         pass
Sender-ID check:    pass
SpamAssassin check: ham
```
Sie könnten auch eine **Nachricht an ein Gmail-Konto unter Ihrer Kontrolle senden** und die **Header der E-Mail** in Ihrem Gmail-Posteingang überprüfen. `dkim=pass` sollte im Feld `Authentication-Results` des Headers vorhanden sein.
```
Authentication-Results: mx.google.com;
spf=pass (google.com: domain of contact@example.com designates --- as permitted sender) smtp.mail=contact@example.com;
dkim=pass header.i=@example.com;
```
### Entfernen aus der Spamhaus-Blacklist

Die Seite [www.mail-tester.com](www.mail-tester.com) kann Ihnen anzeigen, ob Ihre Domain von Spamhaus blockiert wird. Sie können die Entfernung Ihrer Domain/IP unter [https://www.spamhaus.org/lookup/](https://www.spamhaus.org/lookup/) beantragen.

### Entfernen aus der Microsoft-Blacklist

Sie können die Entfernung Ihrer Domain/IP unter [https://sender.office.com/](https://sender.office.com) beantragen.

## GoPhish-Kampagne erstellen und starten

### Versandprofil

* Geben Sie einen **Namen zur Identifizierung** des Absenderprofils ein.
* Entscheiden Sie, von welchem Konto aus Sie die Phishing-E-Mails senden möchten. Vorschläge: _noreply, support, servicedesk, salesforce..._
* Sie können den Benutzernamen und das Passwort leer lassen, aber stellen Sie sicher, dass Sie die Option "Zertifikatsfehler ignorieren" aktivieren.

![](<../../.gitbook/assets/image (253) (1) (2) (1) (1) (2) (2) (3) (3) (5) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (17).png>)

{% hint style="info" %}
Es wird empfohlen, die Funktion "**Test-E-Mail senden**" zu verwenden, um zu überprüfen, ob alles funktioniert.\
Ich empfehle, die **Test-E-Mails an 10min-E-Mail-Adressen zu senden**, um zu vermeiden, dass Sie beim Testen auf die Blacklist geraten.
{% endhint %}

### E-Mail-Vorlage

* Geben Sie einen **Namen zur Identifizierung** der Vorlage ein.
* Schreiben Sie dann einen **Betreff** (nichts Seltsames, nur etwas, das Sie in einer regulären E-Mail erwarten würden).
* Stellen Sie sicher, dass Sie "**Tracking-Bild hinzufügen**" aktiviert haben.
* Schreiben Sie die **E-Mail-Vorlage** (Sie können Variablen wie im folgenden Beispiel verwenden):
```markup
<html>
<head>
<title></title>
</head>
<body>
<p class="MsoNormal"><span style="font-size:10.0pt;font-family:&quot;Verdana&quot;,sans-serif;color:black">Dear {{.FirstName}} {{.LastName}},</span></p>
<br />
Note: We require all user to login an a very suspicios page before the end of the week, thanks!<br />
<br />
Regards,</span></p>

WRITE HERE SOME SIGNATURE OF SOMEONE FROM THE COMPANY

<p>{{.Tracker}}</p>
</body>
</html>
```
Hinweis: **Um die Glaubwürdigkeit der E-Mail zu erhöhen**, wird empfohlen, eine Signatur aus einer E-Mail des Kunden zu verwenden. Vorschläge:

* Senden Sie eine E-Mail an eine **nicht existierende Adresse** und überprüfen Sie, ob die Antwort eine Signatur enthält.
* Suchen Sie nach **öffentlichen E-Mails** wie info@ex.com oder press@ex.com oder public@ex.com und senden Sie ihnen eine E-Mail und warten Sie auf die Antwort.
* Versuchen Sie, **eine gültig entdeckte** E-Mail zu kontaktieren und warten Sie auf die Antwort.

![](<../../.gitbook/assets/image (393).png>)

{% hint style="info" %}
Die E-Mail-Vorlage ermöglicht auch das **Anhängen von Dateien zum Senden**. Wenn Sie auch NTLM-Herausforderungen stehlen möchten, indem Sie speziell erstellte Dateien/Dokumente verwenden, [lesen Sie diese Seite](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md).
{% endhint %}

### Zielseite

* Schreiben Sie einen **Namen**
* **Schreiben Sie den HTML-Code** der Webseite. Beachten Sie, dass Sie Webseiten **importieren** können.
* Markieren Sie **Eingereichte Daten erfassen** und **Passwörter erfassen**
* Legen Sie eine **Weiterleitung** fest

![](<../../.gitbook/assets/image (394).png>)

{% hint style="info" %}
Normalerweise müssen Sie den HTML-Code der Seite anpassen und lokale Tests durchführen (vielleicht mit einem Apache-Server), **bis Ihnen das Ergebnis gefällt**. Schreiben Sie dann diesen HTML-Code in das Feld.\
Beachten Sie, dass Sie, wenn Sie **statische Ressourcen** für das HTML benötigen (vielleicht einige CSS- und JS-Seiten), diese unter _**/opt/gophish/static/endpoint**_ speichern und dann von _**/static/\<filename>**_ darauf zugreifen können.
{% endhint %}

{% hint style="info" %}
Für die Weiterleitung könnten Sie die Benutzer auf die legitime Hauptwebseite des Opfers **weiterleiten** oder sie z. B. auf _/static/migration.html_ weiterleiten, einen **drehenden Kreis (**[**https://loading.io/**](https://loading.io)**) für 5 Sekunden anzeigen und dann angeben, dass der Vorgang erfolgreich war**.
{% endhint %}

### Benutzer & Gruppen

* Geben Sie einen Namen ein
* **Importieren Sie die Daten** (beachten Sie, dass Sie für die Verwendung der Vorlage für das Beispiel den Vornamen, Nachnamen und die E-Mail-Adresse jedes Benutzers benötigen)

![](<../../.gitbook/assets/image (395).png>)

### Kampagne

Erstellen Sie schließlich eine Kampagne, indem Sie einen Namen, die E-Mail-Vorlage, die Zielseite, die URL, das Sendeprofil und die Gruppe auswählen. Beachten Sie, dass die URL der Link ist, der an die Opfer gesendet wird.

Beachten Sie, dass das **Sendeprofil es ermöglicht, eine Test-E-Mail zu senden, um zu sehen, wie die endgültige Phishing-E-Mail aussieht**:

![](<../../.gitbook/assets/image (396).png>)

{% hint style="info" %}
Ich würde empfehlen, die Test-E-Mails an 10min-E-Mail-Adressen zu senden, um zu vermeiden, dass Sie beim Testen auf die schwarze Liste gesetzt werden.
{% endhint %}

Sobald alles bereit ist, starten Sie einfach die Kampagne!

## Website-Klonen

Wenn Sie aus irgendeinem Grund die Website klonen möchten, überprüfen Sie die folgende Seite:

{% content-ref url="clone-a-website.md" %}
[clone-a-website.md](clone-a-website.md)
{% endcontent-ref %}

## Mit Backdoor versehene Dokumente & Dateien

In einigen Phishing-Bewertungen (hauptsächlich für Red Teams) möchten Sie auch **Dateien senden, die eine Art Backdoor enthalten** (vielleicht eine C2 oder einfach etwas, das eine Authentifizierung auslöst).\
Schauen Sie sich die folgende Seite für einige Beispiele an:

{% content-ref url="phishing-documents.md" %}
[phishing-documents.md](phishing-documents.md)
{% endcontent-ref %}

## Phishing MFA

### Über Proxy MitM

Der vorherige Angriff ist ziemlich clever, da Sie eine echte Website vortäuschen und die vom Benutzer festgelegten Informationen sammeln. Leider, wenn der Benutzer das richtige Passwort nicht eingegeben hat oder wenn die von Ihnen gefälschte Anwendung mit 2FA konfiguriert ist, **ermöglichen Ihnen diese Informationen nicht, den getäuschten Benutzer zu imitieren**.

Hier kommen Tools wie [**evilginx2**](https://github.com/kgretzky/evilginx2)**,** [**CredSniper**](https://github.com/ustayready/CredSniper) und [**muraena**](https://github.com/muraenateam/muraena) ins Spiel. Dieses Tool ermöglicht es Ihnen, einen MitM-Angriff zu generieren. Grundsätzlich funktionieren die Angriffe folgendermaßen:

1. Sie **imitieren das Anmeldeformular** der echten Webseite.
2. Der Benutzer **sendet** seine **Anmeldeinformationen** an Ihre gefälschte Seite und das Tool sendet sie an die echte Webseite, **überprüft, ob die Anmeldeinformationen funktionieren**.
3. Wenn das Konto mit **2FA** konfiguriert ist, wird die MitM-Seite danach fragen, und sobald der **Benutzer es eingibt**, sendet das Tool es an die echte Webseite.
4. Sobald der Benutzer authentifiziert ist, haben Sie (als Angreifer) die **Anmeldeinformationen, die 2FA, das Cookie und alle Informationen** jeder Interaktion erfasst, während das Tool einen MitM durchführt.

### Über VNC

Was ist, wenn Sie anstelle des **Opfers auf eine bösartige Seite zu leiten**, die genauso aussieht wie die Originalseite, ihn zu einer **VNC-Sitzung mit einem Browser verbinden, der mit der echten Webseite verbunden ist**? Sie können sehen, was er tut, das Passwort stehlen, die verwendete MFA, die Cookies...\
Dies können Sie mit [**EvilnVNC**](https://github.com/JoelGMSec/EvilnoVNC) tun

## Die Entdeckung der Entdeckung

Offensichtlich ist eine der besten Möglichkeiten zu wissen, ob Sie erwischt wurden, **Ihre Domain in Blacklists zu suchen**. Wenn sie dort aufgeführt ist, wurde Ihre Domain irgendwie als verdächtig erkannt.\
Eine einfache Möglichkeit zu überprüfen, ob Ihre Domain in einer Blacklist erscheint, ist die Verwendung von [https://malwareworld.com/](https://malwareworld.com)

Es gibt jedoch andere Möglichkeiten zu wissen, ob das Opfer **aktiv nach verdächtiger Phishing-Aktivität im Internet sucht**, wie in:

{% content-ref url="detecting-phising.md" %}
[detecting-phising.md](detecting-phising.md)
{% endcontent-ref %}

Sie können eine Domain mit einem sehr ähnlichen Namen wie der Domain des Opfers **kaufen** und/oder ein Zertifikat für eine **Subdomain** einer von Ihnen kontrollierten Domain **erstellen**, die das **Schlüsselwort** der Domain des Opfers enthält. Wenn das **Opfer** eine Art von **DNS- oder HTTP-Interaktion** mit ihnen durchführt, werden Sie wissen, dass **er aktiv nach verdächtigen Domains sucht** und Sie sehr unauffällig sein müssen.

### Bewertung des Phishings

Verwenden Sie [**Phishious** ](https://github.com/Rices/Phishious), um zu bewerten, ob Ihre E-Mail im Spam-Ordner landen wird oder ob sie blockiert oder erfolgreich sein wird.

## Referenzen

* [https://zeltser.com/domain-name-variations-in-phishing/](https://zeltser.com/domain-name-variations-in-phishing/)
* [https://0xpatrik.com/phishing-domains/](https://0xpatrik.com/phishing-domains/)
* [https://darkbyte.net/robando-sesiones-y-bypasseando-2fa-con-evilnovnc/](https://darkbyte.net/robando-sesiones-y-bypasseando-2fa-con-evilnovnc/)
* [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)

<details>

<summary><strong>Erlernen Sie AWS-Hacking von Grund auf mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen** oder **HackTricks im PDF-Format herunterladen** möchten, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merch**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>
