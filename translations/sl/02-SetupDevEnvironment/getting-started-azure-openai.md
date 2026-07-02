# Nastavitev razvojnega okolja za Azure AI Foundry

> Ta vodič nastavi modele **Azure AI Foundry** za Java AI aplikacije v tem tečaju, z uporabo **avtentikacije brez ključev** (Microsoft Entra ID) — ni treba upravljati API ključev. Ste novi pri orodjih? Začnite z [vodičem za razvojno okolje](./README.md).

Ta vodič nastavi modele **Azure AI Foundry** za Java AI aplikacije v tem tečaju. Imate dve poti:

- **Možnost A — Zagotovitev s `azd` + Bicep (priporočeno):** ena ukazna vrstica za namestitev računa Foundry in modelov kot koda. Brez klikanja po portalu.
- **Možnost B — Ročna ustvaritev virov** v portalu Azure AI Foundry.

Obe poti uporabljata **avtentikacijo brez ključev** (Microsoft Entra ID) — ni treba kopirati ali razkriti API ključev.

## Vsebina

- [Kaj se ustvari](#kaj-se-ustvari)
- [Pogoji](#pogoji)
- [Možnost A: Zagotovitev s pomočjo azd + Bicep (priporočeno)](#option-a-provision-with-azd--bicep-recommended)
- [Možnost B: Ročna ustvaritev virov](#možnost-b-ročna-ustvaritev-virov)
- [Konfiguracija okolja](#konfiguracija-okolja)
- [Preizkus nastavitve](#preizkus-nastavitve)
- [Kaj sledi?](#kaj-sledi)
- [Viri](#viri)
- [Dodatni viri](#dodatni-viri)

## Kaj se ustvari

Bicep predloge v [`infra/`](../../../02-SetupDevEnvironment/infra) zagotovijo:

- Račun **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, vrsta `AIServices`) s projektom
- Namestitev **chat** — `gpt-4o-mini`
- Namestitev **embedding** — `text-embedding-3-small` (uporabljeno v kasnejših poglavjih)
- Dodelitev vloge brez ključev (`Cognitive Services OpenAI User`), da se prijavite z `az login` namesto upravljanja ključev

## Pogoji

- [Azure naročnina](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) in [Maven 3.9+](https://maven.apache.org/download.cgi)

## Možnost A: Zagotovitev s pomočjo azd + Bicep (priporočeno)

Iz mape `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Prijava (obe orodji)
azd auth login
az login

# Zagotavljanje računa Foundry + nameščanja modelov
azd up
```

`azd` zahteva **ime okolja** (na primer `genai-java`) in **regijo**. Izberite regijo, kjer sta na voljo `gpt-4o-mini` in `text-embedding-3-small` — na primer `eastus2` ali `swedencentral`.

Ko se zagotavljanje konča, azd:

1. Namesti vse, kar je definirano v [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Izvede postprovision hook, ki zapiše [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) z vašim končnim točkam in imeni namestitev (brez skrivnosti).

> **Namig:** Za izvajanje sprememb znova zaženite `azd up`. Za izbris vsega in prekinitev stroškov zaženite `azd down`.

Za ogled ustvarjenih nastavitev:

```bash
azd env get-values
```

Sedaj preskočite na [Preizkus nastavitve](#preizkus-nastavitve).

## Možnost B: Ročna ustvaritev virov

Raje uporabljate portal? Ustvarite vire ročno:

1. Pojdite na [Azure AI Foundry portal](https://ai.azure.com/) in se prijavite.
2. **Ustvarite projekt** (to ustvari tudi vir AI Foundry). Poimenujte ga, na primer, `GenAIJava`.
3. V projektu odprite **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Namestite **gpt-4o-mini** (ime namestitve `gpt-4o-mini`). Postopek ponovite za **text-embedding-3-small**, če želite primere embeddingov.
5. Na strani **Overview** kopirajte **končno točko** (na primer `https://<resource>.openai.azure.com/`).
6. Dodelite si dostop brez ključa: na viru odprite **Access control (IAM)** → **Add role assignment** → dodelite vlogo **Cognitive Services OpenAI User** svojemu računu.

> **Še vedno imate težave?** Oglejte si [dokumentacijo Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfiguracija okolja

**Če ste uporabili Možnost A (`azd up`)**, je vaša datoteka nastavitev že napisana — ničesar ni treba nastaviti. Preskočite na [Preizkus nastavitve](#preizkus-nastavitve).

**Če ste uporabili Možnost B (ročno)**, ustvarite `.env` datoteko za primer sami:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Uredite `.env` z vašo končno točko (brez ključa — avtentikacija je brezključna):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Varnostna opomba:** Ni treba shranjevati nobenega API ključa. Avtenticirate se z Microsoft Entra ID preko `az login` (lokalno) ali upravljanega identiteta (v Azure). Datoteka `.env` vsebuje le nastavitve brez skrivnosti in je že vključena v `.gitignore`.

## Preizkus nastavitve

Prepričajte se, da ste prijavljeni, da lahko brezključna avtentikacija pridobi žeton, nato zaženite primer:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # če še niste prijavljeni
mvn clean spring-boot:run
```

Videli boste odgovor modela `gpt-4o-mini`!

> **Uporabniki VS Code:** Pritisnite `F5` za zagon. Aplikacija samodejno naloži vašo `.env`.

> **Popoln primer:** Podrobnosti in odpravljanje težav si oglejte v [Basic Chat z Azure AI Foundry primeru](./examples/basic-chat-azure/README.md).

## Kaj sledi?

**Nastavitev je zaključena!** Zdaj imate:
- Azure AI Foundry z nameščenima `gpt-4o-mini` in `text-embedding-3-small`
- Avtentikacijo brez ključev (Microsoft Entra ID) — brez upravljanja ključev
- Lokalno `.env` z vašo končno točko in imeni namestitev
- Pripravljeno razvojno okolje za Java

**Nadaljujte** na [Poglavje 3: Osnovne tehnike generativne AI](../03-CoreGenerativeAITechniques/README.md) za začetek izdelave AI aplikacij!

## Viri

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Avtentikacija brez ključev z Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Dokumentacija Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Dokumentacija](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Dodatni viri

- [Prenesite VS Code](https://code.visualstudio.com/Download)
- [Pridobite Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Konfiguracija razvojnega kontejnerja](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Omejitev odgovornosti**:
Ta dokument je bil preveden z uporabo AI prevajalske storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku je treba obravnavati kot avtoritativni vir. Za kritične informacije je priporočljiv strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->