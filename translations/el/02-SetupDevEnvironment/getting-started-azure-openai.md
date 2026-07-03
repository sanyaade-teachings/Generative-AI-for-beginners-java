# Ρύθμιση του Περιβάλλοντος Ανάπτυξης για το Azure AI Foundry

> Αυτός ο οδηγός ρυθμίζει τα μοντέλα **Azure AI Foundry** για τις εφαρμογές Java AI αυτού του μαθήματος, χρησιμοποιώντας **αυθεντικοποίηση χωρίς κλειδί** (Microsoft Entra ID) — χωρίς να χρειάζεται να διαχειρίζεστε κλειδιά API. Καινούργιος στα εργαλεία; Ξεκινήστε με τον [οδηγό περιβάλλοντος ανάπτυξης](./README.md).

Αυτός ο οδηγός ρυθμίζει τα μοντέλα **Azure AI Foundry** για τις εφαρμογές Java AI αυτού του μαθήματος. Έχετε δύο επιλογές:

- **Επιλογή Α — Παροχή με `azd` + Bicep (συνιστάται):** μία εντολή αναπτύσσει τον λογαριασμό Foundry και τα μοντέλα ως κώδικα. Χωρίς κλικ στο portal.
- **Επιλογή Β — Δημιουργία πόρων χειροκίνητα** στο portal του Azure AI Foundry.

Και οι δύο τρόποι χρησιμοποιούν **αυθεντικοποίηση χωρίς κλειδί** (Microsoft Entra ID) — δεν υπάρχουν κλειδιά API για αντιγραφή ή διαρροή.

## Πίνακας Περιεχομένων

