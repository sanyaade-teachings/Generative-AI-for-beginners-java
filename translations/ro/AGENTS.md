# AGENTS.md

## Prezentare generală a proiectului

Acesta este un depozit educațional pentru învățarea dezvoltării AI generative cu Java. Oferă un curs practic complet care acoperă Modelele Mari de Limbaj (LLM-uri), ingineria prompturilor, embedding-uri, RAG (Generare augmentată cu recuperare) și Protocolul Contextului Modelului (MCP).

**Tehnologii cheie:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI și SDK-urile OpenAI

**Arhitectură:**
- Mai multe aplicații Spring Boot independente organizate pe capitole
- Proiecte exemplu care demonstrează diferite modele AI
- Pregătit pentru GitHub Codespaces cu containere de dezvoltare preconfigurate

## Comenzi de configurare

### Cerințe preliminare
- Java 21 sau versiune superioară
- Maven 3.x
- Abonament Azure cu un deployment de model Azure AI Foundry (provisionare cu `azd up`)
- Azure CLI (`az`) și Azure Developer CLI (`azd`), autentificat pentru autentificare fără chei

### Configurarea mediului

**Opțiunea 1: GitHub Codespaces (Recomandat)**
```bash
# Creează un fork al depozitului și creează un codespace din interfața GitHub
# Containerul de dezvoltare va instala automat toate dependențele
# Așteaptă aproximativ 2 minute pentru configurarea mediului
```

**Opțiunea 2: Container de dezvoltare local**
```bash
# Clonează depozitul
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Deschide în VS Code cu extensia Dev Containers
# Redeschide în Container când ți se cere
```

**Opțiunea 3: Configurare locală**
```bash
# Instalează dependențele
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Verifică instalarea
java -version
mvn -version
```

### Configurare

**Configurare Azure AI Foundry (fără chei, recomandat):**
```bash
# Provisionează contul Foundry + implementările modelului ca cod
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd scrie examples/basic-chat-azure/.env cu punctul tău final (fără cheie)
```

**Configurare manuală a endpoint-ului:**
```bash
# Dacă nu ați folosit azd, setați singur punctul final (autentificarea rămâne fără cheie prin az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Editează .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Flux de lucru pentru dezvoltare

### Structura proiectului
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

### Rularea aplicațiilor

**Rularea unei aplicații Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Construirea unui proiect:**
```bash
cd [project-directory]
mvn clean install
```

**Pornirea MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Serverul rulează pe http://localhost:8080
```

