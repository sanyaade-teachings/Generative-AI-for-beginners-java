# Osnovni pogovor z Azure AI Foundry – primer od začetka do konca

Ta primer je preprosta aplikacija Spring Boot, ki se poveže z modelom **Azure AI Foundry** z uporabo **avtentikacije brez ključa** (Microsoft Entra ID) in preveri vašo nastavitev. Uporablja `ChatClient` Spring AI.

## Kazalo

- [Pogoji](#pogoji)
- [Hiter začetek](#hiter-začetek)
- [Kako deluje avtentikacija](#kako-deluje-avtentikacija)
- [Zagon aplikacije](#zagon-aplikacije)
  - [Uporaba Mavena](#uporaba-mavena)
  - [Uporaba VS Code](#uporaba-vs-code)
  - [Pričakovani izhod](#pričakovani-izhod)
- [Referenca konfiguracije](#referenca-konfiguracije)
  - [Sistemske spremenljivke](#sistemske-spremenljivke)
  - [Spring konfiguracija](#spring-konfiguracija)
- [Reševanje težav](#reševanje-težav)
  - [Pogoste težave](#pogoste-težave)
  - [Način odpravljanja napak](#način-odpravljanja-napak)
- [Nadaljnji koraki](#nadaljnji-koraki)
- [Viri](#viri)

## Pogoji

Pred zagonom tega primera poskrbite, da imate:

- Vir Azure AI Foundry z nameščeno različico `gpt-4o-mini` — namestite jo z `azd up` ali ročno preko [vodnika za nastavitev Azure AI Foundry](../../getting-started-azure-openai.md)
- Vlogo **Cognitive Services OpenAI User** za ta vir (Bicep predloge vam to nastavijo)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), prijavljeni z `az login`
- Java 21+ in Maven 3.9+

> **Ni potreben API ključ** — avtentikacija poteka brez ključa preko Microsoft Entra ID.

## Hiter začetek

```bash
# 1. Pojdi do projekta
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Prijavi se, da lahko brezključna avtentikacija pridobi žeton
az login

# 3. Konfiguriraj končno točko
#    - Če si zagnal `azd up`, je bila .env datoteka ustvarjena (to preskoči).
#    - V nasprotnem primeru kopiraj predlogo in nastavi AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Zaženi aplikacijo
mvn spring-boot:run
```

## Kako deluje avtentikacija

Ta primer se avtorizira z **Microsoft Entra ID** — ni API ključa.

Ko je nastavljen samo `spring.ai.azure.openai.endpoint` (in ni api-ključa), Spring AI ustvari Azure OpenAI odjemalca z [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Ta poverilnica samodejno najde žeton iz vaše lokalne seje `az login` ali iz upravljane identitete, ko tečete v Azure — zato ista koda deluje na obeh mestih brez sprememb.

## Zagon aplikacije

### Uporaba Mavena

```bash
mvn spring-boot:run
```

### Uporaba VS Code

1. Odprite projekt v VS Code
2. Pritisnite `F5` ali uporabite ploščo "Run and Debug"
3. Izberite konfiguracijo "Spring Boot-BasicChatApplication"

> **Opomba**: Konfiguracija VS Code samodejno naloži vašo datoteko .env

### Pričakovani izhod

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

### Sistemske spremenljivke

| Spremenljivka           | Opis                                   | Zahtevano | Primer                              |
|-------------------------|---------------------------------------|-----------|-----------------------------------|
| `AZURE_OPENAI_ENDPOINT` | URL končne točke Foundry (Azure OpenAI) | Da        | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Ime nameščene različice modela za chat | Ne        | `gpt-4o-mini` (privzeto)          |

> Ni **nobene** spremenljivke API ključa — avtentikacija je brez ključa (Microsoft Entra ID preko `az login`).

### Spring konfiguracija

Datoteka `application.yml` konfigurira:
- **Končna točka**: `${AZURE_OPENAI_ENDPOINT}` - Iz sistemske spremenljivke
- **Nameščena različica**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Iz sistemske spremenljivke z rezervno vrednostjo
- **Avtentikacija**: brez ključa — ni nastavljen `api-key`, zato Spring AI uporablja `DefaultAzureCredential`
- **Temperatura**: `0.7` - Nadzoruje ustvarjalnost (0.0 = determinističen, 1.0 = ustvarjalen)
- **Maksimalno število tokenov**: `500` - Najdaljša dovoljena dolžina odgovora

## Reševanje težav

### Pogoste težave

<details>
<summary><strong>Napaka: 401 / "PermissionDenied" / napake s tokenom</strong></summary>

- Zaženite `az login` — avtentikacija brez ključa potrebuje aktivno prijavo, da dobi žeton
- Preverite, ali ima vaš račun vlogo **Cognitive Services OpenAI User** za vir
- Če ste pravkar dodelili vlogo, počakajte minuto, da začne delovati
- Preverite, ali ste v pravem najemniku/naročnini (`az account show`)
</details>

<details>
<summary><strong>Napaka: "Končna točka ni veljavna" / napake povezave</strong></summary>

- Prepričajte se, da je `AZURE_OPENAI_ENDPOINT` popolni osnovni URL (npr. `https://your-resource.openai.azure.com/`)
- Preverite doslednost končne poševnice
- Potrdite, da končna točka ustreza vašemu nameščenemu viru (`azd env get-values`)
</details>

<details>
<summary><strong>Napaka: "Nameščenec ni bil najden"</strong></summary>

- Preverite, da `AZURE_OPENAI_DEPLOYMENT` ustreza imenu nameščenca v Azure
- Preverite, da je model uspešno nameščen in aktiven
- Privzeto ime nameščenca je `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Sistemske spremenljivke se ne nalagajo</strong></summary>

- Prepričajte se, da je vaša datoteka `.env` v korenskem imeniku projekta (na isti ravni kot `pom.xml`)
- Poskusite zagnati `mvn spring-boot:run` v vgrajenem terminalu VS Code
- Preverite, da je razširitev Java za VS Code pravilno nameščena
</details>

### Način odpravljanja napak

Za omogočanje podrobnega beleženja odkomentirajte te vrstice v `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Nadaljnji koraki

**Namestitev je končana!** Nadaljujte z učenjem:

[3. poglavje: Osnovne tehnike generativne umetne inteligence](../../../03-CoreGenerativeAITechniques/README.md)

## Viri

- [Spring AI Azure OpenAI dokumentacija](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Avtentikacija brez ključa z Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal Azure AI Foundry](https://ai.azure.com/)
- [Dokumentacija Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Omejitev odgovornosti**:
Ta dokument je bil preveden z uporabo AI prevajalske storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku je treba obravnavati kot avtoritativni vir. Za kritične informacije je priporočljiv strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->