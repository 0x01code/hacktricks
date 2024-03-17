# ACLs - DACLs/SACLs/ACEs

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Użyj [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), aby łatwo tworzyć i **automatyzować przepływy pracy** z wykorzystaniem najbardziej zaawansowanych narzędzi społecznościowych na świecie.\
Otrzymaj dostęp już dziś:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Zacznij od zera i zostań mistrzem hakowania AWS z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

## **Lista Kontroli Dostępu (ACL)**

Lista Kontroli Dostępu (ACL) składa się z uporządkowanego zestawu Pozycji Kontroli Dostępu (ACE), które określają zabezpieczenia obiektu i jego właściwości. W zasadzie ACL definiuje, które działania przez które podmioty bezpieczeństwa (użytkownicy lub grupy) są dozwolone lub zabronione dla danego obiektu.

Istnieją dwa rodzaje ACL:

* **Lista Kontroli Dostępu Dyskrecyjnego (DACL):** Określa, które użytkownicy i grupy mają lub nie mają dostępu do obiektu.
* **Lista Kontroli Dostępu Systemowego (SACL):** Zarządza audytem prób dostępu do obiektu.

Proces uzyskiwania dostępu do pliku polega na sprawdzeniu deskryptora zabezpieczeń obiektu w stosunku do tokena dostępu użytkownika, aby określić, czy dostęp powinien być udzielony i zakres tego dostępu, na podstawie ACE.

### **Kluczowe Składniki**

* **DACL:** Zawiera ACE, które przyznają lub odmawiają uprawnienia dostępu użytkownikom i grupom do obiektu. Jest to głównie ACL, które określa prawa dostępu.
* **SACL:** Używany do audytowania dostępu do obiektów, gdzie ACE definiują rodzaje dostępu do zapisania w Dzienniku Zdarzeń Bezpieczeństwa. Może to być nieocenione przy wykrywaniu prób nieautoryzowanego dostępu lub rozwiązywaniu problemów z dostępem.

### **Interakcja Systemu z ACL**

Każda sesja użytkownika jest powiązana z tokenem dostępu zawierającym informacje zabezpieczeń istotne dla tej sesji, w tym tożsamości użytkownika, grupy i uprawnień. Token ten zawiera również SID logowania, który jednoznacznie identyfikuje sesję.

Lokalny System Bezpieczeństwa (LSASS) przetwarza żądania dostępu do obiektów, sprawdzając DACL w poszukiwaniu ACE, które pasują do podmiotu bezpieczeństwa próbującego uzyskać dostęp. Dostęp jest natychmiastowo udzielany, jeśli nie zostaną znalezione odpowiednie ACE. W przeciwnym razie LSASS porównuje ACE z SID podmiotu bezpieczeństwa w tokenie dostępu, aby określić uprawnienia dostępu.

### **Podsumowany Proces**

* **ACL:** Definiuje uprawnienia dostępu za pomocą DACL i zasady audytu za pomocą SACL.
* **Token Dostępu:** Zawiera informacje o użytkowniku, grupie i uprawnieniach dla sesji.
* **Decyzja o Dostępie:** Podejmowana poprzez porównanie ACE DACL z tokenem dostępu; SACL są używane do audytu.

### ACE

Istnieją **trzy główne rodzaje Pozycji Kontroli Dostępu (ACE)**:

* **ACE Odmowy Dostępu**: Ten ACE wyraźnie odmawia dostępu do obiektu określonym użytkownikom lub grupom (w DACL).
* **ACE Zezwolenia Dostępu**: Ten ACE wyraźnie przyznaje dostęp do obiektu określonym użytkownikom lub grupom (w DACL).
* **ACE Audytu Systemowego**: Umieszczony w Systemowej Liście Kontroli Dostępu (SACL), ten ACE jest odpowiedzialny za generowanie dzienników audytu podczas prób dostępu do obiektu przez użytkowników lub grupy. Dokumentuje, czy dostęp został udzielony czy odmówiony oraz charakter dostępu.

Każda ACE ma **cztery istotne składniki**:

1. **Identyfikator Zabezpieczeń (SID)** użytkownika lub grupy (lub ich nazwę główną w reprezentacji graficznej).
2. **Flaga**, która identyfikuje typ ACE (odmowa dostępu, zezwolenie dostępu lub audyt systemowy).
3. **Flagi dziedziczenia**, które określają, czy obiekty potomne mogą dziedziczyć ACE od swojego rodzica.
4. [**Maska dostępu**](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-dtyp/7a53f60e-e730-4dfe-bbe9-b21b62eb790b?redirectedfrom=MSDN), 32-bitowa wartość określająca prawa przyznane obiektowi.

Decyzja o dostępie jest podejmowana poprzez sekwencyjne sprawdzanie każdej ACE, aż do:

* ACE **Odmowy Dostępu** wyraźnie odmawia żądanych praw trustee zidentyfikowanemu w tokenie dostępu.
* ACE **Zezwolenia Dostępu** wyraźnie przyznają wszystkie żądane prawa trustee w tokenie dostępu.
* Po sprawdzeniu wszystkich ACE, jeśli jakiekolwiek żądane prawo nie zostało wyraźnie zezwolone, dostęp jest domyślnie **odmawiany**.

