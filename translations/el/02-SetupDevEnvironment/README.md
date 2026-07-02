# Ρύθμιση Περιβάλλοντος Ανάπτυξης για Γενετική Τεχνητή Νοημοσύνη με Java

> **Γρήγορη εκκίνηση:** Παρέχετε τα μοντέλα AI σας στο **Azure AI Foundry** ως κώδικα με Bicep + `azd` σε λίγα λεπτά — δείτε τον [Οδηγό Ρύθμισης Azure AI Foundry](getting-started-azure-openai.md). Η πιστοποίηση είναι **χωρίς κλειδί** (Microsoft Entra ID), οπότε δεν χρειάζεται διαχείριση κλειδιών API.

## Τι Θα Μάθετε

- Ρύθμιση περιβάλλοντος ανάπτυξης Java για εφαρμογές AI
- Επιλογή και διαμόρφωση του προτιμώμενου περιβάλλοντος ανάπτυξης (cloud-first με Codespaces, τοπικός dev container ή πλήρης τοπική ρύθμιση)
- Δοκιμή της ρύθμισής σας με σύνδεση σε μοντέλο Azure AI Foundry

## Περιεχόμενα

- [Τι Θα Μάθετε](#τι-θα-μάθετε)
- [Εισαγωγή](#εισαγωγή)
- [Βήμα 1: Ρύθμιση Περιβάλλοντος Ανάπτυξης](#βήμα-1-ρύθμιση-περιβάλλοντος-ανάπτυξης)
  - [Επιλογή Α: GitHub Codespaces (Προτεινόμενο)](#επιλογή-α-github-codespaces-προτεινόμενο)
  - [Επιλογή Β: Τοπικός Dev Container](#επιλογή-β-τοπικός-dev-container)
  - [Επιλογή Γ: Χρήση Υφιστάμενης Τοπικής Εγκατάστασης](#επιλογή-γ-χρήση-υφιστάμενης-τοπικής-εγκατάστασης)
- [Βήμα 2: Παροχή Azure AI Foundry](#βήμα-2-παροχή-azure-ai-foundry)
- [Βήμα 3: Δοκιμή της Ρύθμισής σας](#βήμα-3-δοκιμή-της-ρύθμισής-σας)
- [Επίλυση Προβλημάτων](#επίλυση-προβλημάτων)
- [Περίληψη](#περίληψη)
- [Επόμενα Βήματα](#επόμενα-βήματα)

## Εισαγωγή

Αυτό το κεφάλαιο θα σας καθοδηγήσει στη ρύθμιση ενός περιβάλλοντος ανάπτυξης. Θα χρησιμοποιήσουμε **Azure AI Foundry** για τα μοντέλα σε όλο το μάθημα. Παρέχετε τα μοντέλα ως κώδικα με Bicep και το Azure Developer CLI (`azd`), και μετά συνδέεστε με **πιστοποίηση χωρίς κλειδί** (Microsoft Entra ID) — δεν χρειάζεται να αντιγράψετε ή να διαρρεύσετε κλειδιά API.

**Δεν απαιτείται τοπική ρύθμιση!** Μπορείτε να χρησιμοποιήσετε GitHub Codespaces, που παρέχει πλήρες περιβάλλον ανάπτυξης στο πρόγραμμα περιήγησής σας, και να παράσχετε το Foundry από εκεί.

Χρησιμοποιούμε **Azure AI Foundry** για αυτό το μάθημα επειδή είναι:
- **Παρεχόμενο ως κώδικας** — ένα `azd up` αναπτύσσει τον λογαριασμό και τις αναπτύξεις μοντέλων
- **Χωρίς κλειδί** — αυθεντικοποίηση με το Azure sign-in σας ή μια διαχειριζόμενη ταυτότητα
- **Έτοιμο για παραγωγή** — ο ίδιος κώδικας τρέχει τοπικά και στο Azure
- **Ευέλικτο** — μπορείτε να αλλάξετε μοντέλα αλλάζοντας όνομα ανάπτυξης, όχι τον κώδικά σας

> **Σημείωση:** Οι αναπτύξεις στο Azure AI Foundry τιμολογούνται ανά token (πληρωμή ανά χρήση). Δείτε τον [οδηγό ρύθμισης Azure AI Foundry](getting-started-azure-openai.md) για λεπτομέρειες παροχής, περιοχής και κόστους.


## Βήμα 1: Ρύθμιση Περιβάλλοντος Ανάπτυξης

<a name="quick-start-cloud"></a>

Δημιουργήσαμε έναν προδιαμορφωμένο container ανάπτυξης για να ελαχιστοποιήσουμε τον χρόνο ρύθμισης και να εγγυηθούμε ότι έχετε όλα τα απαραίτητα εργαλεία για το μάθημα «Γενετική AI για Java». Επιλέξτε την προτιμώμενη προσέγγιση ανάπτυξης:

### Επιλογές Ρύθμισης Περιβάλλοντος:

#### Επιλογή Α: GitHub Codespaces (Προτεινόμενο)

**Ξεκινήστε να γράφετε κώδικα σε 2 λεπτά - δεν απαιτείται τοπική ρύθμιση!**

1. Κάντε fork αυτό το αποθετήριο στο λογαριασμό σας στο GitHub  
   > **Σημείωση:** Αν θέλετε να επεξεργαστείτε την βασική ρύθμιση, δείτε την [Διαμόρφωση Dev Container](../../../.devcontainer/devcontainer.json)
2. Πατήστε **Code** → καρτέλα **Codespaces** → **...** → **New with options...**
3. Χρησιμοποιήστε τις προεπιλογές – αυτό θα επιλέξει την **Διαμόρφωση Dev container**: **Generative AI Java Development Environment** προσαρμοσμένο devcontainer δημιουργημένο για αυτό το μάθημα
4. Πατήστε **Create codespace**
5. Περιμένετε ~2 λεπτά για να είναι έτοιμο το περιβάλλον
6. Προχωρήστε στο [Βήμα 2: Παροχή Azure AI Foundry](#βήμα-2-παροχή-azure-ai-foundry)

<img src="../../../translated_images/el/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/el/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/el/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Οφέλη των Codespaces**:
> - Δεν απαιτείται τοπική εγκατάσταση
> - Λειτουργεί σε οποιαδήποτε συσκευή με πρόγραμμα περιήγησης
> - Προεγκατεστημένα όλα τα εργαλεία και εξαρτήσεις
> - Δωρεάν 60 ώρες ανά μήνα για προσωπικούς λογαριασμούς
> - Ομοιογενές περιβάλλον για όλους τους εκπαιδευόμενους

#### Επιλογή Β: Τοπικός Dev Container

**Για προγραμματιστές που προτιμούν τοπική ανάπτυξη με Docker**

1. Κάντε fork και clone αυτό το αποθετήριο στη τοπική σας μηχανή  
   > **Σημείωση:** Αν θέλετε να επεξεργαστείτε την βασική ρύθμιση, δείτε την [Διαμόρφωση Dev Container](../../../.devcontainer/devcontainer.json)
2. Εγκαταστήστε το [Docker Desktop](https://www.docker.com/products/docker-desktop/) και το [VS Code](https://code.visualstudio.com/)
3. Εγκαταστήστε την επέκταση [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) στο VS Code
4. Ανοίξτε τον φάκελο του αποθετηρίου στο VS Code
5. Όταν σας ζητηθεί, πατήστε **Reopen in Container** (ή χρησιμοποιήστε `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Περιμένετε να ολοκληρωθεί η κατασκευή και εκκίνηση του container
7. Προχωρήστε στο [Βήμα 2: Παροχή Azure AI Foundry](#βήμα-2-παροχή-azure-ai-foundry)

<img src="../../../translated_images/el/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/el/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Επιλογή Γ: Χρήση Υφιστάμενης Τοπικής Εγκατάστασης

**Για προγραμματιστές με υπάρχον περιβάλλον Java**

Προαπαιτούμενα:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) ή το προτιμώμενο IDE σας

Βήματα:
1. Κλωνοποιήστε το αποθετήριο στη τοπική σας μηχανή
2. Ανοίξτε το έργο στο IDE σας
3. Προχωρήστε στο [Βήμα 2: Παροχή Azure AI Foundry](#βήμα-2-παροχή-azure-ai-foundry)

> **Συμβουλή**: Αν έχετε χαμηλής ισχύος μηχανή αλλά θέλετε VS Code τοπικά, χρησιμοποιήστε GitHub Codespaces! Μπορείτε να συνδέσετε τον τοπικό VS Code σας σε hosted Codespace στο cloud για τα καλύτερα και των δύο κόσμων.

<img src="../../../translated_images/el/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Βήμα 2: Παροχή Azure AI Foundry

Αναπτύξτε τα μοντέλα AI του μαθήματος στο Azure AI Foundry ως κώδικα. Από τη ρίζα του αποθετηρίου:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` ζητά όνομα περιβάλλοντος και περιοχή, παρέχει λογαριασμό Azure AI Foundry με αναπτύξεις `gpt-4o-mini` και `text-embedding-3-small`, και γράφει το endpoint στο αρχείο `.env` του παραδείγματος — όλα με **πιστοποίηση χωρίς κλειδί** (χωρίς κλειδιά API).

> **Ολοκληρωμένος οδηγός:** Δείτε τον [Οδηγό Ρύθμισης Azure AI Foundry](getting-started-azure-openai.md) για προαπαιτούμενα, εναλλακτική χειροκίνητη (portal), οδηγίες περιοχής και σημειώσεις κόστους/καθαρισμού.

## Βήμα 3: Δοκιμή της Ρύθμισής σας

Μόλις τα μοντέλα Foundry σας παρασχεθούν, δοκιμάστε τη σύνδεση με το παράδειγμα εφαρμογής στο [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Ανοίξτε το τερματικό στο περιβάλλον ανάπτυξής σας.  
2. Πλοηγηθείτε στο παράδειγμα:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Βεβαιωθείτε ότι έχετε συνδεθεί (η πιστοποίηση χωρίς κλειδί απαιτεί token):  
   ```bash
   az login
   ```
  
   > Αν κάνατε `azd up`, το αρχείο `.env` με το endpoint γράφτηκε ήδη για εσάς.  
4. Εκτελέστε την εφαρμογή:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Θα πρέπει να δείτε μια απόκριση από το μοντέλο `gpt-4o-mini`.

### Κατανόηση του Κώδικα Παράδειγματος

Το παράδειγμα στο `examples/basic-chat-azure` είναι μια εφαρμογή Spring Boot που χρησιμοποιεί **Spring AI** για σύνδεση με Azure AI Foundry με πιστοποίηση χωρίς κλειδί.

**Τι κάνει αυτός ο κώδικας:**
- **Συνδέεται** στο Azure AI Foundry με το Azure sign-in σας (Microsoft Entra ID) — χωρίς κλειδί API  
- **Στέλνει** ένα prompt στο μοντέλο `gpt-4o-mini`  
- **Λαμβάνει** και εμφανίζει την απόκριση της ΤΝ  
- **Επαληθεύει** ότι η ρύθμισή σας λειτουργεί σωστά

**Βασική εξάρτηση** (στο `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Διαμόρφωση** (`application.yml`):  
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```
  
## Περίληψη

Τέλεια! Πλέον έχετε όλα ρυθμισμένα:

- Παρέχετε μοντέλα Azure AI Foundry ως κώδικα με Bicep + `azd`
- Εκτελείτε το περιβάλλον ανάπτυξης Java σας (είτε Codespaces, dev containers, είτε τοπικά)
- Συνδέεστε στο Azure AI Foundry με πιστοποίηση χωρίς κλειδί (Microsoft Entra ID) — χωρίς κλειδιά API  
- Δοκιμάσατε ότι όλα λειτουργούν με ένα απλό παράδειγμα που επικοινωνεί με το μοντέλο σας  

## Επόμενα Βήματα

[Κεφάλαιο 3: Βασικές Τεχνικές Γενετικής AI](../03-CoreGenerativeAITechniques/README.md)

## Επίλυση Προβλημάτων

Αν αντιμετωπίζετε θέματα, δείτε κοινά προβλήματα και λύσεις:

- **Αποτυχία αυθεντικοποίησης (401/403);**  
  - Τρέξτε `az login` — η αυθεντικοποίηση είναι χωρίς κλειδί, οπότε πρέπει να είστε συνδεδεμένοι  
  - Ελέγξτε ότι ο λογαριασμός σας έχει τον ρόλο **Cognitive Services OpenAI User** στον πόρο  
  - Αν μόλις παρέχετε, περιμένετε ένα λεπτό για να διαδοθεί η ανάθεση ρόλου

- **Δεν βρέθηκε το Maven;**  
  - Αν χρησιμοποιείτε dev containers/Codespaces, το Maven πρέπει να είναι προεγκατεστημένο  
  - Για τοπική ρύθμιση, βεβαιωθείτε ότι έχετε Java 21+ και Maven 3.9+ εγκατεστημένα  
  - Δοκιμάστε `mvn --version` για επιβεβαίωση εγκατάστασης

- **Δεν βρέθηκε το `azd` ή αποτυγχάνει η παροχή;**  
  - Εγκαταστήστε το [Azure Developer CLI](https://aka.ms/azure-dev/install) και τρέξτε `azd auth login`  
  - Επιλέξτε περιοχή όπου είναι διαθέσιμο το `gpt-4o-mini` (π.χ. `eastus2`)  
  - Δείτε τον [οδηγό ρύθμισης Azure AI Foundry](getting-started-azure-openai.md) για λεπτομέρειες

- **Δεν ξεκινά ο dev container;**  
  - Βεβαιωθείτε ότι τρέχει το Docker Desktop (για τοπική ανάπτυξη)  
  - Δοκιμάστε να ξαναχτίσετε το container: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Σφάλματα μεταγλώττισης εφαρμογής;**  
  - Βεβαιωθείτε ότι είστε στον σωστό φάκελο: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Δοκιμάστε καθαρισμό και επαναμεταγλώττιση: `mvn clean compile`

> **Χρειάζεστε βοήθεια?**: Αν έχετε ακόμα προβλήματα, ανοίξτε ένα issue στο αποθετήριο και θα σας βοηθήσουμε.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Αποποίηση ευθυνών**:
Αυτό το έγγραφο έχει μεταφραστεί χρησιμοποιώντας την υπηρεσία μετάφρασης με τεχνητή νοημοσύνη [Co-op Translator](https://github.com/Azure/co-op-translator). Ενώ επιδιώκουμε την ακρίβεια, παρακαλούμε να έχετε υπόψη ότι οι αυτοματοποιημένες μεταφράσεις ενδέχεται να περιέχουν λάθη ή ανακρίβειες. Το πρωτότυπο έγγραφο στη μητρική του γλώσσα πρέπει να θεωρείται η αυθεντική πηγή. Για κρίσιμες πληροφορίες, συνιστάται επαγγελματική ανθρώπινη μετάφραση. Δεν φέρουμε ευθύνη για τυχόν παρεξηγήσεις ή λανθασμένες ερμηνείες που προκύπτουν από τη χρήση αυτής της μετάφρασης.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->