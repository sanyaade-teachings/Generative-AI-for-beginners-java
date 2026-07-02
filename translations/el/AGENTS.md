# AGENTS.md

## Επισκόπηση Έργου

Αυτό είναι ένα εκπαιδευτικό αποθετήριο για την εκμάθηση ανάπτυξης Γεννητικής Τεχνητής Νοημοσύνης με Java. Παρέχει ένα ολοκληρωμένο πρακτικό μάθημα που καλύπτει Μεγάλα Γλωσσικά Μοντέλα (LLMs), μηχανική προτροπών, embeddings, RAG (Retrieval-Augmented Generation) και το Πρωτόκολλο Πλαισίου Μοντέλου (MCP).

**Κύριες Τεχνολογίες:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, και OpenAI SDKs

**Αρχιτεκτονική:**
- Πολλαπλές αυτόνομες εφαρμογές Spring Boot οργανωμένες κατά κεφάλαια
- Δείγματα έργων που παρουσιάζουν διαφορετικά πρότυπα AI
- Έτοιμο για GitHub Codespaces με προδιαμορφωμένα περιβάλλοντα ανάπτυξης

## Εντολές Ρύθμισης

### Απαιτήσεις
- Java 21 ή νεότερη
- Maven 3.x
- Συνδρομή Azure με ανάπτυξη μοντέλου Azure AI Foundry (παρέχεται με `azd up`)
- Azure CLI (`az`) και Azure Developer CLI (`azd`), συνδεδεμένα για αυθεντικοποίηση χωρίς κλειδιά

### Ρύθμιση Περιβάλλοντος

**Επιλογή 1: GitHub Codespaces (Προτεινόμενο)**
```bash
# Δημιουργήστε ένα fork του αποθετηρίου και δημιουργήστε έναν κώδικα χώρου από το GitHub UI
# Το δοχείο ανάπτυξης θα εγκαταστήσει αυτόματα όλες τις εξαρτήσεις
# Περιμένετε περίπου 2 λεπτά για τη ρύθμιση του περιβάλλοντος
```

**Επιλογή 2: Τοπικό Container Ανάπτυξης**
```bash
# Κλωνοποίηση αποθετηρίου
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Άνοιγμα στο VS Code με την επέκταση Dev Containers
# Άνοιγμα ξανά στο Container όταν ζητηθεί
```

**Επιλογή 3: Τοπική Ρύθμιση**
```bash
# Εγκατάσταση εξαρτήσεων
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Επαλήθευση εγκατάστασης
java -version
mvn -version
```

### Διαμόρφωση

**Ρύθμιση Azure AI Foundry (χωρίς κλειδιά, προτεινόμενο):**
```bash
# Παρέχετε τον λογαριασμό Foundry + την υλοποίηση μοντέλων ως κώδικα
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# το azd γράφει το examples/basic-chat-azure/.env με το endpoint σας (χωρίς κλειδί)
```

**Χειροκίνητη ρύθμιση endpoint:**
```bash
# Εάν δεν χρησιμοποιήσατε το azd, ορίστε την κατάληξη μόνοι σας (η πιστοποίηση παραμένει χωρίς κλειδί μέσω az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Επεξεργαστείτε το .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Ροή Εργασίας Ανάπτυξης

### Δομή Έργου
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### Εκτέλεση Εφαρμογών

**Εκτέλεση εφαρμογής Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Κατασκευή έργου:**
```bash
cd [project-directory]
mvn clean install
```

**Εκκίνηση MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Ο διακομιστής τρέχει στο http://localhost:8080
```

