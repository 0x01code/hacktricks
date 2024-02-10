# Έλεγχος Σύνδεσης Διεργασίας XPC στο macOS

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs** στα αποθετήρια [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) στο github.

</details>

## Έλεγχος Σύνδεσης Διεργασίας XPC

Όταν γίνεται μια σύνδεση σε ένα XPC service, ο διακομιστής θα ελέγξει αν η σύνδεση επιτρέπεται. Αυτοί είναι οι έλεγχοι που συνήθως πραγματοποιούνται:

1. Έλεγχος αν η συνδεόμενη **διεργασία έχει υπογραφεί με ένα Apple-signed πιστοποιητικό** (που δίνεται μόνο από την Apple).
* Εάν αυτό **δεν επαληθευτεί**, ένας επιτιθέμενος μπορεί να δημιουργήσει ένα **ψεύτικο πιστοποιητικό** για να ταιριάξει με οποιονδήποτε άλλο έλεγχο.
2. Έλεγχος αν η συνδεόμενη διεργασία έχει υπογραφεί με το **πιστοποιητικό του οργανισμού** (έλεγχος ταυτότητας του ομάδας).
* Εάν αυτό **δεν επαληθευτεί**, μπορεί να χρησιμοποιηθεί οποιοδήποτε πιστοποιητικό ανάπτυξης από την Apple για την υπογραφή και τη σύνδεση με την υπηρεσία.
3. Έλεγχος αν η συνδεόμενη διεργασία **περιέχει έναν κατάλληλο αναγνωριστικό πακέτου**.
* Εάν αυτό **δεν επαληθευτεί**, οποιοδήποτε εργαλείο **που έχει υπογραφεί από τον ίδιο οργανισμό** μπορεί να χρησιμοποιηθεί για την αλληλεπίδραση με την XPC υπηρεσία.
4. (4 ή 5) Έλεγχος αν η συνδεόμενη διεργασία έχει έναν **κατάλληλο αριθμό έκδοσης λογισμικού**.
* Εάν αυτό **δεν επαληθευτεί**, μπορεί να χρησιμοποιηθεί μια παλιά, ευάλωτη πελάτης που είναι ευάλωτη στην ενέργεια εισαγωγής διεργασίας για να συνδεθεί με την XPC υπηρεσία, ακόμα και με τους άλλους ελέγχους που έχουν γίνει.
5. (4 ή 5) Έλεγχος αν η συνδεόμενη διεργασία έχει ενεργοποιημένο το hardened runtime χωρίς επικίνδυνα entitlements (όπως αυτά που επιτρέπουν τη φόρτωση αυθαίρετων βιβλιοθηκών ή τη χρήση μεταβλητών περιβάλλοντος DYLD)
1. Εάν αυτό **δεν επαληθευτεί**, ο πελάτης μπορεί να είναι **ευάλωτος σε εισαγωγή κώδικα**
6. Έλεγχος αν η συνδεόμενη διεργασία έχει ένα **entitlement** που της επιτρέπει να συνδεθεί στην υπηρεσία. Αυτό ισχύει για τα δυαδικά αρχεία της Apple.
7. Η **επαλήθευση** πρέπει να γίνεται **βάσει** του **αναγνωριστικού ελέγχου του πελάτη** αντί για το αναγνωριστικό διεργασίας (**PID**), καθώς το πρώτο αποτρέπει τις επιθέσεις επαναχρησιμοποίησης του PID.
* Οι προγραμματιστές **σπάνια χρησιμοποιούν την κλήση API του αναγνωριστικού ελέγχου** καθώς είναι **ιδιωτική**, οπότε η Apple μπορεί να την **αλλάξει** ανά πάσα στιγμή. Επιπλέον, η χρήση ιδιωτικής API δεν επιτρέπεται σε εφαρμογές του Mac App Store.
* Εάν χρησιμοποιηθεί η μέθοδος **`processIdentifier`**, μπορεί να είναι ευάλωτη
* Πρέπει να χρησιμοποιηθεί η μέθοδος **`xpc_dictionary_get_audit_token`** αντί της **`xpc_connection_get_audit_token`**, καθώς η τελευταία μπορεί επίσης να είναι [ευάλωτη σε συγκεκριμένες καταστάσεις](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/).

### Επιθέσεις Επικοινωνίας

Για περισσότερες πληροφορίες σχετικά με την επίθεση επαναχρησιμοποίησης του PID, ελέγξτε:

{% content-ref url="macos-pid-reuse.md" %}
[macos-pid-reuse.md](macos-pid-reuse.md)
{% endcontent-ref %}

Για περισσότερες πληροφορ
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
//Check connection
return YES;
}
```
{% endcode %}

Το αντικείμενο NSXPCConnection έχει μια **ιδιωτική** ιδιότητα **`auditToken`** (αυτή που πρέπει να χρησιμοποιηθεί αλλά μπορεί να αλλάξει) και μια **δημόσια** ιδιότητα **`processIdentifier`** (αυτή που δεν πρέπει να χρησιμοποιηθεί).

Η διαδικασία σύνδεσης μπορεί να επαληθευτεί με κάτι όπως:

{% code overflow="wrap" %}
```objectivec
[...]
SecRequirementRef requirementRef = NULL;
NSString requirementString = @"anchor apple generic and identifier \"xyz.hacktricks.service\" and certificate leaf [subject.CN] = \"TEAMID\" and info [CFBundleShortVersionString] >= \"1.0\"";
/* Check:
- Signed by a cert signed by Apple
- Check the bundle ID
- Check the TEAMID of the signing cert
- Check the version used
*/

// Check the requirements with the PID (vulnerable)
SecRequirementCreateWithString(requirementString, kSecCSDefaultFlags, &requirementRef);
SecCodeCheckValidity(code, kSecCSDefaultFlags, requirementRef);

// Check the requirements wuing the auditToken (secure)
SecTaskRef taskRef = SecTaskCreateWithAuditToken(NULL, ((ExtendedNSXPCConnection*)newConnection).auditToken);
SecTaskValidateForRequirement(taskRef, (__bridge CFStringRef)(requirementString))
```
{% endcode %}

Αν ένας προγραμματιστής δεν θέλει να ελέγξει την έκδοση του πελάτη, μπορεί τουλάχιστον να ελέγξει ότι ο πελάτης δεν είναι ευάλωτος στην ενέργεια εισαγωγής διεργασίας:

{% code overflow="wrap" %}
```objectivec
[...]
CFDictionaryRef csInfo = NULL;
SecCodeCopySigningInformation(code, kSecCSDynamicInformation, &csInfo);
uint32_t csFlags = [((__bridge NSDictionary *)csInfo)[(__bridge NSString *)kSecCodeInfoStatus] intValue];
const uint32_t cs_hard = 0x100;        // don't load invalid page.
const uint32_t cs_kill = 0x200;        // Kill process if page is invalid
const uint32_t cs_restrict = 0x800;    // Prevent debugging
const uint32_t cs_require_lv = 0x2000; // Library Validation
const uint32_t cs_runtime = 0x10000;   // hardened runtime
if ((csFlags & (cs_hard | cs_require_lv)) {
return Yes; // Accept connection
}
```
{% endcode %}

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
