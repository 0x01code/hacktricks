<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!

- Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)

- Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)

- **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **Compartilhe seus truques de hacking enviando PRs para o [repositório hacktricks](https://github.com/carlospolop/hacktricks) e [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


# Invoke
```text
powershell -ep bypass
. .\powerup.ps
Invoke-AllChecks
```
# Verificações

_03/2019_

* [x] Privilégios atuais
* [x] Caminhos de serviço sem aspas
* [x] Permissões executáveis de serviço
* [x] Permissões de serviço
* [x] %PATH% para locais de DLLs sequestráveis
* [x] Chave de registro AlwaysInstallElevated
* [x] Credenciais de autologon no registro
* [x] Autoruns e configs modificáveis do registro
* [x] Arquivos/configs schtask modificáveis
* [x] Arquivos de instalação não assistida
* [x] Strings web.config criptografadas
* [x] Senhas de diretório virtual e de pool de aplicativos criptografadas
* [x] Senhas em texto simples no arquivo McAfee SiteList.xml
* [x] Arquivos .xml de Preferências de Política de Grupo em cache
