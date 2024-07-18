# macOS进程滥用

{% hint style="success" %}
学习并练习AWS黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks培训AWS红队专家（ARTE）**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习GCP黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks培训GCP红队专家（GRTE）**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持HackTricks</summary>

* 查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享黑客技巧。

</details>
{% endhint %}

## 进程基本信息

进程是运行可执行文件的实例，但进程不运行代码，这些是线程。因此，**进程只是运行线程的容器**，提供内存、描述符、端口、权限等。

传统上，进程是在其他进程（除了PID 1）内启动的，通过调用**`fork`**来创建当前进程的精确副本，然后**子进程**通常会调用**`execve`**来加载新的可执行文件并运行它。然后，引入了**`vfork`**来加快此过程，而无需进行任何内存复制。\
然后引入了**`posix_spawn`**，将**`vfork`**和**`execve`**结合在一个调用中，并接受标志：

- `POSIX_SPAWN_RESETIDS`：将有效ID重置为真实ID
- `POSIX_SPAWN_SETPGROUP`：设置进程组关联
- `POSUX_SPAWN_SETSIGDEF`：设置信号的默认行为
- `POSIX_SPAWN_SETSIGMASK`：设置信号掩码
- `POSIX_SPAWN_SETEXEC`：在同一进程中执行（类似于具有更多选项的`execve`）
- `POSIX_SPAWN_START_SUSPENDED`：启动时挂起
- `_POSIX_SPAWN_DISABLE_ASLR`：无ASLR启动
- `_POSIX_SPAWN_NANO_ALLOCATOR:` 使用libmalloc的Nano分配器
- `_POSIX_SPAWN_ALLOW_DATA_EXEC:` 允许数据段上的`rwx`
- `POSIX_SPAWN_CLOEXEC_DEFAULT`：默认情况下在exec(2)上关闭所有文件描述符
- `_POSIX_SPAWN_HIGH_BITS_ASLR:` 随机化ASLR幻数位

此外，`posix_spawn`允许指定一个控制生成进程某些方面的**`posix_spawnattr`**数组，以及**`posix_spawn_file_actions`**来修改描述符的状态。

当进程终止时，它会向父进程发送**返回代码**（如果父进程已经终止，则新父进程是PID 1），使用信号`SIGCHLD`。父进程需要调用`wait4()`或`waitid()`来获取此值，直到发生这种情况，子进程将保持僵尸状态，仍然列出但不会消耗资源。

### 进程ID

进程ID（PID，Process Identifiers）标识唯一进程。在XNU中，**PID**是**64位**的，单调递增，**永不回绕**（以避免滥用）。

### 进程组、会话和联合

**进程**可以被放入**组**中以便更容易处理它们。例如，shell脚本中的命令将位于同一进程组中，因此可以使用kill等命令**一起向它们发送信号**。\
还可以将进程**分组到会话**中。当进程启动会话（`setsid(2)`）时，子进程将被放入会话中，除非它们启动自己的会话。

Coalition是在Darwin中将进程分组的另一种方式。加入联合的进程允许访问池资源，共享账本或面对Jetsam。联合有不同的角色：领导者、XPC服务、扩展。

### 凭证和身份

每个进程都持有**凭证**，用于**标识其在系统中的特权**。每个进程将有一个主要的`uid`和一个主要的`gid`（尽管可能属于多个组）。\
如果二进制文件具有`setuid/setgid`位，则还可以更改用户和组ID。\
有几个函数可用于**设置新的uid/gid**。

系统调用**`persona`**提供了**备用**一组**凭证**。采用人物意味着在一次性地假定其uid、gid和组成员身份。在[**源代码**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h)中可以找到该结构：
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## 线程基本信息

1. **POSIX 线程 (pthreads):** macOS 支持 POSIX 线程 (`pthreads`), 这是 C/C++ 的标准线程 API 的一部分。macOS 中的 pthreads 实现位于 `/usr/lib/system/libsystem_pthread.dylib`，这来自公开可用的 `libpthread` 项目。该库提供了创建和管理线程所需的函数。
2. **创建线程:** 使用 `pthread_create()` 函数来创建新线程。在内部，此函数调用 `bsdthread_create()`，这是一个特定于 XNU 内核（macOS 基于的内核）的较低级系统调用。此系统调用接受从 `pthread_attr`（属性）派生的各种标志，指定线程行为，包括调度策略和堆栈大小。
* **默认堆栈大小:** 新线程的默认堆栈大小为 512 KB，对于典型操作来说足够了，但如果需要更多或更少的空间，可以通过线程属性进行调整。
3. **线程初始化:** `__pthread_init()` 函数在线程设置期间至关重要，利用 `env[]` 参数来解析环境变量，这些变量可以包含有关堆栈位置和大小的详细信息。

