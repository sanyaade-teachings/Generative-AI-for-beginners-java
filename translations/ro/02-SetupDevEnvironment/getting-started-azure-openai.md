# Configurarea Mediului de Dezvoltare pentru Azure AI Foundry

> Acest ghid configurează modelele **Azure AI Foundry** pentru aplicațiile Java AI din acest curs, folosind autentificare **fără chei** (Microsoft Entra ID) — fără chei API de gestionat. Ești nou cu această unealtă? Începe cu [ghidul mediului de dezvoltare](./README.md).

Acest ghid configurează modelele **Azure AI Foundry** pentru aplicațiile Java AI din acest curs. Ai două opțiuni:

- **Opțiunea A — Provisionare cu `azd` + Bicep (recomandat):** o singură comandă implementează contul Foundry și modelele ca cod. Fără clicuri în portal.
- **Opțiunea B — Creează resursele manual** în portalul Azure AI Foundry.

Ambele opțiuni folosesc **autentificare fără chei** (Microsoft Entra ID) — nu există chei API de copiat sau scurs accidental.

## Cuprins

- [Ce se creează](#ce-se-creează)
- [Precondiții](#precondiții)
- [Opțiunea A: Provisionare cu azd + Bicep (Recomandat)](#option-a-provision-with-azd--bicep-recommended)
- [Opțiunea B: Creare resurse manual](#opțiunea-b-creare-resurse-manual)
- [Configurarea mediului](#configurarea-mediului)
- [Testarea configurației](#testarea-configurației)
- [Ce urmează?](#ce-urmează)
- [Resurse](#resurse)
- [Resurse suplimentare](#resurse-suplimentare)

## Ce se creează

Șabloanele Bicep din [`infra/`](../../../02-SetupDevEnvironment/infra) creează:

- Un cont **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, tip `AIServices`) cu un proiect
- O implementare **chat** — `gpt-4o-mini`
- O implementare **embedding** — `text-embedding-3-small` (folosită în capitolele ulterioare)
- O **atribuire de rol fără cheie** (`Cognitive Services OpenAI User`) astfel încât să te poți autentifica cu `az login` în loc să gestionezi chei

## Precondiții

- Un [abonament Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) și [Maven 3.9+](https://maven.apache.org/download.cgi)

## Opțiunea A: Provisionare cu azd + Bicep (Recomandat)

Din folderul `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Autentificare (ambele unelte)
azd auth login
az login

# Asigură contul Foundry + implementările modelelor
azd up
```

`azd` solicită un **nume de mediu** (de exemplu `genai-java`) și o **regiune**. Alege o regiune unde `gpt-4o-mini` și `text-embedding-3-small` sunt disponibile — de exemplu `eastus2` sau `swedencentral`.

Când provisioning-ul se termină, azd:

1. Implementează tot ce este definit în [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Rulează un hook post-provisionare care scrie [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) cu endpoint-ul tău și numele implementărilor (fără secrete).

> **Sfat:** Rulează din nou `azd up` oricând pentru a aplica modificări. Rulează `azd down` pentru a șterge totul și a opri acumularea costurilor.

Pentru a vedea setările generate:

```bash
azd env get-values
```

Acum săriți la [Testarea configurației](#testarea-configurației).

## Opțiunea B: Creare resurse manual

Preferi portalul? Creează resursele manual:

1. Accesează [portalul Azure AI Foundry](https://ai.azure.com/) și autentifică-te.
2. **Creează un proiect** (acesta creează și o resursă AI Foundry). Dă-i un nume, de exemplu `GenAIJava`.
3. În proiectul tău, deschide **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Implementează **gpt-4o-mini** (numele implementării `gpt-4o-mini`). Repetă pentru **text-embedding-3-small** dacă dorești exemplele cu embedding.
5. Din **Overview**, copiază **endpoint-ul** (de exemplu `https://<resource>.openai.azure.com/`).
6. Acordă-ți acces fără cheie: pe resursă, deschide **Access control (IAM)** → **Add role assignment** → atribuie **Cognitive Services OpenAI User** contului tău.

> **Încă ai probleme?** Vezi [documentația Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Configurarea mediului

**Dacă ai folosit Opțiunea A (`azd up`)**, fișierul tău de setări este deja scris — nu mai ai nimic de configurat. Sari la [Testarea configurației](#testarea-configurației).

**Dacă ai folosit Opțiunea B (manual)**, creează tu fișierul `.env` al exemplului:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Editează `.env` cu endpoint-ul tău (fără cheie — autentificarea este fără chei):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Notă de securitate:** Nu există nici o cheie API de stocat. Te autentifici cu Microsoft Entra ID prin `az login` (local) sau o identitate gestionată (în Azure). Fișierul `.env` conține doar setări non-secrete și este deja inclus în `.gitignore`.

## Testarea configurației

Asigură-te că ești autentificat pentru ca autentificarea fără chei să poată obține un token, apoi rulează exemplul:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # dacă nu sunteți deja autentificat
mvn clean spring-boot:run
```

Ar trebui să vezi un răspuns de la modelul `gpt-4o-mini`!

> **Utilizatori VS Code:** Apasă `F5` pentru a rula. Aplicația încarcă automat fișierul tău `.env`.

> **Exemplu complet:** Vezi [exemplul Basic Chat cu Azure AI Foundry](./examples/basic-chat-azure/README.md) pentru detalii și depanare.

## Ce urmează?

**Configurarea este completă!** Acum ai:
- Azure AI Foundry cu `gpt-4o-mini` și `text-embedding-3-small` implementate
- Autentificare fără chei (Microsoft Entra ID) — fără chei de gestionat
- Un `.env` local cu endpoint-ul și numele implementărilor tale
- Un mediu de dezvoltare Java gata de folosit

**Continuă cu** [Capitolul 3: Tehnici de bază în AI generativ](../03-CoreGenerativeAITechniques/README.md) pentru a începe să construiești aplicații AI!

## Resurse

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Autentificare fără chei cu Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Documentație Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Documentație Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [SDK Azure OpenAI pentru Java](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Resurse suplimentare

- [Descarcă VS Code](https://code.visualstudio.com/Download)
- [Descarcă Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Configurare Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare a responsabilității**:
Acest document a fost tradus folosind serviciul de traducere AI [Co-op Translator](https://github.com/Azure/co-op-translator). În timp ce ne străduim pentru acuratețe, vă rugăm să rețineți că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa autorizată. Pentru informații critice, se recomandă traducerea profesională realizată de un om. Nu ne asumăm responsabilitatea pentru eventualele neînțelegeri sau interpretări greșite care decurg din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->