- [Τι Δημιουργείται](#τι-δημιουργείται)
- [Προαπαιτούμενα](#προαπαιτούμενα)
- [Επιλογή Α: Παροχή με azd + Bicep (Συνιστάται)](#επιλογή-α-παροχή-με-azd--bicep-συνιστάται)
- [Επιλογή Β: Δημιουργία Πόρων Χειροκίνητα](#επιλογή-β-δημιουργία-πόρων-χειροκίνητα)
- [Ρύθμιση του Περιβάλλοντός σας](#ρύθμιση-του-περιβάλλοντός-σας)
- [Δοκιμή της Ρύθμισής σας](#δοκιμή-της-ρύθμισής-σας)
- [Τι Ακολουθεί;](#τι-ακολουθεί)
- [Πόροι](#πόροι)
- [Πρόσθετοι Πόροι](#πρόσθετοι-πόροι)

## Τι Δημιουργείται

Τα πρότυπα Bicep στον φάκελο [`infra/`](../../../02-SetupDevEnvironment/infra) παρέχουν:

- Λογαριασμό **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, τύπος `AIServices`) με ένα έργο
- Μια ανάπτυξη **chat** — `gpt-4o-mini`
- Μια ανάπτυξη **embedding** — `text-embedding-3-small` (χρησιμοποιείται σε επόμενα κεφάλαια)
- Μια **ανάθεση ρόλου χωρίς κλειδί** (`Cognitive Services OpenAI User`) ώστε να συνδεθείτε με `az login` αντί να διαχειρίζεστε κλειδιά

## Προαπαιτούμενα

- Ένα [Azure συνδρομή](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) και [Maven 3.9+](https://maven.apache.org/download.cgi)

## Επιλογή Α: Παροχή με azd + Bicep (Συνιστάται)

Από τον φάκελο `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Συνδεθείτε (και τα δύο εργαλεία)
azd auth login
az login

# Παρέχετε τον λογαριασμό Foundry + αναπτύξεις μοντέλων
azd up
```

Η εντολή `azd` ζητά ένα **όνομα περιβάλλοντος** (π.χ. `genai-java`) και μια **περιοχή**. Επιλέξτε μια περιοχή όπου είναι διαθέσιμα τα `gpt-4o-mini` και `text-embedding-3-small` — για παράδειγμα `eastus2` ή `swedencentral`.

Όταν ολοκληρωθεί η παροχή, το azd:

1. Αναπτύσσει όλα όσα ορίζονται στο [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Εκτελεί ένα postprovision hook που γράφει το [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) με το endpoint και τα ονόματα ανάπτυξης (χωρίς μυστικά).

> **Συμβουλή:** Εκτελέστε ξανά `azd up` οποτεδήποτε για να εφαρμόσετε αλλαγές. Τρέξτε `azd down` για να διαγράψετε τα πάντα και να σταματήσετε να επιβαρύνεστε οικονομικά.

Για να δείτε τις ρυθμίσεις που δημιουργήθηκαν:

```bash
azd env get-values
```

Τώρα πηγαίνετε στο [Δοκιμή της Ρύθμισής σας](#δοκιμή-της-ρύθμισής-σας).

## Επιλογή Β: Δημιουργία Πόρων Χειροκίνητα

Προτιμάτε το portal; Δημιουργήστε τους πόρους χειροκίνητα:

1. Μεταβείτε στο [portal Azure AI Foundry](https://ai.azure.com/) και συνδεθείτε.
2. **Δημιουργήστε ένα έργο** (αυτό δημιουργεί επίσης έναν πόρο AI Foundry). Δώστε του ένα όνομα όπως `GenAIJava`.
3. Στο έργο σας, ανοίξτε **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Αναπτύξτε το **gpt-4o-mini** (όνομα ανάπτυξης `gpt-4o-mini`). Επαναλάβετε για το **text-embedding-3-small** αν θέλετε τα παραδείγματα embedding.
5. Από την **Επισκόπηση**, αντιγράψτε το **endpoint** (π.χ. `https://<resource>.openai.azure.com/`).
6. Δώστε στον εαυτό σας πρόσβαση χωρίς κλειδί: στον πόρο, ανοίξτε **Έλεγχος πρόσβασης (IAM)** → **Προσθήκη ανάθεσης ρόλου** → αναθέστε **Cognitive Services OpenAI User** στον λογαριασμό σας.

> **Ακόμη έχετε πρόβλημα;** Δείτε την [τεκμηρίωση Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Ρύθμιση του Περιβάλλοντός σας

**Αν χρησιμοποιήσατε την Επιλογή Α (`azd up`)**, το αρχείο ρυθμίσεών σας έχει ήδη γραφτεί — δεν χρειάζεται καμία περαιτέρω ρύθμιση. Παραλείψτε και πηγαίνετε στο [Δοκιμή της Ρύθμισής σας](#δοκιμή-της-ρύθμισής-σας).

**Αν χρησιμοποιήσατε την Επιλογή Β (χειροκίνητα)**, δημιουργήστε μόνοι σας το αρχείο `.env` του παραδείγματος:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Επεξεργαστείτε το `.env` με το endpoint σας (χωρίς κλειδί — η αυθεντικοποίηση είναι χωρίς κλειδί):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Σημείωση ασφάλειας:** Δεν υπάρχει κλειδί API να αποθηκεύσετε. Αυθεντικοποιείστε μέσω Microsoft Entra ID με `az login` (τοπικά) ή με managed identity (στο Azure). Το αρχείο `.env` περιέχει μόνο μη μυστικές ρυθμίσεις και καλύπτεται ήδη από `.gitignore`.

## Δοκιμή της Ρύθμισής σας

Βεβαιωθείτε ότι είστε συνδεδεμένοι ώστε η αυθεντικοποίηση χωρίς κλειδί να λάβει ένα token, και στη συνέχεια τρέξτε το παράδειγμα:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # αν δεν έχετε ήδη συνδεθεί
mvn clean spring-boot:run
```

Θα πρέπει να δείτε μια απόκριση από το μοντέλο `gpt-4o-mini`!

> **Χρήστες VS Code:** Πατήστε `F5` για εκτέλεση. Η εφαρμογή φορτώνει αυτόματα το `.env` σας.

> **Πλήρες παράδειγμα:** Δείτε το [Βασικό Chat με Azure AI Foundry](./examples/basic-chat-azure/README.md) για λεπτομέρειες και αντιμετώπιση προβλημάτων.

## Τι Ακολουθεί;

**Η ρύθμιση ολοκληρώθηκε!** Τώρα διαθέτετε:
- Το Azure AI Foundry με τα `gpt-4o-mini` και `text-embedding-3-small` αναπτυγμένα
- Αυθεντικοποίηση χωρίς κλειδί (Microsoft Entra ID) — χωρίς κλειδιά για διαχείριση
- Ένα τοπικό `.env` με το endpoint και τα ονόματα ανάπτυξης
- Ένα περιβάλλον ανάπτυξης Java έτοιμο για χρήση

**Συνεχίστε στο** [Κεφάλαιο 3: Βασικές Τεχνικές Γεννητικής Τεχνητής Νοημοσύνης](../03-CoreGenerativeAITechniques/README.md) για να ξεκινήσετε την κατασκευή εφαρμογών AI!

## Πόροι

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Αυθεντικοποίηση χωρίς κλειδί με Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Τεκμηρίωση Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Documentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Πρόσθετοι Πόροι

- [Κατέβασμα VS Code](https://code.visualstudio.com/Download)
- [Κατέβασμα Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Διαμόρφωση Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Αποποίηση ευθυνών**:
Αυτό το έγγραφο έχει μεταφραστεί χρησιμοποιώντας την υπηρεσία μετάφρασης με τεχνητή νοημοσύνη [Co-op Translator](https://github.com/Azure/co-op-translator). Ενώ επιδιώκουμε την ακρίβεια, παρακαλούμε να έχετε υπόψη ότι οι αυτοματοποιημένες μεταφράσεις ενδέχεται να περιέχουν λάθη ή ανακρίβειες. Το πρωτότυπο έγγραφο στη μητρική του γλώσσα πρέπει να θεωρείται η αυθεντική πηγή. Για κρίσιμες πληροφορίες, συνιστάται επαγγελματική ανθρώπινη μετάφραση. Δεν φέρουμε ευθύνη για τυχόν παρεξηγήσεις ή λανθασμένες ερμηνείες που προκύπτουν από τη χρήση αυτής της μετάφρασης.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->