**Εκτέλεση παραδειγμάτων πελάτη:**
```bash
# Μετά την εκκίνηση του διακομιστή σε άλλο τερματικό
cd 04-PracticalSamples/calculator

# Άμεσος πελάτης MCP
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Πελάτης με υποστήριξη AI (απαιτεί AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Διαδραστικό bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Ζεστή Επαναφόρτωση
Το Spring Boot DevTools περιλαμβάνεται στα έργα που υποστηρίζουν ζεστή επαναφόρτωση:
```bash
# Οι αλλαγές στα αρχεία Java θα επανεφορτώνονται αυτόματα όταν αποθηκεύονται
mvn spring-boot:run
```

## Οδηγίες Δοκιμών

### Εκτέλεση Δοκιμών

**Εκτέλεση όλων των δοκιμών σε έργο:**
```bash
cd [project-directory]
mvn test
```

**Εκτέλεση δοκιμών με λεπτομερή έξοδο:**
```bash
mvn test -X
```

**Εκτέλεση συγκεκριμένης κλάσης δοκιμών:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Δομή Δοκιμών
- Τα αρχεία δοκιμών χρησιμοποιούν JUnit 5 (Jupiter)
- Οι κλάσεις δοκιμών βρίσκονται στο `src/test/java/`
- Παραδείγματα πελάτη στο έργο calculator είναι στο `src/test/java/com/microsoft/mcp/sample/client/`

### Χειροκίνητη Δοκιμή
Πολλά παραδείγματα είναι διαδραστικές εφαρμογές που απαιτούν χειροκίνητη δοκιμή:

1. Εκκινήστε την εφαρμογή με `mvn spring-boot:run`
2. Δοκιμάστε τα endpoints ή αλληλεπιδράστε με το CLI
3. Επιβεβαιώστε ότι η αναμενόμενη συμπεριφορά ταιριάζει με την τεκμηρίωση σε κάθε README.md έργου

### Δοκιμή με Azure AI Foundry
- Συνδεθείτε με `az login` πριν εκτελέσετε παραδείγματα (χωρίς κλειδιά)
- Βεβαιωθείτε ότι ο λογαριασμός σας έχει ρόλο Cognitive Services OpenAI User στο resource
- Δοκιμάστε φιλτράρισμα περιεχομένου με το παράδειγμα υπεύθυνης AI στο Κεφάλαιο 5

## Οδηγίες Στυλ Κώδικα

### Συμβάσεις Java
- **Έκδοση Java:** Java 21 με σύγχρονες λειτουργίες
- **Στυλ:** Τηρείτε τις τυπικές συμβάσεις Java
- **Ονοματοδοσία:** 
  - Κλάσεις: PascalCase
  - Μέθοδοι/μεταβλητές: camelCase
  - Σταθερές: UPPER_SNAKE_CASE
  - Ονόματα πακέτων: πεζά

### Πρότυπα Spring Boot
- Χρήση `@Service` για επιχειρησιακή λογική
- Χρήση `@RestController` για REST endpoints
- Ρύθμιση μέσω `application.yml` ή `application.properties`
- Μεταβλητές περιβάλλοντος προτιμούνται έναντι σταθερών τιμών στο κώδικα
- Χρήση της σημείωσης `@Tool` για μεθόδους που εκτίθενται μέσω MCP

### Οργάνωση Αρχείων
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### Εξαρτήσεις
- Διαχείριση μέσω Maven `pom.xml`
- Spring AI BOM για διαχείριση εκδόσεων
- LangChain4j για ενσωματώσεις AI
- Spring Boot starter parent για εξαρτήσεις Spring

### Σχόλια Κώδικα
- Προσθέστε JavaDoc για δημόσια API
- Συμπεριλάβετε επεξηγηματικά σχόλια για σύνθετες αλληλεπιδράσεις AI
- Τεκμηριώστε σαφώς τις περιγραφές εργαλείων MCP

## Κατασκευή και Ανάπτυξη

### Κατασκευή Έργων

**Κατασκευή χωρίς δοκιμές:**
```bash
mvn clean install -DskipTests
```

**Κατασκευή με όλους τους ελέγχους:**
```bash
mvn clean install
```

**Πακετάρισμα εφαρμογής:**
```bash
mvn package
# Δημιουργεί JAR στον φάκελο target/
```

### Φάκελοι Εξόδου
- Συμπιεσμένες κλάσεις: `target/classes/`
- Κλάσεις δοκιμών: `target/test-classes/`
- Αρχεία JAR: `target/*.jar`
- Αντικείμενα Maven: `target/`

### Διαμόρφωση ανά Περιβάλλον

**Ανάπτυξη:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**Παραγωγή:**
- Χρησιμοποιήστε διαχειριζόμενη ταυτότητα αντί για `az login` για αυθεντικοποίηση χωρίς κλειδιά
- Δείξτε το `AZURE_OPENAI_ENDPOINT` στο πόρο Foundry παραγωγής σας
- Διαχειριστείτε ρυθμίσεις μέσω μεταβλητών περιβάλλοντος ή Azure Key Vault

### Σκέψεις Ανάπτυξης
- Πρόκειται για εκπαιδευτικό αποθετήριο με δείγματα εφαρμογών
- Δεν είναι σχεδιασμένο για παραγωγική ανάπτυξη όπως έχει
- Τα δείγματα παρουσιάζουν πρότυπα για προσαρμογή σε παραγωγικό περιβάλλον
- Δείτε τα README κάθε έργου για συγκεκριμένες σημειώσεις ανάπτυξης

## Πρόσθετες Σημειώσεις

### Azure AI Foundry
- **Αυθεντικοποίηση χωρίς κλειδιά:** σύνδεση με Microsoft Entra ID — χωρίς διαχείριση API κλειδιών
- **Παροχή ως κώδικας:** Bicep + azd (`azd up`) δημιουργούν τον λογαριασμό και τις αναπτύξεις μοντέλων
- Ο ίδιος κώδικας συμβατός με OpenAI εκτελείται τοπικά (`az login`) και στο Azure (managed identity)

### Εργασία με Πολλαπλά Έργα
Κάθε δείγμα έργου είναι αυτόνομο:
```bash
# Πλοηγηθείτε σε συγκεκριμένο έργο
cd 04-PracticalSamples/[project-name]

# Κάθε ένα έχει το δικό του pom.xml και μπορεί να κατασκευαστεί ανεξάρτητα
mvn clean install
```

### Συνήθη Προβλήματα

**Ασυμφωνία Έκδοσης Java:**
```bash
# Επαληθεύστε το Java 21
java -version
# Ενημερώστε το JAVA_HOME εάν απαιτείται
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Προβλήματα Λήψης Εξαρτήσεων:**
```bash
# Εκκαθαρίστε την προσωρινή μνήμη Maven και δοκιμάστε ξανά
rm -rf ~/.m2/repository
mvn clean install
```

**Μη Εύρεση Endpoint ή Σύνδεσης:**
```bash
# Ορίστε το τελικό σημείο στη τρέχουσα συνεδρία και συνδεθείτε (χωρίς κλειδί)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Ή χρησιμοποιήστε ένα αρχείο .env στον κατάλογο του έργου
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Θύρα ήδη σε χρήση:**
```bash
# Το Spring Boot χρησιμοποιεί την πόρτα 8080 από προεπιλογή
# Αλλαγή στο application.properties:
server.port=8081
```

### Υποστήριξη Πολλών Γλωσσών
- Διαθέσιμη τεκμηρίωση σε 45+ γλώσσες μέσω αυτοματοποιημένης μετάφρασης
- Μεταφράσεις στον φάκελο `translations/`
- Η διαχείριση μετάφρασης γίνεται μέσω ροής εργασίας GitHub Actions

### Μονοπάτι Μάθησης
1. Ξεκινήστε με το [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Ακολουθήστε τα κεφάλαια με σειρά (01 → 05)
3. Ολοκληρώστε πρακτικά παραδείγματα σε κάθε κεφάλαιο
4. Εξερευνήστε δείγματα έργων στο Κεφάλαιο 4
5. Μάθετε υπεύθυνες πρακτικές AI στο Κεφάλαιο 5

### Περιβάλλον Ανάπτυξης
Το `.devcontainer/devcontainer.json` ρυθμίζει:
- Περιβάλλον ανάπτυξης Java 21
- Προεγκατεστημένο Maven
- Επεκτάσεις Java για VS Code
- Εργαλεία Spring Boot
- Ενσωμάτωση GitHub Copilot
- Υποστήριξη Docker-in-Docker
- Azure CLI

### Σκέψεις Απόδοσης
- Οι αναπτύξεις Azure AI Foundry έχουν όρια token/ερώτησης ανά λεπτό
- Χρησιμοποιήστε κατάλληλα μεγέθη batch για embeddings
- Σκεφτείτε caching για επαναλαμβανόμενες κλήσεις API
- Παρακολουθείτε τη χρήση token για βελτιστοποίηση κόστους

### Σημειώσεις Ασφάλειας
- Μην προσθέτετε αρχεία `.env` στο αποθετήριο (είναι ήδη στο `.gitignore`)
- Προτιμήστε αυθεντικοποίηση χωρίς κλειδιά (Microsoft Entra ID) αντί για API keys
- Χρησιμοποιήστε διαχειριζόμενες ταυτότητες στο Azure · `az login` για τοπική ανάπτυξη
- Ακολουθήστε οδηγίες υπεύθυνης χρήσης AI στο Κεφάλαιο 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Αποποίηση ευθυνών**:
Αυτό το έγγραφο έχει μεταφραστεί χρησιμοποιώντας την υπηρεσία μετάφρασης με τεχνητή νοημοσύνη [Co-op Translator](https://github.com/Azure/co-op-translator). Ενώ επιδιώκουμε την ακρίβεια, παρακαλούμε να έχετε υπόψη ότι οι αυτοματοποιημένες μεταφράσεις ενδέχεται να περιέχουν λάθη ή ανακρίβειες. Το πρωτότυπο έγγραφο στη μητρική του γλώσσα πρέπει να θεωρείται η αυθεντική πηγή. Για κρίσιμες πληροφορίες, συνιστάται επαγγελματική ανθρώπινη μετάφραση. Δεν φέρουμε ευθύνη για τυχόν παρεξηγήσεις ή λανθασμένες ερμηνείες που προκύπτουν από τη χρήση αυτής της μετάφρασης.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->