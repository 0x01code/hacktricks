# CGroups

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLAN SUBSKRYPCJI**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## Podstawowe informacje

**Linux Control Groups**, znane również jako **cgroups**, to funkcja jądra Linux, która umożliwia alokację, ograniczanie i priorytetyzację zasobów systemowych, takich jak CPU, pamięć i operacje wejścia/wyjścia na dysku, wśród grup procesów. Oferują one mechanizm do **zarządzania i izolowania użycia zasobów** przez kolekcje procesów, co jest korzystne w celu ograniczania zasobów, izolacji obciążenia i priorytetyzacji zasobów między różnymi grupami procesów.

Istnieją **dwie wersje cgroups**: wersja 1 i wersja 2. Obydwie mogą być używane jednocześnie w systemie. Główną różnicą jest to, że **wersja 2 cgroups** wprowadza **hierarchiczną strukturę przypominającą drzewo**, umożliwiając bardziej wyszczególnione i szczegółowe rozdział zasobów między grupami procesów. Dodatkowo, wersja 2 wprowadza różne ulepszenia, w tym:

Oprócz nowej organizacji hierarchicznej, wersja 2 cgroups wprowadza również **kilka innych zmian i ulepszeń**, takich jak obsługa **nowych kontrolerów zasobów**, lepsza obsługa aplikacji zgodnych z wcześniejszymi wersjami i poprawiona wydajność.

Ogólnie rzecz biorąc, wersja 2 cgroups **oferuje więcej funkcji i lepszą wydajność** niż wersja 1, ale ta ostatnia może być nadal używana w określonych scenariuszach, gdzie istnieje konieczność kompatybilności z starszymi systemami.

Możesz wyświetlić grupy cgroups v1 i v2 dla dowolnego procesu, patrząc na jego plik cgroup w /proc/\<pid>. Możesz zacząć od sprawdzenia grup cgroups powłoki za pomocą tego polecenia:
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
Struktura wyjścia jest następująca:

- **Numery 2–12**: cgroups v1, gdzie każda linia reprezentuje inny cgroup. Kontrolery dla tych cgroupów są określone obok numeru.
- **Numer 1**: Również cgroups v1, ale wyłącznie w celach zarządzania (ustawiane przez np. systemd) i nie posiada kontrolera.
- **Numer 0**: Reprezentuje cgroups v2. Nie są wymienione żadne kontrolery, a ta linia jest wyłączna dla systemów działających tylko na cgroups v2.
- **Nazwy są hierarchiczne**, przypominające ścieżki plików, wskazujące strukturę i relacje między różnymi cgroupami.
- **Nazwy takie jak /user.slice lub /system.slice** określają kategoryzację cgroupów, gdzie user.slice zwykle dotyczy sesji logowania zarządzanych przez systemd, a system.slice dotyczy usług systemowych.

### Przeglądanie cgroupów

System plików jest zwykle wykorzystywany do dostępu do **cgroupów**, odbiegając od tradycyjnego interfejsu wywołań systemowych Unix używanego do interakcji z jądrem. Aby zbadać konfigurację cgroupu powłoki, należy przejrzeć plik **/proc/self/cgroup**, który ujawnia cgroup powłoki. Następnie, przechodząc do katalogu **/sys/fs/cgroup** (lub **`/sys/fs/cgroup/unified`**), i zlokalizowaniu katalogu o tej samej nazwie co cgroup, można obserwować różne ustawienia i informacje o wykorzystaniu zasobów dotyczące cgroupu.

![System plików cgroup](../../../.gitbook/assets/image%20(10)%20(2)%20(2).png)

Kluczowe pliki interfejsu dla cgroupów mają przedrostek **cgroup**. Plik **cgroup.procs**, który można wyświetlić za pomocą standardowych poleceń takich jak cat, wyświetla procesy wewnątrz cgroupu. Inny plik, **cgroup.threads**, zawiera informacje o wątkach.

![Procesy cgroupu](../../../.gitbook/assets/image%20(1)%20(1)%20(5).png)

Cgroupi zarządzające powłokami zwykle obejmują dwa kontrolery, które regulują użycie pamięci i liczbę procesów. Aby działać z kontrolerem, należy skonsultować pliki z przedrostkiem kontrolera. Na przykład, **pids.current** byłby odwołaniem do określenia liczby wątków w cgroupu.

![Pamięć cgroupu](../../../.gitbook/assets/image%20(3)%20(5).png)

Wartość **max** wskazuje brak określonego limitu dla cgroupu. Jednak ze względu na hierarchiczną naturę cgroupów, limity mogą być narzucane przez cgroup na niższym poziomie w hierarchii katalogów.


### Manipulowanie i tworzenie cgroupów

Procesy są przypisywane do cgroupów przez **zapisanie ich identyfikatora procesu (PID) do pliku `cgroup.procs`**. Wymaga to uprawnień roota. Na przykład, aby dodać proces:
```bash
echo [pid] > cgroup.procs
```
Podobnie, **modyfikowanie atrybutów cgroup, takich jak ustawienie limitu PID**, odbywa się poprzez zapisanie żądanej wartości do odpowiedniego pliku. Aby ustawić maksymalnie 3000 PID-ów dla cgroup:
```bash
echo 3000 > pids.max
```
**Tworzenie nowych cgroups** polega na utworzeniu nowego podkatalogu w hierarchii cgroup, co powoduje automatyczne wygenerowanie niezbędnych plików interfejsu przez jądro. Chociaż cgroups bez aktywnych procesów można usunąć za pomocą polecenia `rmdir`, należy pamiętać o pewnych ograniczeniach:

- **Procesy mogą być umieszczone tylko w liściowych cgroups** (czyli najbardziej zagnieżdżonych w hierarchii).
- **Cgroup nie może posiadać kontrolera, który nie istnieje w jego rodzicu**.
- **Kontrolery dla podgrup muszą być jawnie zadeklarowane** w pliku `cgroup.subtree_control`. Na przykład, aby włączyć kontrolery CPU i PID w podgrupie:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
**Grupa root cgroup** jest wyjątkiem od tych zasad, umożliwiając bezpośrednie umieszczanie procesów. Może to być wykorzystane do usunięcia procesów z zarządzania systemd.

**Monitorowanie użycia CPU** w obrębie grupy cgroup jest możliwe za pomocą pliku `cpu.stat`, który wyświetla łączny czas CPU zużyty przez procesy, co jest pomocne przy śledzeniu użycia w ramach podprocesów usługi:

<figure><img src="../../../.gitbook/assets/image (2) (6) (3).png" alt=""><figcaption>Statystyki użycia CPU widoczne w pliku cpu.stat</figcaption></figure>

## Referencje
* **Książka: Jak działa Linux, 3. wydanie: To, co każdy superużytkownik powinien wiedzieć autorstwa Briana Warda**

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć **reklamę swojej firmy w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLAN SUBSKRYPCJI**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi trikami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) **i** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **na GitHubie.**

</details>
