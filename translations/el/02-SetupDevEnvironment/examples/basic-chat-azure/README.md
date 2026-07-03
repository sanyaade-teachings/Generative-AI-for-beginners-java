# Βασική Συνομιλία με το Azure AI Foundry - Παράδειγμα Από Άκρη Σε Άκρη

Αυτό το παράδειγμα είναι μια απλή εφαρμογή Spring Boot που συνδέεται με ένα μοντέλο **Azure AI Foundry** χρησιμοποιώντας **αυθεντικοποίηση χωρίς κλειδί** (Microsoft Entra ID) και ελέγχει τη ρύθμισή σας. Χρησιμοποιεί το `ChatClient` του Spring AI.

## Περιεχόμενα

- [Προαπαιτούμενα](#προαπαιτούμενα)
- [Γρήγορη Εκκίνηση](#γρήγορη-εκκίνηση)
- [Πώς Λειτουργεί η Αυθεντικοποίηση](#πώς-λειτουργεί-η-αυθεντικοποίηση)
- [Εκτέλεση της Εφαρμογής](#εκτέλεση-της-εφαρμογής)
  - [Χρήση Maven](#χρήση-maven)
  - [Χρήση VS Code](#χρήση-vs-code)
  - [Αναμενόμενη Έξοδος](#αναμενόμενη-έξοδος)
- [Αναφορά Ρύθμισης](#αναφορά-ρύθμισης)
  - [Μεταβλητές Περιβάλλοντος](#μεταβλητές-περιβάλλοντος)
  - [Ρύθμιση Spring](#ρύθμιση-spring)
- [Αντιμετώπιση Προβλημάτων](#αντιμετώπιση-προβλημάτων)
  - [Συνηθισμένα Προβλήματα](#συνηθισμένα-προβλήματα)
  - [Λειτουργία Αποσφαλμάτωσης](#λειτουργία-αποσφαλμάτωσης)
- [Επόμενα Βήματα](#επόμενα-βήματα)
- [Πόροι](#πόροι)

## Προαπαιτούμενα

Πριν εκτελέσετε αυτό το παράδειγμα, βεβαιωθείτε ότι έχετε:

- Έναν πόρο Azure AI Foundry με μια ανάπτυξη `gpt-4o-mini` — δημιουργήστε τον με `azd up` ή χειροκίνητα μέσω του [οδηγού ρύθμισης Azure AI Foundry](../../getting-started-azure-openai.md)
- Το ρόλο **Cognitive Services OpenAI User** σε αυτόν τον πόρο (τα πρότυπα Bicep το αναθέτουν για εσάς)
- Το [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), συνδεδεμένος με `az login`
- Java 21+ και Maven 3.9+

> **Δεν απαιτείται κλειδί API** — η αυθεντικοποίηση είναι χωρίς κλειδί μέσω Microsoft Entra ID.

## Γρήγορη Εκκίνηση

```bash
# 1. Μεταβείτε στο έργο
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Συνδεθείτε ώστε η πιστοποίηση χωρίς κλειδί να μπορεί να πάρει ένα διακριτικό
az login

# 3. Διαμορφώστε το τελικό σημείο
#    - Εάν εκτελέσατε `azd up`, το .env γράφτηκε για εσάς (παραλείψτε αυτό).
#    - Διαφορετικά αντιγράψτε το πρότυπο και ορίστε το AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Εκτελέστε την εφαρμογή
mvn spring-boot:run
```

## Πώς Λειτουργεί η Αυθεντικοποίηση

Αυτό το παράδειγμα αυθεντικοποιείται με **Microsoft Entra ID** — δεν υπάρχει κλειδί API.

Όταν έχει οριστεί μόνο το `spring.ai.azure.openai.endpoint` (και δεν υπάρχει api-key), το Spring AI δημιουργεί τον πελάτη Azure OpenAI με το [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Αυτή η διαπίστευση βρίσκει αυτόματα ένα διακριτικό από τη συνεδρία `az login` τοπικά, ή από μια διαχειριζόμενη ταυτότητα όταν εκτελείται στο Azure — οπότε ο ίδιος κώδικας λειτουργεί και στα δύο περιβάλλοντα χωρίς αλλαγές.

## Εκτέλεση της Εφαρμογής

### Χρήση Maven

```bash
mvn spring-boot:run
```

### Χρήση VS Code

1. Ανοίξτε το έργο στο VS Code  
2. Πατήστε `F5` ή χρησιμοποιήστε το πάνελ "Εκτέλεση και Αποσφαλμάτωση"  
3. Επιλέξτε τη ρύθμιση "Spring Boot-BasicChatApplication"

> **Σημείωση**: Η ρύθμιση του VS Code φορτώνει αυτόματα το αρχείο .env σας

### Αναμενόμενη Έξοδος

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## Αναφορά Ρύθμισης

### Μεταβλητές Περιβάλλοντος

| Μεταβλητή | Περιγραφή | Απαιτείται | Παράδειγμα |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Διεύθυνση endpoint Foundry (Azure OpenAI) | Ναι | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Όνομα ανάπτυξης μοντέλου συνομιλίας | Όχι | `gpt-4o-mini` (προεπιλογή) |

> Δεν υπάρχει μεταβλητή κλειδιού API — η αυθεντικοποίηση είναι χωρίς κλειδί (Microsoft Entra ID μέσω `az login`).

### Ρύθμιση Spring

Το αρχείο `application.yml` ρυθμίζει:  
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Από μεταβλητή περιβάλλοντος  
- **Ανάπτυξη**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Από μεταβλητή περιβάλλοντος με εναλλακτική  
- **Αυθεντικοποίηση**: χωρίς κλειδί — δεν ορίζεται `api-key`, οπότε το Spring AI χρησιμοποιεί `DefaultAzureCredential`  
- **Θερμοκρασία**: `0.7` - Ελέγχει τη δημιουργικότητα (0.0 = ντετερμινιστικό, 1.0 = δημιουργικό)  
- **Μέγιστος αριθμός tokens**: `500` - Μέγιστο μήκος απάντησης  

## Αντιμετώπιση Προβλημάτων

### Συνηθισμένα Προβλήματα

<details>
<summary><strong>Σφάλμα: 401 / "PermissionDenied" / σφάλματα διακριτικού</strong></summary>

- Εκτελέστε `az login` — η αυθεντικοποίηση χωρίς κλειδί χρειάζεται ενεργή σύνδεση για να λάβει διακριτικό  
- Επαληθεύστε ότι ο λογαριασμός σας έχει τον ρόλο **Cognitive Services OpenAI User** στον πόρο  
- Αν μόλις αναθέσατε το ρόλο, περιμένετε ένα λεπτό για να εφαρμοστεί  
- Επιβεβαιώστε ότι είστε στον σωστό tenant/συνδρομή (`az account show`)  
</details>

<details>
<summary><strong>Σφάλμα: "Το endpoint δεν είναι έγκυρο" / σφάλματα σύνδεσης</strong></summary>

- Βεβαιωθείτε ότι το `AZURE_OPENAI_ENDPOINT` είναι η πλήρης βασική διεύθυνση URL (π.χ., `https://your-resource.openai.azure.com/`)  
- Ελέγξτε για ομοιομορφία στο τελικό slash  
- Επαληθεύστε ότι το endpoint ταιριάζει με τον πόρο που έχετε δημιουργήσει (`azd env get-values`)  
</details>

<details>
<summary><strong>Σφάλμα: "Η ανάπτυξη δεν βρέθηκε"</strong></summary>

- Επαληθεύστε ότι το `AZURE_OPENAI_DEPLOYMENT` ταιριάζει με το όνομα μιας ανάπτυξης στο Azure  
- Ελέγξτε ότι το μοντέλο έχει αναπτυχθεί επιτυχώς και είναι ενεργό  
- Το προεπιλεγμένο όνομα ανάπτυξης είναι `gpt-4o-mini`  
</details>

<details>
<summary><strong>VS Code: Δεν φορτώνονται οι μεταβλητές περιβάλλοντος</strong></summary>

- Βεβαιωθείτε ότι το αρχείο `.env` βρίσκεται στον ριζικό φάκελο του έργου (στο ίδιο επίπεδο με το `pom.xml`)  
- Δοκιμάστε να εκτελέσετε `mvn spring-boot:run` στο ενσωματωμένο τερματικό του VS Code  
- Ελέγξτε ότι η επέκταση Java του VS Code είναι εγκατεστημένη σωστά  
</details>

### Λειτουργία Αποσφαλμάτωσης

Για να ενεργοποιήσετε λεπτομερείς καταγραφές, αποσχολιάστε αυτές τις γραμμές στο `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Επόμενα Βήματα

**Η Ρύθμισή σας Ολοκληρώθηκε!** Συνεχίστε το ταξίδι μάθησής σας:

[Κεφάλαιο 3: Κύριες Τεχνικές Παραγωγικής Τεχνητής Νοημοσύνης](../../../03-CoreGenerativeAITechniques/README.md)

## Πόροι

- [Τεκμηρίωση Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)  
- [Αυθεντικοποίηση χωρίς κλειδί με Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)  
- [Πύλη Azure AI Foundry](https://ai.azure.com/)  
- [Τεκμηρίωση Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Αποποίηση ευθυνών**:
Αυτό το έγγραφο έχει μεταφραστεί χρησιμοποιώντας την υπηρεσία μετάφρασης με τεχνητή νοημοσύνη [Co-op Translator](https://github.com/Azure/co-op-translator). Ενώ επιδιώκουμε την ακρίβεια, παρακαλούμε να έχετε υπόψη ότι οι αυτοματοποιημένες μεταφράσεις ενδέχεται να περιέχουν λάθη ή ανακρίβειες. Το πρωτότυπο έγγραφο στη μητρική του γλώσσα πρέπει να θεωρείται η αυθεντική πηγή. Για κρίσιμες πληροφορίες, συνιστάται επαγγελματική ανθρώπινη μετάφραση. Δεν φέρουμε ευθύνη για τυχόν παρεξηγήσεις ή λανθασμένες ερμηνείες που προκύπτουν από τη χρήση αυτής της μετάφρασης.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->