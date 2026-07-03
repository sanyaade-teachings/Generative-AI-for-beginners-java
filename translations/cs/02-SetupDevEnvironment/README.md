# Nastavení vývojového prostředí pro Generative AI pro Java

> **Rychlý start:** Zprovozněte své AI modely na **Azure AI Foundry** jako kód pomocí Bicep + `azd` během několika minut — podívejte se na [Průvodce nastavením Azure AI Foundry](getting-started-azure-openai.md). Autentizace je **bezklíčová** (Microsoft Entra ID), takže není potřeba spravovat žádné API klíče.

## Co se naučíte

- Nastavit vývojové prostředí pro AI aplikace v jazyce Java
- Vybrat a nakonfigurovat preferované vývojové prostředí (cloud-first s Codespaces, lokální dev container nebo plná lokální instalace)
- Otestovat nastavení připojením k modelu Azure AI Foundry

## Obsah

- [Co se naučíte](#co-se-naučíte)
- [Úvod](#úvod)
- [Krok 1: Nastavte si vývojové prostředí](#krok-1-nastavte-si-vývojové-prostředí)
  - [Možnost A: GitHub Codespaces (doporučeno)](#možnost-a-github-codespaces-doporučeno)
  - [Možnost B: Lokální dev container](#možnost-b-lokální-dev-container)
  - [Možnost C: Použijte stávající lokální instalaci](#možnost-c-použijte-stávající-lokální-instalaci)
- [Krok 2: Provision Azure AI Foundry](#krok-2-provision-azure-ai-foundry)
- [Krok 3: Otestujte své nastavení](#krok-3-otestujte-své-nastavení)
- [Řešení problémů](#řešení-problémů)
- [Shrnutí](#shrnutí)
- [Další kroky](#další-kroky)

## Úvod

Tato kapitola vás provede nastavením vývojového prostředí. Během tohoto kurzu budeme používat **Azure AI Foundry** pro modely. Modely zprovozníte jako kód pomocí Bicep a Azure Developer CLI (`azd`), a připojíte se s **bezklíčovou autentizací** (Microsoft Entra ID) — není potřeba kopírovat či uchovávat API klíče.

**Není potřeba žádné lokální nastavení!** Můžete použít GitHub Codespaces, který poskytuje plné vývojové prostředí přímo v prohlížeči a zde i zprovozníte Foundry.

Používáme **Azure AI Foundry**, protože:
- **Provisionuje se jako kód** — jedno `azd up` nasadí účet a deploymenty modelů
- **Je bezklíčové** — autentizace pomocí přihlášení do Azure nebo spravované identity
- **Je připravené pro produkci** — stejný kód běží lokálně i v Azure
- **Je flexibilní** — vyměníte model změnou názvu deploymentu, ne kódu

> **Poznámka**: Deploymenty Azure AI Foundry se účtují za tokeny (pay-as-you-go). Viz [průvodce nastavením Azure AI Foundry](getting-started-azure-openai.md) pro detaily o provisioningu, regionech a nákladech.

## Krok 1: Nastavte si vývojové prostředí

<a name="quick-start-cloud"></a>

Vytvořili jsme předkonfigurovaný vývojový kontejner, který minimalizuje nastavení a zajistí, že máte všechny potřebné nástroje pro tento kurz Generative AI pro Java. Vyberte si preferovaný přístup k vývoji:

### Možnosti nastavení prostředí:

#### Možnost A: GitHub Codespaces (Doporučeno)

**Začněte kódovat za 2 minuty – bez lokální instalace!**

1. Forkněte si tento repozitář na svůj GitHub účet  
   > **Poznámka**: Pokud chcete upravit základní konfiguraci, podívejte se na [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Klikněte na **Code** → záložka **Codespaces** → **...** → **New with options...**
3. Použijte výchozí nastavení – vybere se **Dev container configuration**: **Generative AI Java Development Environment** vlastní devcontainer vytvořený pro tento kurz
4. Klikněte na **Create codespace**
5. Počkejte asi 2 minuty, než se prostředí připraví
6. Pokračujte na [Krok 2: Provision Azure AI Foundry](#krok-2-provision-azure-ai-foundry)

<img src="../../../translated_images/cs/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/cs/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/cs/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **Výhody Codespaces**:
> - Není potřeba žádná lokální instalace
> - Funguje na libovolném zařízení s prohlížečem
> - Předkonfigurováno se všemi nástroji a závislostmi
> - Zdarma 60 hodin/měsíc pro osobní účty
> - Konzistentní prostředí pro všechny studenty

#### Možnost B: Lokální Dev Container

**Pro vývojáře, kteří preferují lokální vývoj s Dockerem**

1. Forkněte a naklonujte tento repozitář do svého počítače  
   > **Poznámka**: Pokud chcete upravit základní konfiguraci, podívejte se na [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Nainstalujte [Docker Desktop](https://www.docker.com/products/docker-desktop/) a [VS Code](https://code.visualstudio.com/)
3. Nainstalujte rozšíření [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) do VS Code
4. Otevřete složku s repozitářem ve VS Code
5. Jakmile se zobrazí výzva, klikněte na **Reopen in Container** (nebo použijte `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Počkejte, až se container sestaví a spustí
7. Pokračujte na [Krok 2: Provision Azure AI Foundry](#krok-2-provision-azure-ai-foundry)

<img src="../../../translated_images/cs/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/cs/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Možnost C: Použijte stávající lokální instalaci

**Pro vývojáře s již existujícím Java prostředím**

Požadavky:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) nebo preferované IDE

Postup:
1. Naklonujte tento repozitář do svého počítače
2. Otevřete projekt ve svém IDE
3. Pokračujte na [Krok 2: Provision Azure AI Foundry](#krok-2-provision-azure-ai-foundry)

> **Tip**: Pokud máte zařízení s nízkým výkonem, ale chcete používat VS Code lokálně, využijte GitHub Codespaces! Můžete připojit svůj lokální VS Code ke cloudovému Codespace a mít tak výhody obou světů.

<img src="../../../translated_images/cs/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## Krok 2: Provision Azure AI Foundry

Nasadíte AI modely z kurzu do Azure AI Foundry jako kód. Ze složky kořene repozitáře spusťte:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` vás vyzve k zadání názvu prostředí a regionu, provisionuje účet Azure AI Foundry s deploymenty `gpt-4o-mini` a `text-embedding-3-small`, a zapíše endpoint do souboru `.env` příkladu — vše s **bezklíčovou** autentizací (bez API klíčů).

> **Plný průvodce:** Viz [Průvodce nastavením Azure AI Foundry](getting-started-azure-openai.md) pro předpoklady, manuální (portal) alternativu, doporučení regionu a informace o nákladech a úklidu.

## Krok 3: Otestujte své nastavení

Po zprovoznění Foundry modelů otestujte připojení pomocí příkladové aplikace v [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Otevřete terminál ve vývojovém prostředí.
2. Přejděte do příkladu:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Ujistěte se, že jste přihlášeni (bezklíčová autentizace vyžaduje token):  
   ```bash
   az login
   ```
  
   > Pokud jste spustili `azd up`, `.env` se endpointem už byl vytvořen automaticky.
4. Spusťte aplikaci:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Měli byste vidět odpověď z modelu `gpt-4o-mini`.

### Pochopení příkladového kódu

Příklad v `examples/basic-chat-azure` je aplikace Spring Boot využívající **Spring AI** k připojení k Azure AI Foundry s bezklíčovou autentizací.

**Co tento kód dělá:**
- **Připojuje se** k Azure AI Foundry pomocí přihlášení do Azure (Microsoft Entra ID) — bez API klíče
- **Odesílá** prompt modelu `gpt-4o-mini`
- **Přijímá** a zobrazuje odpověď AI
- **Ověřuje**, že je nastavení správné a funguje

**Klíčová závislost** (v `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfigurace** (`application.yml`):  
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


## Shrnutí

Skvělé! Nyní máte vše nastaveno:

- Zprovozněné modely Azure AI Foundry jako kód pomocí Bicep + `azd`
- Běží vaše Java vývojové prostředí (Codespaces, dev container, nebo lokální)
- Připojení k Azure AI Foundry s bezklíčovou autentizací (Microsoft Entra ID) — žádné API klíče
- Otestováno vše funkční jednoduchým příkladem komunikujícím s modelem

## Další kroky

[Kapitola 3: Základní techniky GenAI](../03-CoreGenerativeAITechniques/README.md)

## Řešení problémů

Máte problémy? Zde jsou běžné potíže a řešení:

- **Selhání autentizace (401/403)?**  
  - Spusťte `az login` — autentizace je bezklíčová, musíte být přihlášeni  
  - Zkontrolujte, že máte roli **Cognitive Services OpenAI User** na daném zdroji  
  - Pokud jste právě provedli provisioning, počkejte chvíli, než se role projeví

- **Maven nenalezen?**  
  - Pokud používáte dev container/Codespaces, Maven by měl být předinstalovaný  
  - U lokálního nastavení se ujistěte, že máte Java 21+ a Maven 3.9+  
  - Zkuste příkaz `mvn --version` pro ověření instalace

- **`azd` nenalezen nebo provisioning selhává?**  
  - Nainstalujte [Azure Developer CLI](https://aka.ms/azure-dev/install) a spusťte `azd auth login`  
  - Vyberte region, kde je `gpt-4o-mini` dostupný (např. `eastus2`)  
  - Viz [průvodce nastavením Azure AI Foundry](getting-started-azure-openai.md) pro podrobnosti

- **Dev container se nespouští?**  
  - Ověřte, že Docker Desktop běží (pro lokální vývoj)  
  - Zkuste container přebuildovat: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Chyby při kompilaci aplikace?**  
  - Ujistěte se, že jste ve správném adresáři: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Zkuste `mvn clean compile` pro vyčištění a znovukompilování

> **Potřebujete pomoc?**: Stále máte problémy? Otevřete issue v repozitáři a rádi vám pomůžeme.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->