### Kolejność ACE

Sposób, w jaki **ACE** (reguły określające, kto może lub nie może uzyskać dostęp do czegoś) są umieszczane na liście zwaną **DACL**, jest bardzo ważny. Dzieje się tak, ponieważ gdy system udziela lub odmawia dostępu na podstawie tych reguł, przestaje szukać dalej.

Istnieje najlepszy sposób organizacji tych ACE, nazywany **"kolejnością kanoniczną"**. Ta metoda pomaga zapewnić, że wszystko działa sprawnie i sprawiedliwie. Oto jak to wygląda w systemach takich jak **Windows 2000** i **Windows Server 2003**:

* Najpierw umieść wszystkie reguły, które są **specjalnie przeznaczone dla tego elementu**, przed tymi, które pochodzą skądś indziej, jak np. z folderu nadrzędnego.
* W tych specjalnych regułach, umieść te, które mówią **"nie" (odmowa)** przed tymi, które mówią **"tak" (zezwolenie)**.
* Dla reguł pochodzących skądś indziej, zacznij od tych z **najbliższego źródła**, jak rodzic, a następnie idź wstecz. Ponownie, umieść **"nie"** przed **"tak"**.

Taka konfiguracja pomaga w dwóch głównych aspektach:

* Zapewnia, że jeśli istnieje konkretne **"nie,"** jest ono respektowane, bez względu na to, jakie inne reguły **"tak"** są obecne.
* Pozwala właścicielowi elementu na **ostateczne decydowanie**, kto ma dostęp, zanim zacznie obowiązywać jakiekolwiek reguły z folderów nadrzędnych lub dalszych.

Dzięki temu podejściu właściciel pliku lub folderu może być bardzo precyzyjny co do tego, kto ma dostęp, dbając o to, aby odpowiednie osoby mogły uzyskać dostęp, a niewłaściwe nie mogły.

