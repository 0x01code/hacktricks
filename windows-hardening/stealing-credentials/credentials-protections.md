# Protections des identifiants Windows

## Protections des identifiants

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## WDigest

Le protocole [WDigest](https://technet.microsoft.com/pt-pt/library/cc778868\(v=ws.10\).aspx?f=255\&MSPPError=-2147217396) a été introduit dans Windows XP et a été conçu pour être utilisé avec le protocole HTTP pour l'authentification. Microsoft a activé ce protocole **par défaut dans plusieurs versions de Windows** (Windows XP - Windows 8.0 et Windows Server 2003 - Windows Server 2012), ce qui signifie que **les mots de passe en texte clair sont stockés dans LSASS** (Local Security Authority Subsystem Service). **Mimikatz** peut interagir avec LSASS, permettant à un attaquant de **récupérer ces informations d'identification** grâce à la commande suivante :
```
sekurlsa::wdigest
```
Ce comportement peut être **désactivé/activé en définissant à 1** la valeur de _**UseLogonCredential**_ et _**Negotiate**_ dans _**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest**_.\
Si ces clés de registre **n'existent pas** ou que la valeur est **"0"**, alors WDigest sera **désactivé**.
```
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
## Protection LSA

Microsoft a fourni une protection supplémentaire pour LSA dans **Windows 8.1 et versions ultérieures** pour **empêcher** les processus non fiables de pouvoir **lire sa mémoire** ou d'injecter du code. Cela empêchera le fonctionnement correct de `mimikatz.exe sekurlsa:logonpasswords`.\
Pour **activer cette protection**, vous devez définir la valeur _**RunAsPPL**_ dans _**HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\LSA**_ sur 1.
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### Contournement

Il est possible de contourner cette protection en utilisant le pilote Mimikatz mimidrv.sys :

![](../../.gitbook/assets/mimidrv.png)

## Credential Guard

**Credential Guard** est une nouvelle fonctionnalité de Windows 10 (éditions Enterprise et Education) qui aide à protéger vos informations d'identification sur une machine contre des menaces telles que le pass the hash. Cela fonctionne grâce à une technologie appelée Virtual Secure Mode (VSM) qui utilise les extensions de virtualisation du CPU (mais ce n'est pas une machine virtuelle réelle) pour fournir une **protection aux zones de mémoire** (vous pouvez entendre cela appelé Sécurité basée sur la virtualisation ou VBS). VSM crée une "bulle" séparée pour les **processus** clés qui sont **isolés** des processus réguliers du **système d'exploitation**, même le noyau, et **seuls des processus de confiance spécifiques peuvent communiquer avec les processus** (connus sous le nom de **trustlets**) dans VSM. Cela signifie qu'un processus dans le système d'exploitation principal ne peut pas lire la mémoire de VSM, même les processus du noyau. L'**Autorité de sécurité locale (LSA) est l'un des trustlets** dans VSM en plus du processus standard **LSASS** qui s'exécute toujours dans le système d'exploitation principal pour assurer la prise en charge des processus existants, mais qui agit vraiment comme un proxy ou un stub pour communiquer avec la version dans VSM, en veillant à ce que les informations d'identification réelles s'exécutent sur la version dans VSM et soient donc protégées contre les attaques. Credential Guard doit être activé et déployé dans votre organisation car il n'est **pas activé par défaut**.\
À partir de [https://www.itprotoday.com/windows-10/what-credential-guard](https://www.itprotoday.com/windows-10/what-credential-guard)\
Plus d'informations et un script PS1 pour activer Credential Guard [peuvent être trouvés ici](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage).

Dans ce cas, **Mimikatz ne peut pas faire grand-chose pour contourner** cela et extraire les hachages de LSASS. Mais vous pouvez toujours ajouter votre **SSP personnalisé** et **capturer les informations d'identification** lorsqu'un utilisateur essaie de se connecter en **clair**.\
Plus d'informations sur [**SSP et comment le faire ici**](../active-directory-methodology/custom-ssp.md).

Credentials Guard peut être **activé de différentes manières**. Pour vérifier s'il a été activé en utilisant le registre, vous pouvez vérifier la valeur de la clé _**LsaCfgFlags**_ dans _**HKLM\System\CurrentControlSet\Control\LSA**_. Si la valeur est **"1"**, elle est active avec un verrou UEFI, si **"2"** est active sans verrou et si **"0"** elle n'est pas activée.\
Cela n'est **pas suffisant pour activer Credentials Guard** (mais c'est un indicateur fort).\
Plus d'informations et un script PS1 pour activer Credential Guard [peuvent être trouvés ici](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage).
```
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
## Mode RestreintAdmin pour RDP

Avec Windows 8.1 et Windows Server 2012 R2, de nouvelles fonctionnalités de sécurité ont été introduites. L'une de ces fonctionnalités de sécurité est le _mode RestreintAdmin pour RDP_. Cette nouvelle fonctionnalité de sécurité est introduite pour atténuer le risque d'attaques [pass the hash](https://blog.ahasayen.com/pass-the-hash/).

Lorsque vous vous connectez à un ordinateur distant en utilisant RDP, vos informations d'identification sont stockées sur l'ordinateur distant sur lequel vous vous connectez. Habituellement, vous utilisez un compte puissant pour vous connecter à des serveurs distants, et avoir vos informations d'identification stockées sur tous ces ordinateurs est en effet une menace pour la sécurité.

En utilisant le _mode RestreintAdmin pour RDP_, lorsque vous vous connectez à un ordinateur distant en utilisant la commande **mstsc.exe /RestrictedAdmin**, vous serez authentifié sur l'ordinateur distant, mais **vos informations d'identification ne seront pas stockées sur cet ordinateur distant**, comme cela aurait été le cas dans le passé. Cela signifie que si un logiciel malveillant ou même un utilisateur malveillant est actif sur ce serveur distant, vos informations d'identification ne seront pas disponibles sur ce serveur de bureau distant pour que le logiciel malveillant puisse les attaquer.

Notez que comme vos informations d'identification ne sont pas enregistrées dans la session RDP, si vous **essayez d'accéder à des ressources réseau**, vos informations d'identification ne seront pas utilisées. **L'identité de la machine sera utilisée à la place**.

![](../../.gitbook/assets/ram.png)

À partir de [ici](https://blog.ahasayen.com/restricted-admin-mode-for-rdp/).

## Informations d'identification mises en cache

Les **informations d'identification de domaine** sont utilisées par les composants du système d'exploitation et sont **authentifiées** par l'**Autorité de sécurité locale** (LSA). En général, les informations d'identification de domaine sont établies pour un utilisateur lorsqu'un package de sécurité enregistré authentifie les données de connexion de l'utilisateur. Ce package de sécurité enregistré peut être le protocole **Kerberos** ou **NTLM**.

**Windows stocke les dix dernières informations d'identification de connexion de domaine au cas où le contrôleur de domaine serait hors ligne**. Si le contrôleur de domaine est hors ligne, un utilisateur pourra **toujours se connecter à son ordinateur**. Cette fonctionnalité est principalement destinée aux utilisateurs d'ordinateurs portables qui ne se connectent pas régulièrement au domaine de leur entreprise. Le nombre d'informations d'identification que l'ordinateur stocke peut être contrôlé par la clé de registre suivante, ou via une stratégie de groupe :
```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
Les informations d'identification sont cachées aux utilisateurs normaux, même aux comptes administrateurs. L'utilisateur **SYSTEM** est le seul utilisateur ayant les **privilèges** pour **voir** ces **informations d'identification**. Afin qu'un administrateur puisse voir ces informations d'identification dans le registre, il doit y accéder en tant qu'utilisateur SYSTEM.\
Les informations d'identification mises en cache sont stockées dans le registre à l'emplacement suivant :
```
HKEY_LOCAL_MACHINE\SECURITY\Cache
```
## Extraction à partir de Mimikatz: `lsadump::cache`
À partir de [ici](http://juggernaut.wikidot.com/cached-credentials).

## Utilisateurs protégés

Lorsque l'utilisateur connecté est membre du groupe Utilisateurs protégés, les protections suivantes sont appliquées :

* La délégation d'informations d'identification (CredSSP) ne mettra pas en cache les informations d'identification en texte brut de l'utilisateur, même lorsque le paramètre de stratégie de groupe **Autoriser la délégation des informations d'identification par défaut** est activé.
* À partir de Windows 8.1 et de Windows Server 2012 R2, Windows Digest ne mettra pas en cache les informations d'identification en texte brut de l'utilisateur, même lorsque Windows Digest est activé.
* **NTLM** ne mettra **pas en cache** les informations d'identification en texte brut de l'utilisateur ou la fonction unidirectionnelle NT (NTOWF).
* **Kerberos** ne créera plus de clés **DES** ou **RC4**. De plus, il ne mettra pas en cache les informations d'identification en texte brut de l'utilisateur ou les clés à long terme après l'acquisition initiale du TGT.
* Un vérificateur mis en cache n'est pas créé lors de la connexion ou du déverrouillage, de sorte que la connexion hors ligne n'est plus prise en charge.

Après l'ajout du compte utilisateur au groupe Utilisateurs protégés, la protection commencera lorsque l'utilisateur se connectera à l'appareil. À partir de [ici](https://docs.microsoft.com/fr-fr/windows-server/security/credentials-protection-and-management/protected-users-security-group).

| Windows Server 2003 RTM | Windows Server 2003 SP1+ | <p>Windows Server 2012,<br>Windows Server 2008 R2,<br>Windows Server 2008</p> | Windows Server 2016          |
| ----------------------- | ------------------------ | ----------------------------------------------------------------------------- | ---------------------------- |
| Opérateurs de compte     | Opérateurs de compte     | Opérateurs de compte                                                          | Opérateurs de compte         |
| Administrateur           | Administrateur           | Administrateur                                                                 | Administrateur               |
| Administrateurs          | Administrateurs          | Administrateurs                                                                | Administrateurs              |
| Opérateurs de sauvegarde | Opérateurs de sauvegarde | Opérateurs de sauvegarde                                                       | Opérateurs de sauvegarde     |
| Éditeurs de certificats   |                          |                                                                               |                              |
| Administrateurs de domaine | Administrateurs de domaine | Administrateurs de domaine                                                     | Administrateurs de domaine   |
| Contrôleurs de domaine   | Contrôleurs de domaine   | Contrôleurs de domaine                                                          | Contrôleurs de domaine        |
| Administrateurs d'entreprise | Administrateurs d'entreprise | Administrateurs d'entreprise                                               | Administrateurs d'entreprise |
|                         |                          |                                                                               | Administrateurs de clé d'entreprise |
|                         |                          |                                                                               | Administrateurs de clé       |
| Krbtgt                  | Krbtgt                   | Krbtgt                                                                        | Krbtgt                       |
| Opérateurs d'impression  | Opérateurs d'impression  | Opérateurs d'impression                                                        | Opérateurs d'impression       |
|                         |                          | Contrôleurs de domaine en lecture seule                                        | Contrôleurs de domaine en lecture seule |
| Réplicateur             | Réplicateur              | Réplicateur                                                                    | Réplicateur                   |
| Administrateurs de schéma | Administrateurs de schéma | Administrateurs de schéma                                                     | Administrateurs de schéma    |
| Opérateurs de serveur   | Opérateurs de serveur    | Opérateurs de serveur                                                          | Opérateurs de serveur         |

**Tableau à partir de** [**ici**](https://docs.microsoft.com/fr-fr/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory).