**Rularea exemplelor de client:**
```bash
# După pornirea serverului într-un alt terminal
cd 04-PracticalSamples/calculator

# Client MCP direct
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Client bazat pe AI (necesită AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot interactiv
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools este inclus în proiectele care suportă încărcare la cald:
```bash
# Modificările fișierelor Java se vor reîncărca automat la salvare
mvn spring-boot:run
```

## Instrucțiuni pentru testare

### Rularea testelor

**Rulează toate testele dintr-un proiect:**
```bash
cd [project-directory]
mvn test
```

**Rulează testele cu output detaliat:**
```bash
mvn test -X
```

**Rulează o clasă de test specifică:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Structura testelor
- Fișierele de test folosesc JUnit 5 (Jupiter)
- Clasele de test sunt situate în `src/test/java/`
- Exemplele de client din proiectul calculator sunt în `src/test/java/com/microsoft/mcp/sample/client/`

### Testare manuală
Multe exemple sunt aplicații interactive care necesită testare manuală:

1. Porniți aplicația cu `mvn spring-boot:run`
2. Testați endpoint-urile sau interacționați cu CLI-ul
3. Verificați dacă comportamentul așteptat corespunde documentației din README.md-ul fiecărui proiect

### Testare cu Azure AI Foundry
- Autentificați-vă cu `az login` înainte de a rula exemplele (autentificare fără chei)
- Asigurați-vă că contul dvs. are rolul Cognitive Services OpenAI User pe resursa respectivă
- Testați filtrarea conținutului cu exemplul responsible AI din Capitolul 5

## Ghid stilistic pentru cod

### Convenții Java
- **Versiunea Java:** Java 21 cu funcționalități moderne
- **Stil:** Urmați convențiile standard Java
- **Denominare:** 
  - Clase: PascalCase
  - Metode/variabile: camelCase
  - Constante: UPPER_SNAKE_CASE
  - Numele pachetelor: lowercase

### Modele Spring Boot
- Folosiți `@Service` pentru logica de afaceri
- Folosiți `@RestController` pentru endpoint-urile REST
- Configurare prin `application.yml` sau `application.properties`
- Variabilele de mediu sunt preferate în locul valorilor hardcodate
- Folosiți adnotarea `@Tool` pentru metodele expuse prin MCP

### Organizarea fișierelor
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

### Dependențe
- Gestionate prin Maven `pom.xml`
- Spring AI BOM pentru gestionarea versiunilor
- LangChain4j pentru integrări AI
- Spring Boot starter parent pentru dependențele Spring

### Comentarii în cod
- Adăugați JavaDoc pentru API-urile publice
- Includeți comentarii explicative pentru interacțiuni AI complexe
- Documentați clar descrierile instrumentelor MCP

## Construire și deploy

### Construirea proiectelor

**Build fără teste:**
```bash
mvn clean install -DskipTests
```

**Build cu toate verificările:**
```bash
mvn clean install
```

**Pachetizare aplicație:**
```bash
mvn package
# Creează JAR în directorul target/
```

### Directorii de output
- Clase compilate: `target/classes/`
- Clase de test: `target/test-classes/`
- Fișiere JAR: `target/*.jar`
- Artefacte Maven: `target/`

### Configurare specifică mediului

**Dezvoltare:**
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

**Producție:**
- Folosiți o identitate administrată în loc de `az login` pentru autentificare fără chei
- Configurați `AZURE_OPENAI_ENDPOINT` să indice spre resursa Foundry de producție
- Configurați prin variabile de mediu sau Azure Key Vault

### Considerații pentru deploy
- Acesta este un depozit educațional cu aplicații exemplu
- Nu este conceput pentru deploy în producție ca atare
- Exemplele demonstrează modele ce pot fi adaptate pentru producție
- Consultați README-urile fiecărui proiect pentru note specifice de deploy

## Note suplimentare

### Azure AI Foundry
- **Autentificare fără chei:** conectați-vă cu Microsoft Entra ID — fără chei API de gestionat
- **Provisionare ca și cod:** Bicep + azd (`azd up`) creează contul și deployment-urile de model
- Același cod compatibil OpenAI rulează local (`az login`) și în Azure (identitate administrată)

### Lucrul cu mai multe proiecte
Fiecare proiect exemplu este independent:
```bash
# Navigați la proiectul specific
cd 04-PracticalSamples/[project-name]

# Fiecare are propriul pom.xml și poate fi construit independent
mvn clean install
```

### Probleme comune

**Neconcordanță versiune Java:**
```bash
# Verifică Java 21
java -version
# Actualizează JAVA_HOME dacă este necesar
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Probleme la descărcarea dependențelor:**
```bash
# Curățați memoria cache Maven și încercați din nou
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint sau autentificare indisponibilă:**
```bash
# Setează punctul final în sesiunea curentă și autentifică-te (fără cheie)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Sau folosește un fișier .env în directorul proiectului
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port deja utilizat:**
```bash
# Spring Boot folosește implicit portul 8080
# Schimbare în application.properties:
server.port=8081
```

### Suport multi-limbaj
- Documentația este disponibilă în peste 45 de limbi prin traducere automată
- Traducerile sunt în directorul `translations/`
- Traducerea este gestionată prin fluxul de lucru GitHub Actions

### Parcurs de învățare
1. Începeți cu [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Urmați capitolele în ordine (01 → 05)
3. Completați exemplele practice din fiecare capitol
4. Explorați proiectele exemplu din Capitolul 4
5. Învațați practicile responsible AI în Capitolul 5

### Containerul de dezvoltare
Fișierul `.devcontainer/devcontainer.json` configurează:
- Mediu de dezvoltare Java 21
- Maven preinstalat
- Extensii Java pentru VS Code
- Unelte Spring Boot
- Integrare GitHub Copilot
- Suport Docker-in-Docker
- Azure CLI

### Considerații de performanță
- Deploy-urile Azure AI Foundry au cote per minut pentru tokeni/cereri
- Folosiți dimensiuni de batch adecvate pentru embedding-uri
- Luați în considerare cache pentru apelurile API repetate
- Monitorizați consumul de tokeni pentru optimizarea costurilor

### Note de securitate
- Nu comiteți fișiere `.env` (sunt deja în `.gitignore`)
- Preferă autentificarea fără chei (Microsoft Entra ID) față de chei API
- Folosiți identități administrate în Azure; `az login` pentru dezvoltare locală
- Urmați ghidurile responsible AI din Capitolul 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare a responsabilității**:
Acest document a fost tradus folosind serviciul de traducere AI [Co-op Translator](https://github.com/Azure/co-op-translator). În timp ce ne străduim pentru acuratețe, vă rugăm să rețineți că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa autorizată. Pentru informații critice, se recomandă traducerea profesională realizată de un om. Nu ne asumăm responsabilitatea pentru eventualele neînțelegeri sau interpretări greșite care decurg din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->