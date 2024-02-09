# Checklist - Escalação de Privilégios Local do Windows

<details>

<summary><strong>Aprenda hacking na AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe suas dicas de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

### **Melhor ferramenta para procurar vetores de escalonamento de privilégios locais do Windows:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

### [Informações do Sistema](windows-local-privilege-escalation/#system-info)

* [ ] Obter [**informações do sistema**](windows-local-privilege-escalation/#system-info)
* [ ] Procurar por **exploits de kernel** [**usando scripts**](windows-local-privilege-escalation/#version-exploits)
* [ ] Usar o **Google para pesquisar** por **exploits de kernel**
* [ ] Usar o **searchsploit para pesquisar** por **exploits de kernel**
* [ ] Informações interessantes em [**vars de ambiente**](windows-local-privilege-escalation/#environment)?
* [ ] Senhas no [**histórico do PowerShell**](windows-local-privilege-escalation/#powershell-history)?
* [ ] Informações interessantes nas [**configurações de Internet**](windows-local-privilege-escalation/#internet-settings)?
* [ ] [**Drives**](windows-local-privilege-escalation/#drives)?
* [ ] [**Exploração do WSUS**](windows-local-privilege-escalation/#wsus)?
* [**AlwaysInstallElevated**](windows-local-privilege-escalation/#alwaysinstallelevated)?

### Enumeração de Logging/AV (Antivírus) (windows-local-privilege-escalation/#enumeration)

* [ ] Verificar as configurações de [**Auditoria** ](windows-local-privilege-escalation/#audit-settings)e [**WEF** ](windows-local-privilege-escalation/#wef)
* [ ] Verificar o [**LAPS**](windows-local-privilege-escalation/#laps)
* [ ] Verificar se o [**WDigest** ](windows-local-privilege-escalation/#wdigest)está ativo
* [ ] [**Proteção LSA**](windows-local-privilege-escalation/#lsa-protection)?
* [ ] [**Guarda de Credenciais**](windows-local-privilege-escalation/#credentials-guard)[?](windows-local-privilege-escalation/#cached-credentials)
* [ ] [**Credenciais em Cache**](windows-local-privilege-escalation/#cached-credentials)?
* [ ] Verificar se há algum [**AV**](windows-av-bypass)
* [ ] [**Política AppLocker**](authentication-credentials-uac-and-efs#applocker-policy)?
* [ ] [**UAC**](authentication-credentials-uac-and-efs/uac-user-account-control)
* [ ] [**Privilégios de Usuário**](windows-local-privilege-escalation/#users-and-groups)
* [ ] Verificar os [**privilégios atuais** do usuário](windows-local-privilege-escalation/#users-and-groups)
* [ ] Você é [**membro de algum grupo privilegiado**](windows-local-privilege-escalation/#privileged-groups)?
* [ ] Verificar se você tem [algum desses tokens habilitados](windows-local-privilege-escalation/#token-manipulation): **SeImpersonatePrivilege, SeAssignPrimaryPrivilege, SeTcbPrivilege, SeBackupPrivilege, SeRestorePrivilege, SeCreateTokenPrivilege, SeLoadDriverPrivilege, SeTakeOwnershipPrivilege, SeDebugPrivilege** ?
* [ ] [**Sessões de Usuários**](windows-local-privilege-escalation/#logged-users-sessions)?
* [ ] Verificar [**homes dos usuários**](windows-local-privilege-escalation/#home-folders) (acesso?)
* [ ] Verificar a [**Política de Senhas**](windows-local-privilege-escalation/#password-policy)
* [ ] O que há [**dentro da Área de Transferência**](windows-local-privilege-escalation/#get-the-content-of-the-clipboard)?

### [Rede](windows-local-privilege-escalation/#network)

* [ ] Verificar as [**informações atuais** da rede](windows-local-privilege-escalation/#network)
* [ ] Verificar **serviços locais ocultos** restritos ao exterior

### [Processos em Execução](windows-local-privilege-escalation/#running-processes)

* [ ] Permissões de [**arquivos e pastas dos binários de processos**](windows-local-privilege-escalation/#file-and-folder-permissions)
* [ ] [**Mineração de Senhas na Memória**](windows-local-privilege-escalation/#memory-password-mining)
* [ ] [**Aplicativos GUI Inseguros**](windows-local-privilege-escalation/#insecure-gui-apps)

### [Serviços](windows-local-privilege-escalation/#services)

* [ ] [Você pode **modificar algum serviço**?](windows-local-privilege-escalation#permissions)
* [ ] [Você pode **modificar** o **binário** que é **executado** por algum **serviço**?](windows-local-privilege-escalation/#modify-service-binary-path)
* [ ] [Você pode **modificar** o **registro** de algum **serviço**?](windows-local-privilege-escalation/#services-registry-modify-permissions)
* [ ] [Você pode aproveitar algum **caminho de binário de serviço** **não citado**?](windows-local-privilege-escalation/#unquoted-service-paths)

### [**Aplicativos**](windows-local-privilege-escalation/#applications)

* [ ] **Permissões de **escrita em aplicativos instalados**](windows-local-privilege-escalation/#write-permissions)
* [ ] [**Aplicativos de Inicialização**](windows-local-privilege-escalation/#run-at-startup)
* [ ] **Drivers** [**Vulneráveis**](windows-local-privilege-escalation/#drivers)

### [Sequestro de DLL](windows-local-privilege-escalation/#path-dll-hijacking)

* [ ] Você pode **escrever em qualquer pasta dentro do PATH**?
* [ ] Existe algum binário de serviço conhecido que **tenta carregar alguma DLL inexistente**?
* [ ] Você pode **escrever** em qualquer **pasta de binários**?

### [Rede](windows-local-privilege-escalation/#network)

* [ ] Enumerar a rede (compartilhamentos, interfaces, rotas, vizinhos, ...)
* [ ] Dê uma olhada especial nos serviços de rede ouvindo em localhost (127.0.0.1)

### [Credenciais do Windows](windows-local-privilege-escalation/#windows-credentials)

* [ ] [**Credenciais do Winlogon** ](windows-local-privilege-escalation/#winlogon-credentials)
* [ ] [**Windows Vault**](windows-local-privilege-escalation/#credentials-manager-windows-vault) credenciais que você poderia usar?
* [ ] Informações interessantes em [**credenciais DPAPI**](windows-local-privilege-escalation/#dpapi)?
* [ ] Senhas de redes Wi-Fi salvas [**Wifi networks**](windows-local-privilege-escalation/#wifi)?
* [ ] Informações interessantes em [**conexões RDP salvas**](windows-local-privilege-escalation/#saved-rdp-connections)?
* [ ] Senhas em [**comandos recentemente executados**](windows-local-privilege-escalation/#recently-run-commands)?
* [ ] [**Gerenciador de Credenciais do Desktop Remoto**](windows-local-privilege-escalation/#remote-desktop-credential-manager) senhas?
* [ ] [**AppCmd.exe** existe](windows-local-privilege-escalation/#appcmd-exe)? Credenciais?
* [ ] [**SCClient.exe**](windows-local-privilege-escalation/#scclient-sccm)? Carregamento Lateral de DLL?

### [Arquivos e Registro (Credenciais)](windows-local-privilege-escalation/#files-and-registry-credentials)

* [ ] **Putty:** [**Credenciais**](windows-local-privilege-escalation/#putty-creds) **e** [**Chaves de host SSH**](windows-local-privilege-escalation/#putty-ssh-host-keys)
* [ ] [**Chaves SSH no registro**](windows-local-privilege-escalation/#ssh-keys-in-registry)?
* [ ] Senhas em [**arquivos não assistidos**](windows-local-privilege-escalation/#unattended-files)?
* [ ] Algum backup de [**SAM & SYSTEM**](windows-local-privilege-escalation/#sam-and-system-backups)?
* [ ] [**Credenciais de Nuvem**](windows-local-privilege-escalation/#cloud-credentials)?
* [ ] [**McAfee SiteList.xml**](windows-local-privilege-escalation/#mcafee-sitelist.xml) arquivo?
* [ ] [**Senha GPP em Cache**](windows-local-privilege-escalation/#cached-gpp-pasword)?
* [ ] Senha em [**arquivo de configuração web do IIS**](windows-local-privilege-escalation/#iis-web-config)?
* [ ] Informações interessantes em [**logs da web**](windows-local-privilege-escalation/#logs)?
* [ ] Você deseja [**solicitar credenciais**](windows-local-privilege-escalation/#ask-for-credentials) ao usuário?
* [ ] Informações interessantes em [**arquivos dentro da Lixeira**](windows-local-privilege-escalation/#credentials-in-the-recyclebin)?
* [ ] Outro [**registro contendo credenciais**](windows-local-privilege-escalation/#inside-the-registry)?
* [ ] Dentro dos [**dados do navegador**](windows-local-privilege-escalation/#browsers-history) (bancos de dados, histórico, favoritos, ...)?
* [ ] [**Pesquisa genérica de senha**](windows-local-privilege-escalation/#generic-password-search-in-files-and-registry) em arquivos e registro
* [ ] [**Ferramentas**](windows-local-privilege-escalation/#tools-that-search-for-passwords) para pesquisar automaticamente senhas

### [Manipuladores Vazados](windows-local-privilege-escalation/#leaked-handlers)

* [ ] Você tem acesso a algum manipulador de um processo executado pelo administrador?

### [Impersonação de Cliente de Pipe](windows-local-privilege-escalation/#named-pipe-client-impersonation)

* [ ] Verifique se você pode abusar disso
