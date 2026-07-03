# Arenduskeskkonna seadistamine Azure AI Foundry jaoks

> See juhend seab selle kursuse Java AI rakenduste jaoks üles **Azure AI Foundry** mudelid, kasutades **võtmeta** autentimist (Microsoft Entra ID) — pole vaja API võtmeid hallata. Oled tööriistadega uus? Alusta [arenduskeskkonna juhendist](./README.md).

See juhend seab selle kursuse Java AI rakenduste jaoks üles **Azure AI Foundry** mudelid. Sul on kaks võimalust:

- **Variant A — Provisioneeri `azd` + Bicep abil (soovitatav):** üks käsklus viib läbi Foundry konto ja mudelite saidi loomise koodi abil. Ei mingit portaalis klõpsimist.
- **Variant B — Loo ressursid käsitsi** Azure AI Foundry portaali kaudu.

Mõlemad teed kasutavad **võtmeta autentimist** (Microsoft Entra ID) — pole API võtmeid, mida kopeerida ega lekkida.

## Sisukord

- [Mida luuakse](#mida-luuakse)
- [Eeltingimused](#eeltingimused)
- [Variant A: Provisioneeri azd + Bicep abil (Soovitatav)](#option-a-provision-with-azd--bicep-recommended)
- [Variant B: Loo ressursid käsitsi](#variant-b-loo-ressursid-käsitsi)
- [Seadista oma keskkond](#seadista-oma-keskkond)
- [Testi oma seadistust](#testi-oma-seadistust)
- [Mis järgmiseks?](#mis-järgmiseks)
- [Ressursid](#ressursid)
- [Lisamaterjalid](#lisamaterjalid)

## Mida luuakse

[`infra/`](../../../02-SetupDevEnvironment/infra) Bicep mallid loovad:

- **Azure AI Foundry** konto (`Microsoft.CognitiveServices/accounts`, tüüp `AIServices`) koos projektiga
- **vestlus** juurutuse — `gpt-4o-mini`
- **embedimise** juurutuse — `text-embedding-3-small` (kasutatakse hilisemates peatükkides)
- **võtmeta rolli määramise** (`Cognitive Services OpenAI User`), mis võimaldab sisselogimist `az login` kaudu võtmete haldamise asemel

## Eeltingimused

- [Azure tellimus](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) ja [Maven 3.9+](https://maven.apache.org/download.cgi)

## Variant A: Provisioneeri azd + Bicep abil (Soovitatav)

Kaustast `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Logi sisse (mõlemad tööriistad)
azd auth login
az login

# Hangi Foundry konto ja mudelite juurutused
azd up
```

`azd` küsib **keskkonna nime** (näiteks `genai-java`) ja **regiooni**. Vali piirkond, kus on saadaval `gpt-4o-mini` ja `text-embedding-3-small` — näiteks `eastus2` või `swedencentral`.

Kui provisioneerimine lõppeb, teeb azd:

1. Juurutab kõik, mis on defineeritud [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) failis.
2. Käivitab postprovisioneerimise hook’i, mis kirjutab faili [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) koos sinu lõpp-punkti ja juurutuste nimedega (ilma saladusteta).

> **Nipp:** Käivita `azd up` uuesti iga kord muudatuste rakendamiseks. Kustuta kõik ja lõpeta kulutamine käsuga `azd down`.

Sätteid nägemiseks:

```bash
azd env get-values
```

Nüüd liigu edasi [testi oma seadistust](#testi-oma-seadistust).

## Variant B: Loo ressursid käsitsi

Eelistad portaali? Loo ressursid käsitsi:

1. Mine [Azure AI Foundry portaali](https://ai.azure.com/) ja logi sisse.
2. **Loo projekt** (see loob ka AI Foundry ressursi). Anna sellele nimi, näiteks `GenAIJava`.
3. Projektis ava **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Juuruta **gpt-4o-mini** (juurutuse nimi `gpt-4o-mini`). Korda protseduuri **text-embedding-3-small** jaoks, kui soovid embedimise näiteid.
5. Ava **Overview**, kopeeri **lõpp-punkt** (näiteks `https://<resource>.openai.azure.com/`).
6. Anna endale võtmeta ligipääs: ava ressursil **Juurdepääsu juhtimine (IAM)** → **Lisa rolli määramine** → lisa roll **Cognitive Services OpenAI User** oma kontole.

> **Probleemid jätkuvad?** Vaata [Azure AI Foundry dokumentatsiooni](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Seadista oma keskkond

**Kui kasutasid varianti A (`azd up`)**, on sinu seaded juba olemas — pole midagi lisaks seadistada. Liigu edasi [testi oma seadistust](#testi-oma-seadistust).

**Kui kasutasid varianti B (käsitsi)**, loo näite `.env` fail ise:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Redigeeri `.env` faili oma lõpp-punktiga (ilma võtmeta — autentimine on võtmeta):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Turvalisuse märkus:** API võtit pole vaja salvestada. Autentid Microsoft Entra ID abil kas `az login` kaudu (kohapeal) või hallatud identiteediga (Azure’is). `.env` fail sisaldab ainult mitte-saladuslikke sätteid ja on juba `.gitignore` failiga kaetud.

## Testi oma seadistust

Veendu, et oled sisse logitud, et võtmeta autentimine saaks tokeni ja käivita näide:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # kui te pole veel sisse logitud
mvn clean spring-boot:run
```

Peaksid nägema vastust mudelilt `gpt-4o-mini`!

> **VS Code kasutajatele:** Vajuta `F5`, et käivitada. Rakendus laadib sinu `.env` automaatselt.

> **Täielik näide:** Vaata [Basic Chat with Azure AI Foundry näidet](./examples/basic-chat-azure/README.md) detailide ja veaotsingu jaoks.

## Mis järgmiseks?

**Seadistamine on lõpetatud!** Sul on nüüd:

- Azure AI Foundry koos `gpt-4o-mini` ja `text-embedding-3-small` mudelitega juurutatud
- Võtmeta autentimine (Microsoft Entra ID) — pole võtmete haldamist
- Kohalik `.env` sinu lõpp-punktide ja juurutuste nimedega
- Java arenduskeskkond valmis kasutamiseks

**Jätka** peatüki [3 juurde: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md), et alustada AI rakenduste loomist!

## Ressursid

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Võtmeta autentimine Microsoft Entra ID-ga](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry dokumentatsioon](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI dokumentatsioon](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Lisamaterjalid

- [Laadi alla VS Code](https://code.visualstudio.com/Download)
- [Hangi Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev Container konfiguratsioon](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Lahtiütlus**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, palun pange tähele, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlkega seotud eksimustest või valesti mõistmistest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->