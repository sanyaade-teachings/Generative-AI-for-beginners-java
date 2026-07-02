# Configurarea mediului de dezvoltare pentru Generative AI pentru Java

> **Pornire rapidă:** Provisionați modelele AI pe **Azure AI Foundry** ca și cod cu Bicep + `azd` în câteva minute — consultați [Ghidul de configurare Azure AI Foundry](getting-started-azure-openai.md). Autentificarea este **fără cheie** (Microsoft Entra ID), deci nu trebuie să gestionați chei API.

## Ce veți învăța

- Configurarea unui mediu de dezvoltare Java pentru aplicații AI
- Alegerea și configurarea mediului de dezvoltare preferat (cloud-first cu Codespaces, container de dezvoltare local sau configurare locală completă)
- Testarea configurației prin conectarea la un model Azure AI Foundry

## Cuprins

- [Ce veți învăța](#ce-veți-învăța)
- [Introducere](#introducere)
- [Pasul 1: Configurați mediul de dezvoltare](#pasul-1-configurați-mediul-de-dezvoltare)
  - [Opțiunea A: GitHub Codespaces (Recomandat)](#opțiunea-a-github-codespaces-recomandat)
  - [Opțiunea B: Container de dezvoltare local](#opțiunea-b-container-de-dezvoltare-local)
  - [Opțiunea C: Folosiți instalarea locală existentă](#opțiunea-c-folosiți-instalarea-locală-existentă)
- [Pasul 2: Provisionați Azure AI Foundry](#pasul-2-provisionați-azure-ai-foundry)
- [Pasul 3: Testați configurația](#pasul-3-testați-configurația)
- [Depanare](#depanare)
- [Sumar](#sumar)
- [Următorii pași](#următorii-pași)

## Introducere

Acest capitol vă va ghida în configurarea unui mediu de dezvoltare. Vom folosi **Azure AI Foundry** pentru modelele pe toată durata acestui curs. Provisionați modelele ca și cod cu Bicep și Azure Developer CLI (`azd`), apoi conectați-vă cu **autentificare fără cheie** (Microsoft Entra ID) — fără chei API de copiat sau scurs.

**Nu este necesară configurarea locală!** Puteți utiliza GitHub Codespaces, care oferă un mediu de dezvoltare complet în browser și provisioning pentru Foundry direct de acolo.

Folosim **Azure AI Foundry** pentru acest curs deoarece:
- **Provisionat ca și cod** — un singur `azd up` deployează contul și modelele
- **Fără cheie** — autentificare cu Azure sign-in sau o identitate gestionată
- **Pregătit pentru producție** — același cod rulează local și în Azure
- **Flexibil** — schimbați modelele prin modificarea numelui deployment-ului, nu a codului

> **Notă**: Deployment-urile Azure AI Foundry sunt taxate per token (pay-as-you-go). Consultați [Ghidul de configurare Azure AI Foundry](getting-started-azure-openai.md) pentru detalii despre provisioning, regiuni și costuri.


## Pasul 1: Configurați mediul de dezvoltare

<a name="quick-start-cloud"></a>

Am creat un container de dezvoltare preconfigurat pentru a minimiza timpul de configurare și pentru a vă asigura că aveți toate instrumentele necesare pentru acest curs Generative AI pentru Java. Alegeți-vă abordarea preferată:

### Opțiuni pentru configurarea mediului:

#### Opțiunea A: GitHub Codespaces (Recomandat)

**Începeți să programați în 2 minute - fără configurare locală!**

1. Faceți fork la acest repository pe contul dvs. GitHub
   > **Notă**: Dacă doriți să modificați configurația de bază, consultați [Configurarea containerului de dezvoltare](../../../.devcontainer/devcontainer.json)
2. Click pe **Code** → fila **Codespaces** → **...** → **New with options...**
3. Folosiți setările implicite – se va selecta **Dev container configuration**: **Generative AI Java Development Environment**, containerul dev special creat pentru acest curs
4. Click pe **Create codespace**
5. Așteptați ~2 minute până când mediul devine gata
6. Continuați la [Pasul 2: Provisionați Azure AI Foundry](#pasul-2-provisionați-azure-ai-foundry)

<img src="../../../translated_images/ro/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/ro/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/ro/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Avantajele Codespaces**:
> - Nu este necesară nicio instalare locală
> - Funcționează pe orice dispozitiv cu browser
> - Preconfigurat cu toate uneltele și dependențele
> - Gratuit 60 de ore pe lună pentru conturi personale
> - Mediu consistent pentru toți cursanții

#### Opțiunea B: Container de dezvoltare local

**Pentru dezvoltatorii care preferă dezvoltarea locală cu Docker**

1. Faceți fork și clonați acest repository pe mașina dvs. locală
   > **Notă**: Dacă doriți să modificați configurația de bază, consultați [Configurarea containerului de dezvoltare](../../../.devcontainer/devcontainer.json)
2. Instalați [Docker Desktop](https://www.docker.com/products/docker-desktop/) și [VS Code](https://code.visualstudio.com/)
3. Instalați extensia [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) în VS Code
4. Deschideți folderul repository-ului în VS Code
5. La prompt, faceți click pe **Reopen in Container** (sau folosiți `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Așteptați să se construiască și să pornească containerul
7. Continuați la [Pasul 2: Provisionați Azure AI Foundry](#pasul-2-provisionați-azure-ai-foundry)

<img src="../../../translated_images/ro/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/ro/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Opțiunea C: Folosiți instalarea locală existentă

**Pentru dezvoltatorii cu medii Java deja configurate**

Precondiții:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) sau IDE-ul preferat

Pași:
1. Clonați acest repository pe mașina dvs. locală
2. Deschideți proiectul în IDE-ul dvs.
3. Continuați la [Pasul 2: Provisionați Azure AI Foundry](#pasul-2-provisionați-azure-ai-foundry)

> **Sfat util**: Dacă aveți o mașină cu specificații mici, dar doriți să folosiți VS Code local, folosiți GitHub Codespaces! Puteți conecta VS Code local la un Codespace găzduit în cloud pentru a beneficia de ambele avantaje.

<img src="../../../translated_images/ro/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Pasul 2: Provisionați Azure AI Foundry

Deployați modelele AI din curs pe Azure AI Foundry ca și cod. Din rădăcina repository-ului:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` vă va cere un nume pentru mediu și o regiune, va provisiona un cont Azure AI Foundry cu deployment-uri pentru `gpt-4o-mini` și `text-embedding-3-small`, și va scrie endpoint-ul în exemplul `.env` — toate acestea folosind autentificare **fără cheie** (fără chei API).

> **Parcurgere completă:** Consultați [Ghidul de configurare Azure AI Foundry](getting-started-azure-openai.md) pentru precondiții, alternative manuale (portal), îndrumări privind regiunea și note despre costuri/curățare.

## Pasul 3: Testați configurația

După ce modelele Foundry sunt provisionate, testați conexiunea cu aplicația exemplu din [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Deschideți terminalul în mediul de dezvoltare.
2. Navigați la exemplu:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Asigurați-vă că sunteți autentificat (autentificarea fără cheie necesită token):
   ```bash
   az login
   ```
   > Dacă ați rulat `azd up`, fișierul `.env` cu endpoint-ul a fost deja creat pentru dvs.
4. Rulați aplicația:
   ```bash
   mvn clean spring-boot:run
   ```

Ar trebui să vedeți un răspuns de la modelul `gpt-4o-mini`.

### Înțelegerea codului exemplu

Exemplul de la `examples/basic-chat-azure` este o aplicație Spring Boot care folosește **Spring AI** pentru a se conecta la Azure AI Foundry cu autentificare fără cheie.

**Ce face acest cod:**
- **Se conectează** la Azure AI Foundry folosind sign-in-ul Azure (Microsoft Entra ID) — fără cheie API
- **Trimite** o întrebare către modelul `gpt-4o-mini`
- **Primește** și afișează răspunsul AI-ului
- **Validează** că setup-ul dvs. funcționează corect

**Dependența cheie** (în `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Configurare** (`application.yml`):
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

## Sumar

Perfect! Acum aveți totul configurat:

- Modele Azure AI Foundry provisionate ca și cod cu Bicep + `azd`
- Mediul de dezvoltare Java funcțional (fie Codespaces, containere de dezvoltare, sau local)
- Conexiune la Azure AI Foundry cu autentificare fără cheie (Microsoft Entra ID) — fără chei API
- Testat că totul funcționează cu un exemplu simplu care comunică cu modelul

## Următorii pași

[Capitolul 3: Tehnici de bază Generative AI](../03-CoreGenerativeAITechniques/README.md)

## Depanare

Aveți probleme? Iată probleme comune și soluții:

- **Autentificare eșuată (401/403)?** 
  - Rulați `az login` — autentificarea este fără cheie, trebuie să fiți autentificat
  - Verificați că aveți rolul **Cognitive Services OpenAI User** pe resursă
  - Dacă tocmai ați provisionat, așteptați un minut pentru propagarea rolului

- **Maven nu e găsit?** 
  - Dacă folosiți containere de dezvoltare/Codespaces, Maven trebuie să fie preinstalat
  - Pentru setup local, asigurați-vă că Java 21+ și Maven 3.9+ sunt instalate
  - Încercați `mvn --version` pentru a verifica instalarea

- **`azd` nu este găsit sau provisioning-ul eșuează?** 
  - Instalați [Azure Developer CLI](https://aka.ms/azure-dev/install) și rulați `azd auth login`
  - Alegeți o regiune unde este disponibil `gpt-4o-mini` (ex. `eastus2`)
  - Consultați [Ghidul de configurare Azure AI Foundry](getting-started-azure-openai.md) pentru detalii

- **Containerul de dezvoltare nu pornește?** 
  - Asigurați-vă că Docker Desktop este pornit (pentru dezvoltare locală)
  - Încercați să reconstruiți containerul: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Erori la compilarea aplicației?**
  - Verificați că sunteți în directorul corect: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Încercați să curățați și să reconstruiți: `mvn clean compile`

> **Aveți nevoie de ajutor?**: Dacă mai aveți probleme, deschideți un issue în repository și vă vom ajuta.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare a responsabilității**:
Acest document a fost tradus folosind serviciul de traducere AI [Co-op Translator](https://github.com/Azure/co-op-translator). În timp ce ne străduim pentru acuratețe, vă rugăm să rețineți că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa autorizată. Pentru informații critice, se recomandă traducerea profesională realizată de un om. Nu ne asumăm responsabilitatea pentru eventualele neînțelegeri sau interpretări greșite care decurg din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->