# Azure AI Foundry ile Temel Sohbet - Uçtan Uca Örnek

Bu örnek, **Azure AI Foundry** modeline **anahtarsız kimlik doğrulama** (Microsoft Entra ID) kullanarak bağlanan ve kurulumu test eden basit bir Spring Boot uygulamasıdır. Spring AI'nin `ChatClient`'ını kullanır.

## İçindekiler

- [Ön Koşullar](#ön-koşullar)
- [Hızlı Başlangıç](#hızlı-başlangıç)
- [Kimlik Doğrulama Nasıl Çalışır](#kimlik-doğrulama-nasıl-çalışır)
- [Uygulamayı Çalıştırma](#uygulamayı-çalıştırma)
  - [Maven ile](#maven-ile)
  - [VS Code ile](#vs-code-ile)
  - [Beklenen Çıktı](#beklenen-çıktı)
- [Yapılandırma Referansı](#yapılandırma-referansı)
  - [Ortam Değişkenleri](#ortam-değişkenleri)
  - [Spring Yapılandırması](#spring-yapılandırması)
- [Sorun Giderme](#sorun-giderme)
  - [Yaygın Sorunlar](#yaygın-sorunlar)
  - [Hata Ayıklama Modu](#hata-ayıklama-modu)
- [Sonraki Adımlar](#sonraki-adımlar)
- [Kaynaklar](#kaynaklar)

## Ön Koşullar

Bu örneği çalıştırmadan önce şunlara sahip olduğunuzdan emin olun:

- `gpt-4o-mini` dağıtımı bulunan bir Azure AI Foundry kaynağı — `azd up` ile veya manuel olarak [Azure AI Foundry kurulum kılavuzu](../../getting-started-azure-openai.md) üzerinden temin edin
- O kaynakta **Cognitive Services OpenAI User** rolü (Bicep şablonları bunu sizin için atar)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ile oturum açılmış
- Java 21+ ve Maven 3.9+

> **API anahtarı gerekmez** — kimlik doğrulama Microsoft Entra ID üzerinden anahtarsızdır.

## Hızlı Başlangıç

```bash
# 1. Projeye gidin
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Anahtarsız kimlik doğrulaması bir token alabilmesi için giriş yapın
az login

# 3. Uç noktayı yapılandırın
#    - Eğer `azd up` komutunu çalıştırdıysanız, .env dosyası sizin için yazıldı (bunu atlayın).
#    - Aksi takdirde şablonu kopyalayın ve AZURE_OPENAI_ENDPOINT ayarlayın:
cp .env.example .env

# 4. Uygulamayı çalıştırın
mvn spring-boot:run
```

## Kimlik Doğrulama Nasıl Çalışır

Bu örnek, **Microsoft Entra ID** ile kimlik doğrulaması yapar — API anahtarı yoktur.

Sadece `spring.ai.azure.openai.endpoint` ayarlandığında (api-key olmadan) Spring AI, Azure OpenAI istemcisini [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) ile oluşturur. Bu kimlik bilgisi otomatik olarak yerelde `az login` oturumunuzdan veya Azure'da çalışan yönetilen kimlikten bir belirteç bulur — böylece aynı kod her iki yerde de değişiklik yapmadan çalışır.

## Uygulamayı Çalıştırma

### Maven ile

```bash
mvn spring-boot:run
```

### VS Code ile

1. Projeyi VS Code'da açın
2. `F5` tuşuna basın veya "Çalıştır ve Hata Ayıkla" panelini kullanın
3. "Spring Boot-BasicChatApplication" yapılandırmasını seçin

> **Not**: VS Code yapılandırması .env dosyanızı otomatik yükler

### Beklenen Çıktı

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

## Yapılandırma Referansı

### Ortam Değişkenleri

| Değişken | Açıklama | Gerekli | Örnek |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) uç nokta URL'si | Evet | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Sohbet modeli dağıtım adı | Hayır | `gpt-4o-mini` (varsayılan) |

> API anahtarı değişkeni **yoktur** — kimlik doğrulama anahtarsızdır (Microsoft Entra ID üzerinden `az login`).

### Spring Yapılandırması

`application.yml` dosyası yapılandırır:
- **Uç Nokta**: `${AZURE_OPENAI_ENDPOINT}` - Ortam değişkeninden
- **Dağıtım**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Ortam değişkeninden, yedekli
- **Kimlik Doğrulama**: Anahtarsız — `api-key` ayarlanmadı, bu yüzden Spring AI `DefaultAzureCredential` kullanır
- **Sıcaklık**: `0.7` - Yaratıcılığı kontrol eder (0.0 = belirleyici, 1.0 = yaratıcı)
- **Maksimum Token**: `500` - Maksimum yanıt uzunluğu

## Sorun Giderme

### Yaygın Sorunlar

<details>
<summary><strong>Hata: 401 / "PermissionDenied" / belirteç hataları</strong></summary>

- `az login` komutunu çalıştırın — anahtarsız kimlik doğrulama bir belirteç almak için aktif oturum gerektirir
- Hesabınızda kaynağa yönelik **Cognitive Services OpenAI User** rolü olduğundan emin olun
- Rolü yeni atadıysanız, yayılması için birkaç dakika bekleyin
- Doğru kiracı/abone altında olduğunuzu doğrulayın (`az account show`)
</details>

<details>
<summary><strong>Hata: "Uç nokta geçerli değil" / bağlantı hataları</strong></summary>

- `AZURE_OPENAI_ENDPOINT` değişkeninin tam temel URL (örneğin `https://your-resource.openai.azure.com/`) olduğundan emin olun
- Sonundaki eğik çizgi tutarlılığını kontrol edin
- Uç noktanın sağladığınız kaynakla eşleştiğini doğrulayın (`azd env get-values`)
</details>

<details>
<summary><strong>Hata: "Dağıtım bulunamadı"</strong></summary>

- `AZURE_OPENAI_DEPLOYMENT` değişkeninin Azure’daki geçerli bir dağıtım adıyla eşleştiğini kontrol edin
- Modelin başarıyla dağıtıldığını ve aktif olduğunu doğrulayın
- Varsayılan dağıtım adı `gpt-4o-mini`’dir
</details>

<details>
<summary><strong>VS Code: Ortam değişkenleri yüklenmiyor</strong></summary>

- `.env` dosyanızın proje kök dizininde (pom.xml ile aynı seviyede) olduğundan emin olun
- VS Code entegrasyon terminalinde `mvn spring-boot:run` komutunu deneyin
- VS Code Java eklentisinin doğru yüklendiğini kontrol edin
</details>

### Hata Ayıklama Modu

Ayrıntılı günlükleme için `application.yml` içindeki bu satırların yorumunu kaldırın:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Sonraki Adımlar

**Kurulum Tamamlandı!** Öğrenme yolculuğunuza devam edin:

[3. Bölüm: Temel Üretken AI Teknikleri](../../../03-CoreGenerativeAITechniques/README.md)

## Kaynaklar

- [Spring AI Azure OpenAI Dokümantasyonu](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID ile Anahtarsız Kimlik Doğrulama](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portalı](https://ai.azure.com/)
- [Azure AI Foundry Dokümantasyonu](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba sarf etsek de, otomatik çevirilerin hata veya yanlışlık içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek yanlış anlamalardan veya yanlış yorumlamalardan sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->