#### macOS 中的线程终止

1. **退出线程:** 通常通过调用 `pthread_exit()` 来终止线程。此函数允许线程清理退出，执行必要的清理操作，并允许线程向任何加入者发送返回值。
2. **线程清理:** 调用 `pthread_exit()` 后，将调用 `pthread_terminate()` 函数，该函数处理所有相关线程结构的移除。它释放 Mach 线程端口（Mach 是 XNU 内核中的通信子系统）并调用 `bsdthread_terminate`，这是一个删除与线程相关的内核级结构的系统调用。

#### 同步机制

为了管理对共享资源的访问并避免竞态条件，macOS 提供了几种同步原语。在多线程环境中，这些同步原语至关重要，以确保数据完整性和系统稳定性：

1. **互斥锁 (Mutexes):**
* **常规互斥锁 (Signature: 0x4D555458):** 具有 60 字节的内存占用的标准互斥锁（56 字节用于互斥锁，4 字节用于签名）。
* **快速互斥锁 (Signature: 0x4d55545A):** 类似于常规互斥锁，但针对更快的操作进行了优化，大小也为 60 字节。
2. **条件变量 (Condition Variables):**
* 用于等待特定条件发生，大小为 44 字节（40 字节加上 4 字节的签名）。
* **条件变量属性 (Signature: 0x434e4441):** 条件变量的配置属性，大小为 12 字节。
3. **一次性变量 (Once Variable) (Signature: 0x4f4e4345):**
* 确保初始化代码仅执行一次。大小为 12 字节。
4. **读写锁 (Read-Write Locks):**
* 一次允许多个读取者或一个写入者，促进对共享数据的高效访问。
* **读写锁 (Signature: 0x52574c4b):** 大小为 196 字节。
* **读写锁属性 (Signature: 0x52574c41):** 读写锁的属性，大小为 20 字节。

{% hint style="success" %}
这些对象的最后 4 个字节用于检测溢出。
{% endhint %}

### 线程本地变量 (TLV)

**线程本地变量 (TLV)** 在 Mach-O 文件的上下文中（macOS 中可执行文件的格式）用于声明特定于**每个线程**的变量，确保每个线程都有自己独立的变量实例，提供一种避免冲突并保持数据完整性的方法，而无需像互斥锁那样显式同步机制。

在 C 和相关语言中，可以使用 **`__thread`** 关键字声明线程本地变量。以下是在您的示例中的工作方式：
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
这个片段将`tlv_var`定义为线程本地变量。运行此代码的每个线程都将拥有自己的`tlv_var`，一个线程对`tlv_var`所做的更改不会影响另一个线程中的`tlv_var`。

在Mach-O二进制文件中，与线程本地变量相关的数据被组织到特定的部分中：

- **`__DATA.__thread_vars`**：此部分包含有关线程本地变量的元数据，如它们的类型和初始化状态。
- **`__DATA.__thread_bss`**：此部分用于未显式初始化的线程本地变量。这是为零初始化数据保留的一部分内存。

Mach-O还提供了一个名为**`tlv_atexit`**的特定API，用于在线程退出时管理线程本地变量。此API允许您**注册析构函数** - 在线程终止时清理线程本地数据的特殊函数。

### 线程优先级

了解线程优先级涉及查看操作系统如何决定运行哪些线程以及何时运行的方式。这个决定受每个线程分配的优先级级别的影响。在macOS和类Unix系统中，使用`nice`、`renice`和服务质量（QoS）类等概念来处理这个问题。

#### Nice和Renice

1. **Nice:**
   - 进程的`nice`值是影响其优先级的数字。每个进程的nice值范围从-20（最高优先级）到19（最低优先级）。进程创建时的默认nice值通常为0。
   - 较低的nice值（接近-20）使进程更“自私”，相对于具有较高nice值的其他进程，它获得更多的CPU时间。
