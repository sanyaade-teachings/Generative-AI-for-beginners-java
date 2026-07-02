# Postavljanje razvojne okoline za Azure AI Foundry

> Ovaj vodič postavlja **Azure AI Foundry** modele za Java AI aplikacije u ovom tečaju, koristeći **autentifikaciju bez ključeva** (Microsoft Entra ID) — nema API ključeva za upravljanje. Novi ste u alatima? Počnite s [vodičem za razvojnu okolinu](./README.md).

Ovaj vodič postavlja **Azure AI Foundry** modele za Java AI aplikacije u ovom tečaju. Imate dva načina:

- **Opcija A — Provision s `azd` + Bicep (preporučeno):** jedna naredba implementira Foundry račun i modele kao kod. Nema klikanja po portalu.
- **Opcija B — Ručno kreirajte resurse** u Azure AI Foundry portalu.

Oba načina koriste **autentifikaciju bez ključeva** (Microsoft Entra ID) — nema API ključeva za kopiranje ili curenje.

## Sadržaj

- [Što se kreira](#što-se-kreira)
- [Preduvjeti](#preduvjeti)
- [Opcija A: Provision s azd + Bicep (Preporučeno)](#option-a-provision-with-azd--bicep-recommended)
- [Opcija B: Ručno kreiranje resursa](#opcija-b-ručno-kreiranje-resursa)
- [Konfigurirajte svoju okolinu](#konfigurirajte-svoju-okolinu)
- [Testirajte svoju postavu](#testirajte-svoju-postavu)
- [Što dalje?](#što-dalje)
- [Resursi](#resursi)
- [Dodatni resursi](#dodatni-resursi)

## Što se kreira

Bicep predlošci u [`infra/`](../../../02-SetupDevEnvironment/infra) kreiraju:

- **Azure AI Foundry** račun (`Microsoft.CognitiveServices/accounts`, tip `AIServices`) s projektom
- **chat** implementaciju — `gpt-4o-mini`
- **embedding** implementaciju — `text-embedding-3-small` (koristi se u kasnijim poglavljima)
- **dodjelu uloge bez ključa** (`Cognitive Services OpenAI User`) tako da se prijavljujete s `az login` umjesto da upravljate ključevima

## Preduvjeti

- [Azure pretplata](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) i [Maven 3.9+](https://maven.apache.org/download.cgi)

## Opcija A: Provision s azd + Bicep (Preporučeno)

Iz mape `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Prijavite se (oba alata)
azd auth login
az login

# Osigurajte Foundry račun + implementacije modela
azd up
```

`azd` traži **ime okoline** (na primjer `genai-java`) i **regiju**. Odaberite regiju u kojoj su dostupni `gpt-4o-mini` i `text-embedding-3-small` — na primjer `eastus2` ili `swedencentral`.

Kad provisioning završi, azd:

1. Implementira sve definirano u [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Pokreće postprovision hook koji zapisuje [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) s vašom krajnjom točkom i imenima implementacija (bez tajni).

> **Savjet:** Pokrenite `azd up` bilo kada za primjenu promjena. Pokrenite `azd down` da izbrišete sve i prestanete stvarati troškove.

Da vidite generirane postavke:

```bash
azd env get-values
```

Sada preskočite na [Testirajte svoju postavu](#testirajte-svoju-postavu).

## Opcija B: Ručno kreiranje resursa

Više volite portal? Kreirajte resurse ručno:

1. Idite na [Azure AI Foundry portal](https://ai.azure.com/) i prijavite se.
2. **Kreirajte projekt** (ovo također stvara AI Foundry resurs). Dajte mu ime poput `GenAIJava`.
3. U vašem projektu otvorite **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Implementirajte **gpt-4o-mini** (ime implementacije `gpt-4o-mini`). Ponovite za **text-embedding-3-small** ako želite primjere ugradnje.
5. Iz **Pregleda**, kopirajte **krajnju točku** (na primjer `https://<resource>.openai.azure.com/`).
6. Dodijelite sebi pristup bez ključa: na resursu otvorite **Access control (IAM)** → **Add role assignment** → dodijelite **Cognitive Services OpenAI User** svom računu.

> **I dalje imate problema?** Pogledajte [dokumentaciju Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfigurirajte svoju okolinu

**Ako ste koristili Opciju A (`azd up`)**, vaša datoteka postavki je već zapisana — nema što konfigurirati. Preskočite na [Testirajte svoju postavu](#testirajte-svoju-postavu).

**Ako ste koristili Opciju B (ručno)**, sami napravite `.env` datoteku primjera:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Uredite `.env` s vašom krajnjom točkom (bez ključa — autentifikacija je bez ključa):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Sigurnosna napomena:** Nema API ključa za pohranu. Autentificirate se s Microsoft Entra ID putem `az login` (lokalno) ili putem managed identity (u Azureu). `.env` datoteka sadrži samo postavke koje nisu tajne i već je zaštićena s `.gitignore`.

## Testirajte svoju postavu

Provjerite jeste li prijavljeni da bi autentifikacija bez ključa mogla dohvatiti token, zatim pokrenite primjer:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # ako već niste prijavljeni
mvn clean spring-boot:run
```

Trebali biste vidjeti odgovor od modela `gpt-4o-mini`!

> **Koristite li VS Code?** Pritisnite `F5` za pokretanje. Aplikacija automatski učitava vašu `.env`.

> **Cijeli primjer:** Pogledajte [Osnovni Chat s Azure AI Foundry](./examples/basic-chat-azure/README.md) za detalje i rješavanje problema.

## Što dalje?

**Postavljanje završeno!** Sada imate:
- Azure AI Foundry s implementacijama `gpt-4o-mini` i `text-embedding-3-small`
- Autentifikaciju bez ključeva (Microsoft Entra ID) — nema ključeva za upravljanje
- Lokalnu `.env` datoteku s vašom krajnjom točkom i imenima implementacija
- Java razvojnu okolinu spremnu za rad

**Nastavite na** [Poglavlje 3: Temeljne tehnike generativnog AI](../03-CoreGenerativeAITechniques/README.md) da započnete s izradom AI aplikacija!

## Resursi

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Autentifikacija bez ključeva s Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Dokumentacija Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Dokumentacija](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Dodatni resursi

- [Preuzmite VS Code](https://code.visualstudio.com/Download)
- [Preuzmite Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Konfiguracija Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Napomena**:
Ovaj dokument je preveden korištenjem AI prevoditeljskog servisa [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, imajte na umu da automatski prijevodi mogu sadržavati greške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporuča se profesionalni ljudski prijevod. Nismo odgovorni za bilo kakva nesporazumevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->