![](https://www.ntfs.com/images/screenshots/ACEs.gif)

Więc **"kolejność kanoniczna"** polega na zapewnieniu, że reguły dostępu są jasne i działają dobrze, umieszczając najpierw konkretne reguły i organizując wszystko w inteligentny sposób.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Użyj [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), aby łatwo tworzyć i **automatyzować przepływy pracy** z wykorzystaniem najbardziej zaawansowanych narzędzi społecznościowych na świecie.\
Otrzymaj dostęp już dziś:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
### Przykład interfejsu graficznego

[**Przykład stąd**](https://secureidentity.se/acl-dacl-sacl-and-the-ace/)

To klasyczna karta zabezpieczeń folderu pokazująca ACL, DACL i ACE:

![http://secureidentity.se/wp-content/uploads/2014/04/classicsectab.jpg](../../.gitbook/assets/classicsectab.jpg)

Jeśli klikniemy przycisk **Zaawansowane**, uzyskamy więcej opcji, takich jak dziedziczenie:

![http://secureidentity.se/wp-content/uploads/2014/04/aceinheritance.jpg](../../.gitbook/assets/aceinheritance.jpg)

A jeśli dodasz lub edytujesz podmiot zabezpieczeń:

![http://secureidentity.se/wp-content/uploads/2014/04/editseprincipalpointers1.jpg](../../.gitbook/assets/editseprincipalpointers1.jpg)

Na koniec mamy SACL w karcie Audytowanie:

![http://secureidentity.se/wp-content/uploads/2014/04/audit-tab.jpg](../../.gitbook/assets/audit-tab.jpg)

### Wyjaśnienie kontroli dostępu w uproszczony sposób

Podczas zarządzania dostępem do zasobów, takich jak folder, używamy list i reguł znanych jako Listy Kontroli Dostępu (ACL) i Wpisy Kontroli Dostępu (ACE). Określają one, kto może lub nie może uzyskać dostęp do określonych danych.

#### Odmawianie dostępu określonej grupie

Wyobraź sobie, że masz folder o nazwie Koszty i chcesz, aby wszyscy mieli do niego dostęp, z wyjątkiem zespołu marketingowego. Poprzez odpowiednie ustawienie reguł, możemy zapewnić, że zespół marketingowy jest wyraźnie pozbawiony dostępu przed udzieleniem dostępu pozostałym osobom. Robimy to umieszczając regułę odmowy dostępu dla zespołu marketingowego przed regułą, która zezwala na dostęp dla wszystkich innych.

#### Udzielanie dostępu określonemu członkowi zespołu, któremu odmówiono dostępu

Załóżmy, że Bob, dyrektor marketingu, potrzebuje dostępu do folderu Koszty, chociaż zespół marketingowy generalnie nie powinien mieć dostępu. Możemy dodać konkretną regułę (ACE) dla Boba, która przyznaje mu dostęp i umieścić ją przed regułą, która odmawia dostępu zespołowi marketingowemu. W ten sposób Bob uzyskuje dostęp pomimo ogólnego ograniczenia dla jego zespołu.

#### Zrozumienie Wpisów Kontroli Dostępu

ACE to indywidualne reguły w ACL. Określają one użytkowników lub grupy, określają, jakie dostępy są dozwolone lub zabronione, oraz określają, jak te reguły mają być dziedziczone przez podobiekty (dziedziczenie). Istnieją dwa główne rodzaje ACE:

* **ACE ogólne**: Stosuje się szeroko, wpływając albo na wszystkie typy obiektów, albo rozróżniając tylko między kontenerami (takimi jak foldery) a niekontenerami (takimi jak pliki). Na przykład reguła, która pozwala użytkownikom zobaczyć zawartość folderu, ale nie uzyskać dostępu do plików w nim.
* **ACE specyficzne dla obiektu**: Zapewniają bardziej precyzyjną kontrolę, pozwalając na ustawienie reguł dla konkretnych typów obiektów lub nawet poszczególnych właściwości w obiekcie. Na przykład w katalogu użytkowników reguła może pozwalać użytkownikowi zaktualizować swój numer telefonu, ale nie godziny logowania.

Każde ACE zawiera ważne informacje, takie jak do kogo reguła się odnosi (za pomocą identyfikatora zabezpieczeń lub SID), co reguła zezwala lub odmawia (za pomocą maski dostępu) i jak jest dziedziczona przez inne obiekty.

#### Kluczowe różnice między typami ACE

* **ACE ogólne** są odpowiednie dla prostych scenariuszy kontroli dostępu, gdzie ta sama reguła dotyczy wszystkich aspektów obiektu lub wszystkich obiektów w kontenerze.
* **ACE specyficzne dla obiektu** są używane w bardziej złożonych scenariuszach, zwłaszcza w środowiskach takich jak Active Directory, gdzie konieczne jest kontrolowanie dostępu do konkretnych właściwości obiektu w inny sposób.

Podsumowując, ACL i ACE pomagają zdefiniować precyzyjne kontrole dostępu, zapewniając, że tylko odpowiednie osoby lub grupy mają dostęp do poufnych informacji lub zasobów, z możliwością dostosowania praw dostępu do poziomu poszczególnych właściwości lub typów obiektów.

### Układ Wpisu Kontroli Dostępu

| Pole ACE   | Opis                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Typ        | Flaga wskazująca typ ACE. Systemy Windows 2000 i Windows Server 2003 obsługują sześć typów ACE: Trzy ogólne typy ACE, które są dołączone do wszystkich obiektów możliwych do zabezpieczenia. Trzy specyficzne dla obiektu typy ACE, które mogą wystąpić dla obiektów Active Directory.                                                                                                                                                                                                                                                            |
| Flagi       | Zestaw flag bitowych kontrolujących dziedziczenie i audytowanie.                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Rozmiar        | Liczba bajtów pamięci przeznaczonych na ACE.                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Maska dostępu | Wartość 32-bitowa, której bity odpowiadają prawom dostępu do obiektu. Bity mogą być ustawione na włączone lub wyłączone, ale znaczenie ustawienia zależy od typu ACE. Na przykład, jeśli bit odpowiadający prawu do odczytu uprawnień jest włączony, a typ ACE to Odmowa, ACE odmawia prawo do odczytu uprawnień obiektu. Jeśli ten sam bit jest ustawiony jako włączony, ale typ ACE to Zezwól, ACE przyznaje prawo do odczytu uprawnień obiektu. Więcej szczegółów na temat Maski dostępu znajduje się w następnej tabeli. |
| SID         | Identyfikuje użytkownika lub grupę, której dostęp jest kontrolowany lub monitorowany przez ten ACE.                                                                                                                                                                                                                                                                                                                                                                                                                                 |

### Układ Maski Dostępu

| Bit (Zakres) | Znaczenie                            | Opis/Przykład                       |
| ----------- | ---------------------------------- | ----------------------------------------- |
| 0 - 15      | Prawa dostępu specyficzne dla obiektu      | Odczyt danych, Wykonaj, Dodaj dane           |
| 16 - 22     | Standardowe prawa dostępu             | Usuń, Zapisz listę ACL, Zapisz właściciela            |
| 23          | Może uzyskać dostęp do listy ACL zabezpieczeń            |                                           |
| 24 - 27     | Zarezerwowane                           |                                           |
| 28          | Ogólny WSZYSTKO (Odczyt, Zapis, Wykonaj) | Wszystko poniżej                          |
| 29          | Ogólny Wykonaj                    | Wszystko, co niezbędne do uruchomienia programu |
| 30          | Ogólny Zapis                      | Wszystko, co niezbędne do zapisania pliku   |
| 31          | Ogólny Odczyt                       | Wszystko, co niezbędne do odczytu pliku       |

## Odnośniki

* [https://www.ntfs.com/ntfs-permissions-acl-use.htm](https://www.ntfs.com/ntfs-permissions-acl-use.htm)
* [https://secureidentity.se/acl-dacl-sacl-and-the-ace/](https://secureidentity.se/acl-dacl-sacl-and-the-ace/)
* [https://www.coopware.in2.info/\_ntfsacl\_ht.htm](https://www.coopware.in2.info/\_ntfsacl\_ht.htm)