2. **Renice:**
   - `renice`是一个用于更改已运行进程nice值的命令。这可用于根据新的nice值动态调整进程的优先级，增加或减少它们的CPU时间分配。
   - 例如，如果一个进程暂时需要更多的CPU资源，您可以使用`renice`降低其nice值。

#### 服务质量（QoS）类

QoS类是处理线程优先级的一种更现代方法，特别是在支持**Grand Central Dispatch (GCD)**的macOS等系统中。QoS类允许开发人员根据任务的重要性或紧急性将工作分类到不同级别。macOS根据这些QoS类自动管理线程优先级：

1. **用户交互：**
   - 此类用于当前正在与用户交互或需要立即提供良好用户体验所需的即时结果的任务。为保持界面响应（例如动画或事件处理），这些任务被赋予最高优先级。
2. **用户启动：**
   - 用户启动的任务需要用户启动并期望立即获得结果，例如打开文档或单击需要计算的按钮。这些任务具有较高的优先级，但低于用户交互。
3. **实用程序：**
   - 这些任务运行时间较长，通常显示进度指示器（例如下载文件、导入数据）。它们的优先级低于用户启动的任务，不需要立即完成。
4. **后台：**
   - 此类用于在后台运行且对用户不可见的任务。这些任务可以是索引、同步或备份等任务。它们具有最低的优先级，对系统性能影响最小。

使用QoS类，开发人员无需管理确切的优先级数字，而是专注于任务的性质，系统会相应地优化CPU资源。

此外，还有不同的**线程调度策略**，用于指定调度器将考虑的一组调度参数。这可以使用`thread_policy_[set/get]`来完成。这在竞争条件攻击中可能很有用。
### Python注入

如果环境变量**`PYTHONINSPECT`**被设置，Python进程在完成后将进入Python命令行界面。还可以使用**`PYTHONSTARTUP`**指定在交互会话开始时要执行的Python脚本。\
但是，请注意，当**`PYTHONINSPECT`**创建交互会话时，**`PYTHONSTARTUP`**脚本不会被执行。

其他环境变量如**`PYTHONPATH`**和**`PYTHONHOME`**也可能对使Python命令执行任意代码有用。

请注意，使用**`pyinstaller`**编译的可执行文件即使使用嵌入式Python运行，也不会使用这些环境变量。

{% hint style="danger" %}
总的来说，我找不到一种利用环境变量来使Python执行任意代码的方法。\
然而，大多数人使用**Hombrew**安装Python，这将在默认管理员用户的**可写位置**安装Python。您可以使用以下方式劫持它：
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
即使是**root**在运行Python时也会运行此代码。
{% endhint %}

## 检测

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) 是一个开源应用程序，可以**检测和阻止进程注入**操作：

* 使用**环境变量**：它将监视以下任何环境变量的存在：**`DYLD_INSERT_LIBRARIES`**、**`CFNETWORK_LIBRARY_PATH`**、**`RAWCAMERA_BUNDLE_PATH`** 和 **`ELECTRON_RUN_AS_NODE`**
* 使用**`task_for_pid`** 调用：查找一个进程想要获取另一个进程的**任务端口**，从而允许在进程中注入代码。
* **Electron 应用程序参数**：某人可以使用**`--inspect`**、**`--inspect-brk`** 和 **`--remote-debugging-port`** 命令行参数以调试模式启动 Electron 应用程序，从而向其注入代码。
* 使用**符号链接**或**硬链接**：通常最常见的滥用是**使用我们的用户权限放置一个链接**，并**将其指向更高权限**的位置。对于硬链接和符号链接，检测非常简单。如果创建链接的进程具有**不同的权限级别**，则我们会创建一个**警报**。不幸的是，在符号链接的情况下，阻止是不可能的，因为我们在创建链接之前没有关于链接目标的信息。这是苹果 EndpointSecuriy 框架的一个限制。

### 其他进程发出的调用

在[**这篇博文**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html)中，您可以了解如何使用函数**`task_name_for_pid`**获取有关其他**在进程中注入代码的进程**的信息，然后获取有关该其他进程的信息。

请注意，要调用该函数，您需要与运行进程的用户相同的uid或**root**（它返回有关进程的信息，而不是注入代码的方法）。

## 参考资料

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

{% hint style="success" %}
学习并练习 AWS 黑客技能：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习 GCP 黑客技能：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或在 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** 上关注我们**。
* 通过向 [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
