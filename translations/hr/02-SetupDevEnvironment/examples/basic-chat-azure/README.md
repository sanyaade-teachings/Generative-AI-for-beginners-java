# Osnovni chat s Azure AI Foundry - End-to-End primjer

Ovaj primjer je jednostavna Spring Boot aplikacija koja se povezuje s modelom **Azure AI Foundry** koristeći **autentifikaciju bez ključa** (Microsoft Entra ID) i testira vašu postavku. Koristi Spring AI `ChatClient`.

## Sadržaj

- [Preduvjeti](#preduvjeti)
- [Brzi početak](#brzi-početak)
- [Kako funkcionira autentifikacija](#kako-funkcionira-autentifikacija)
- [Pokretanje aplikacije](#pokretanje-aplikacije)
  - [Korištenje Maven-a](#korištenje-maven-a)
  - [Korištenje VS Code-a](#korištenje-vs-code-a)
  - [Očekivani ispis](#očekivani-ispis)
- [Referenca konfiguracije](#referenca-konfiguracije)
  - [Varijable okoline](#varijable-okoline)
  - [Spring konfiguracija](#spring-konfiguracija)
- [Rješavanje problema](#rješavanje-problema)
  - [Česti problemi](#česti-problemi)
  - [Debug način rada](#debug-način-rada)
- [Sljedeći koraci](#sljedeći-koraci)
- [Resursi](#resursi)

## Preduvjeti

Prije pokretanja ovog primjera, provjerite imate li:

- Azure AI Foundry resurs s `gpt-4o-mini` implementacijom — osigurajte ga pomoću `azd up` ili ručno putem [Azure AI Foundry vodiča za postavljanje](../../getting-started-azure-openai.md)
- Ulogu **Cognitive Services OpenAI User** na tom resursu (Bicep predlošci to dodjeljuju za vas)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), prijavljeni s `az login`
- Java 21+ i Maven 3.9+

> **Nije potreban API ključ** — autentifikacija je bez ključa putem Microsoft Entra ID.

## Brzi početak

```bash
# 1. Idite do projekta
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Prijavite se kako bi keyless autentifikacija mogla dobiti token
az login

# 3. Konfigurirajte krajnju točku
#    - Ako ste pokrenuli `azd up`, .env je napisan za vas (preskočite ovo).
#    - Inače kopirajte predložak i postavite AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Pokrenite aplikaciju
mvn spring-boot:run
```

## Kako funkcionira autentifikacija

Ovaj primjer se autentificira pomoću **Microsoft Entra ID** — nema API ključa.

Kada je postavljen samo `spring.ai.azure.openai.endpoint` (bez api-ključa), Spring AI gradi Azure OpenAI klijent s [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Ta vjerodajnica automatski pronalazi token iz vaše lokalne `az login` sesije, ili iz upravljanog identiteta pri radu u Azure-u — tako isti kod radi na oba mjesta bez promjena.

## Pokretanje aplikacije

### Korištenje Maven-a

```bash
mvn spring-boot:run
```

### Korištenje VS Code-a

1. Otvorite projekt u VS Code-u
2. Pritisnite `F5` ili koristite panel "Run and Debug"
3. Izaberite konfiguraciju "Spring Boot-BasicChatApplication"

> **Napomena**: VS Code konfiguracija automatski učitava vašu .env datoteku

### Očekivani ispis

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

## Referenca konfiguracije

### Varijable okoline

| Varijabla | Opis | Obavezno | Primjer |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) URL krajnje točke | Da | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Naziv implementacije chat modela | Ne | `gpt-4o-mini` (zadano) |

> Ne postoji varijabla za API ključ — autentifikacija je bez ključa (Microsoft Entra ID putem `az login`).

### Spring konfiguracija

Datoteka `application.yml` konfigurira:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - iz varijable okoline
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - iz varijable okoline sa zadanim vrijednostima
- **Auth**: bez ključa — nije postavljen `api-key`, pa Spring AI koristi `DefaultAzureCredential`
- **Temperature**: `0.7` - Kontrolira kreativnost (0.0 = deterministički, 1.0 = kreativan)
- **Max Tokens**: `500` - Maksimalna duljina odgovora

## Rješavanje problema

### Česti problemi

<details>
<summary><strong>Greška: 401 / "PermissionDenied" / pogreške tokena</strong></summary>

- Pokrenite `az login` — autentifikacija bez ključa treba aktivnu prijavu da dobije token
- Provjerite imate li ulogu **Cognitive Services OpenAI User** na resursu
- Ako ste tek dodijelili ulogu, pričekajte minutu da se propagira
- Potvrdite da ste u ispravnom tenantu/pretplati (`az account show`)
</details>

<details>
<summary><strong>Greška: "The endpoint is not valid" / pogreške veze</strong></summary>

- Provjerite je li `AZURE_OPENAI_ENDPOINT` potpuni osnovni URL (npr. `https://your-resource.openai.azure.com/`)
- Provjerite dosljednost završnog kosa crta
- Potvrdite da endpoint odgovara vašem osiguranom resursu (`azd env get-values`)
</details>

<details>
<summary><strong>Greška: "The deployment was not found"</strong></summary>

- Provjerite da `AZURE_OPENAI_DEPLOYMENT` odgovara nazivu implementacije u Azure-u
- Provjerite je li model uspješno implementiran i aktivan
- Zadani naziv implementacije je `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Varijable okoline se ne učitavaju</strong></summary>

- Provjerite je li vaša `.env` datoteka u korijenskom direktoriju projekta (na istoj razini kao `pom.xml`)
- Pokušajte pokrenuti `mvn spring-boot:run` u integriranom terminalu VS Code-a
- Provjerite je li VS Code Java ekstenzija pravilno instalirana
</details>

### Debug način rada

Za omogućavanje detaljnog zapisivanja, uklonite komentar s ovih linija u `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Sljedeći koraci

**Setup dovršen!** Nastavite svoj put učenja:

[3. Poglavlje: Osnovne tehnike generativne AI](../../../03-CoreGenerativeAITechniques/README.md)

## Resursi

- [Spring AI Azure OpenAI dokumentacija](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Autentifikacija bez ključa s Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portal](https://ai.azure.com/)
- [Azure AI Foundry dokumentacija](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Napomena**:
Ovaj dokument je preveden korištenjem AI prevoditeljskog servisa [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, imajte na umu da automatski prijevodi mogu sadržavati greške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporuča se profesionalni ljudski prijevod. Nismo odgovorni za bilo kakva nesporazumevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->