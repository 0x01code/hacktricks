# Seccomp

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

其他支持HackTricks的方式：

- 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
- 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
- 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
- **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注**我的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
- 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

## 基本信息

**Seccomp**，全称Secure Computing mode，是**Linux内核的安全功能，旨在过滤系统调用**。它将进程限制在一组有限的系统调用上（`exit()`、`sigreturn()`、`read()`和`write()`，用于已打开的文件描述符）。如果进程尝试调用其他内容，内核将使用SIGKILL或SIGSYS终止该进程。该机制不会虚拟化资源，而是将进程与资源隔离开来。

有两种激活seccomp的方式：通过`prctl(2)`系统调用使用`PR_SET_SECCOMP`，或者对于Linux内核3.17及以上版本，使用`seccomp(2)`系统调用。通过向`/proc/self/seccomp`写入以启用seccomp的旧方法已被弃用，推荐使用`prctl()`。

一种增强功能**seccomp-bpf**，增加了使用伯克利数据包过滤器（BPF）规则自定义策略来过滤系统调用的能力。此扩展被软件如OpenSSH、vsftpd以及Chrome OS和Linux上的Chrome/Chromium浏览器所利用，用于灵活高效地过滤系统调用，提供了对于Linux中现在不再支持的systrace的替代方案。

### **原始/严格模式**

在此模式下，Seccomp**仅允许系统调用**`exit()`、`sigreturn()`、`read()`和`write()`用于已打开的文件描述符。如果进行任何其他系统调用，进程将被使用SIGKILL终止。

{% code title="seccomp_strict.c" %}
```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <linux/seccomp.h>
#include <sys/prctl.h>

//From https://sysdig.com/blog/selinux-seccomp-falco-technical-discussion/
//gcc seccomp_strict.c -o seccomp_strict

int main(int argc, char **argv)
{
int output = open("output.txt", O_WRONLY);
const char *val = "test";

//enables strict seccomp mode
printf("Calling prctl() to set seccomp strict mode...\n");
prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);

//This is allowed as the file was already opened
printf("Writing to an already open file...\n");
write(output, val, strlen(val)+1);

//This isn't allowed
printf("Trying to open file for reading...\n");
int input = open("output.txt", O_RDONLY);

printf("You will not see this message--the process will be killed first\n");
}
```
### Seccomp-bpf

这种模式允许使用使用伯克利数据包过滤器规则实现的可配置策略来过滤系统调用。

{% code title="seccomp_bpf.c" %}
```c
#include <seccomp.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

//https://security.stackexchange.com/questions/168452/how-is-sandboxing-implemented/175373
//gcc seccomp_bpf.c -o seccomp_bpf -lseccomp

void main(void) {
/* initialize the libseccomp context */
scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);

/* allow exiting */
printf("Adding rule : Allow exit_group\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

/* allow getting the current pid */
//printf("Adding rule : Allow getpid\n");
//seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(getpid), 0);

printf("Adding rule : Deny getpid\n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(getpid), 0);
/* allow changing data segment size, as required by glibc */
printf("Adding rule : Allow brk\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(brk), 0);

/* allow writing up to 512 bytes to fd 1 */
printf("Adding rule : Allow write upto 512 bytes to FD 1\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 2,
SCMP_A0(SCMP_CMP_EQ, 1),
SCMP_A2(SCMP_CMP_LE, 512));

/* if writing to any other fd, return -EBADF */
printf("Adding rule : Deny write to any FD except 1 \n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(write), 1,
SCMP_A0(SCMP_CMP_NE, 1));

/* load and enforce the filters */
printf("Load rules and enforce \n");
seccomp_load(ctx);
seccomp_release(ctx);
//Get the getpid is denied, a weird number will be returned like
//this process is -9
printf("this process is %d\n", getpid());
}
```
## Docker中的Seccomp

**Seccomp-bpf**由**Docker**支持，用于限制容器中的**syscalls**，有效减少攻击面。您可以在[https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/)找到**默认情况下被阻止的syscalls**，并且可以在此处找到**默认的seccomp配置文件**：[https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)。\
您可以使用以下命令以**不同的seccomp策略**运行docker容器：
```bash
docker run --rm \
-it \
--security-opt seccomp=/path/to/seccomp/profile.json \
hello-world
```
如果您想例如**禁止**容器执行一些**系统调用**，比如 `uname`，您可以从[https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)下载默认配置文件，然后只需**从列表中删除 `uname` 字符串**。\
如果您想确保**某个二进制文件在 Docker 容器中无法运行**，您可以使用 strace 列出二进制文件正在使用的系统调用，然后禁止它们。\
在以下示例中，发现了 `uname` 的**系统调用**：
```bash
docker run -it --security-opt seccomp=default.json modified-ubuntu strace uname
```
{% hint style="info" %}
如果您只是使用 **Docker 来启动一个应用程序**，您可以使用 **`strace`** 来为其创建 **配置文件**，并且只允许其需要的 **系统调用**
{% endhint %}

### 示例 Seccomp 策略

[示例来自这里](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)

为了说明 Seccomp 功能，让我们创建一个 Seccomp 配置文件，禁用 "chmod" 系统调用，如下所示。
```json
{
"defaultAction": "SCMP_ACT_ALLOW",
"syscalls": [
{
"name": "chmod",
"action": "SCMP_ACT_ERRNO"
}
]
}
```
在上述配置文件中，我们将默认操作设置为“允许”，并创建了一个黑名单来禁用“chmod”。为了更安全，我们可以将默认操作设置为“拒绝”，并创建一个白名单来有选择性地启用系统调用。\
以下输出显示了“chmod”调用返回错误，因为在seccomp配置文件中已禁用了它。
```bash
$ docker run --rm -it --security-opt seccomp:/home/smakam14/seccomp/profile.json busybox chmod 400 /etc/hosts
chmod: /etc/hosts: Operation not permitted
```
以下输出显示了“docker inspect”显示的配置文件：
```json
"SecurityOpt": [
"seccomp:{\"defaultAction\":\"SCMP_ACT_ALLOW\",\"syscalls\":[{\"name\":\"chmod\",\"action\":\"SCMP_ACT_ERRNO\"}]}"
],
```
### 在Docker中停用它

使用标志启动一个容器：**`--security-opt seccomp=unconfined`**

截至Kubernetes 1.19，**所有Pod默认启用seccomp**。然而，应用于Pod的默认seccomp配置文件是由容器运行时（例如Docker、containerd）提供的“**RuntimeDefault**”配置文件。这个“RuntimeDefault”配置文件允许大多数系统调用，同时阻止一些被认为是危险的或容器通常不需要的系统调用。
