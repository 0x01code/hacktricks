```markdown
<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF** Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-me no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

# Integridade do Firmware

O **firmware personalizado e/ou binários compilados podem ser carregados para explorar falhas de verificação de integridade ou assinatura**. Os seguintes passos podem ser seguidos para a compilação de backdoor bind shell:

1. O firmware pode ser extraído usando o firmware-mod-kit (FMK).
2. A arquitetura e endianness do firmware alvo devem ser identificadas.
3. Um compilador cruzado pode ser construído usando Buildroot ou outros métodos adequados para o ambiente.
4. O backdoor pode ser construído usando o compilador cruzado.
5. O backdoor pode ser copiado para o diretório /usr/bin do firmware extraído.
6. O binário QEMU apropriado pode ser copiado para o rootfs do firmware extraído.
7. O backdoor pode ser emulado usando chroot e QEMU.
8. O backdoor pode ser acessado via netcat.
9. O binário QEMU deve ser removido do rootfs do firmware extraído.
10. O firmware modificado pode ser reempacotado usando FMK.
11. O firmware com backdoor pode ser testado emulando-o com o firmware analysis toolkit (FAT) e conectando-se ao IP e porta do backdoor alvo usando netcat.

Se um shell root já foi obtido através de análise dinâmica, manipulação do bootloader ou testes de segurança de hardware, binários maliciosos pré-compilados como implantes ou reverse shells podem ser executados. Ferramentas automatizadas de payload/implante como o framework Metasploit e 'msfvenom' podem ser utilizadas seguindo os passos abaixo:

1. A arquitetura e endianness do firmware alvo devem ser identificadas.
2. Msfvenom pode ser usado para especificar o payload alvo, IP do host atacante, número da porta de escuta, tipo de arquivo, arquitetura, plataforma e o arquivo de saída.
3. O payload pode ser transferido para o dispositivo comprometido e garantir que ele tenha permissões de execução.
4. Metasploit pode ser preparado para lidar com solicitações de entrada iniciando o msfconsole e configurando as configurações de acordo com o payload.
5. O meterpreter reverse shell pode ser executado no dispositivo comprometido.
6. Sessões do meterpreter podem ser monitoradas à medida que se abrem.
7. Atividades de pós-exploração podem ser realizadas.

Se possível, vulnerabilidades dentro de scripts de inicialização podem ser exploradas para obter acesso persistente a um dispositivo através de reinicializações. Essas vulnerabilidades surgem quando scripts de inicialização referenciam, [linkam simbolicamente](https://www.chromium.org/chromium-os/chromiumos-design-docs/hardening-against-malicious-stateful-data), ou dependem de código localizado em locais montados não confiáveis, como cartões SD e volumes flash usados para armazenar dados fora dos sistemas de arquivos raiz.

# Referências
* Para mais informações, verifique [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF** Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-me no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
```
