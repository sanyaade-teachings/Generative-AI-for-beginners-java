# Chat de bază cu Azure AI Foundry - Exemplu End-to-End

Acest exemplu este o aplicație simplă Spring Boot care se conectează la un model **Azure AI Foundry** folosind **autentificare fără cheie** (Microsoft Entra ID) și testează configurarea ta. Folosește `ChatClient` din Spring AI.

## Cuprins

- [Precondiții](#precondiții)
- [Pornire rapidă](#pornire-rapidă)
- [Cum funcționează autentificarea](#cum-funcționează-autentificarea)
- [Rularea aplicației](#rularea-aplicației)
  - [Folosind Maven](#folosind-maven)
  - [Folosind VS Code](#folosind-vs-code)
  - [Rezultatul așteptat](#rezultatul-așteptat)
- [Referință de configurare](#referință-de-configurare)
  - [Variabile de mediu](#variabile-de-mediu)
  - [Configurarea Spring](#configurarea-spring)
- [Depanare](#depanare)
  - [Probleme comune](#probleme-comune)
  - [Mod depanare](#mod-depanare)
- [Pașii următori](#pașii-următori)
- [Resurse](#resurse)

## Precondiții

Înainte de a rula acest exemplu, asigură-te că ai:

- O resursă Azure AI Foundry cu o implementare `gpt-4o-mini` — configureaz-o cu `azd up` sau manual prin [ghidul de configurare Azure AI Foundry](../../getting-started-azure-openai.md)
- Rolul **Cognitive Services OpenAI User** pentru acea resursă (șabloanele Bicep îl alocă automat)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), autentificat cu `az login`
- Java 21+ și Maven 3.9+

> **Nu este necesară o cheie API** — autentificarea este fără cheie, prin Microsoft Entra ID.

## Pornire rapidă

```bash
# 1. Navigați la proiect
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Autentificați-vă pentru ca autentificarea fără cheie să poată obține un token
az login

# 3. Configurați punctul final
#    - Dacă ați rulat `azd up`, .env a fost creat pentru dvs. (săriți acest pas).
#    - Altfel, copiați șablonul și setați AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Rulați aplicația
mvn spring-boot:run
```

## Cum funcționează autentificarea

Acest exemplu se autentifică cu **Microsoft Entra ID** — nu există cheie API.

Când este setată doar `spring.ai.azure.openai.endpoint` (fără api-key), Spring AI construiește clientul Azure OpenAI cu [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Această acreditare găsește automat un token din sesiunea ta locală `az login`, sau dintr-o identitate gestionată când rulezi în Azure — astfel același cod funcționează în ambele medii fără modificări.

## Rularea aplicației

### Folosind Maven

```bash
mvn spring-boot:run
```

### Folosind VS Code

1. Deschide proiectul în VS Code
2. Apasă `F5` sau folosește panoul "Run and Debug"
3. Selectează configurația "Spring Boot-BasicChatApplication"

> **Notă**: Configurația VS Code încarcă automat fișierul tău .env

### Rezultatul așteptat

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

## Referință de configurare

### Variabile de mediu

| Variabilă | Descriere | Obligatoriu | Exemplu |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL-ul endpoint-ului Foundry (Azure OpenAI) | Da | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Numele implementării modelului de chat | Nu | `gpt-4o-mini` (implicit) |

> Nu există variabilă pentru cheia API — autentificarea este fără cheie (Microsoft Entra ID prin `az login`).

### Configurarea Spring

Fișierul `application.yml` configurează:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Din variabila de mediu
- **Implementare**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Din variabila de mediu cu valoare implicită
- **Autentificare**: fără cheie — nu se setează `api-key`, deci Spring AI folosește `DefaultAzureCredential`
- **Temperature**: `0.7` - Controlează creativitatea (0.0 = determinist, 1.0 = creativ)
- **Max Tokens**: `500` - Lungimea maximă a răspunsului

## Depanare

### Probleme comune

<details>
<summary><strong>Eroare: 401 / "PermissionDenied" / erori token</strong></summary>

- Rulează `az login` — autentificare fără cheie are nevoie de un sign-in activ pentru a obține un token
- Verifică că contul tău are rolul **Cognitive Services OpenAI User** pe resursă
- Dacă tocmai ai alocat rolul, așteaptă un minut pentru propagare
- Confirmă că ești în tenant/abonamentul corect (`az account show`)
</details>

<details>
<summary><strong>Eroare: "The endpoint is not valid" / erori de conexiune</strong></summary>

- Asigură-te că `AZURE_OPENAI_ENDPOINT` este URL-ul complet de bază (exemplu: `https://your-resource.openai.azure.com/`)
- Verifică consistența slash-urilor finale
- Confirmă că endpoint-ul corespunde resursei tale provisionate (`azd env get-values`)
</details>

<details>
<summary><strong>Eroare: "The deployment was not found"</strong></summary>

- Verifică că `AZURE_OPENAI_DEPLOYMENT` corespunde unui nume de implementare în Azure
- Verifică că modelul este implementat cu succes și activ
- Numele implicit al implementării este `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Variabilele de mediu nu se încarcă</strong></summary>

- Asigură-te că fișierul `.env` este în directorul rădăcină al proiectului (la același nivel cu `pom.xml`)
- Încearcă să rulezi `mvn spring-boot:run` în terminalul integrat din VS Code
- Verifică dacă extensia Java pentru VS Code este instalată corect
</details>

### Mod depanare

Pentru a activa logarea detaliată, decomentează aceste linii din `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Pașii următori

**Configurare completă!** Continuă-ți parcursul de învățare:

[Capitolul 3: Tehnici Generative AI de bază](../../../03-CoreGenerativeAITechniques/README.md)

## Resurse

- [Documentația Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Autentificare fără cheie cu Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal Azure AI Foundry](https://ai.azure.com/)
- [Documentația Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare a responsabilității**:
Acest document a fost tradus folosind serviciul de traducere AI [Co-op Translator](https://github.com/Azure/co-op-translator). În timp ce ne străduim pentru acuratețe, vă rugăm să rețineți că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa autorizată. Pentru informații critice, se recomandă traducerea profesională realizată de un om. Nu ne asumăm responsabilitatea pentru eventualele neînțelegeri sau interpretări greșite care decurg din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->