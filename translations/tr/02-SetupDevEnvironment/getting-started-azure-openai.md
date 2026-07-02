# Azure AI Foundry için Geliştirme Ortamının Kurulumu

> Bu kılavuz, bu dersteki Java AI uygulamaları için **Azure AI Foundry** modellerini **anahtarsız** kimlik doğrulama (Microsoft Entra ID) kullanarak kurar — yönetilecek API anahtarları yoktur. Araçlara yeni misiniz? [geliştirme ortamı kılavuzuyla](./README.md) başlayın.

Bu kılavuz, bu dersteki Java AI uygulamaları için **Azure AI Foundry** modellerini kurar. İki yolunuz var:

- **Seçenek A — `azd` + Bicep ile sağlama (önerilen):** bir komutla Foundry hesabı ve modeller kod olarak dağıtılır. Portalda tıklama yok.
- **Seçenek B — Kaynakları elle oluşturun** Azure AI Foundry portalında.

Her iki yol da **anahtarsız kimlik doğrulama** (Microsoft Entra ID) kullanır — kopyalanacak veya sızdırılacak API anahtarı yoktur.

## İçindekiler

- [Oluşturulanlar](#oluşturulanlar)
- [Önkoşullar](#önkoşullar)
- [Seçenek A: azd + Bicep ile Sağlama (Önerilen)](#option-a-provision-with-azd--bicep-recommended)
- [Seçenek B: Kaynakları Elle Oluşturma](#seçenek-b-kaynakları-elle-oluşturma)
- [Ortamınızı Yapılandırma](#ortamınızı-yapılandırma)
- [Kurulumunuzu Test Etme](#kurulumunuzu-test-etme)
- [Sonraki Adımlar?](#sonraki-adımlar)
- [Kaynaklar](#kaynaklar)
- [Ek Kaynaklar](#ek-kaynaklar)

## Oluşturulanlar

[`infra/`](../../../02-SetupDevEnvironment/infra) içindeki Bicep şablonları şunları sağlar:

- Bir **Azure AI Foundry** hesabı (`Microsoft.CognitiveServices/accounts`, tür `AIServices`) ve bir proje
- Bir **sohbet** dağıtımı — `gpt-4o-mini`
- Bir **embedding** dağıtımı — `text-embedding-3-small` (ileriki bölümlerde kullanılır)
- Anahtarsız rol ataması (`Cognitive Services OpenAI User`), böylece anahtar yönetmek yerine `az login` ile oturum açabilirsiniz

## Önkoşullar

- Bir [Azure aboneliği](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) ve [Maven 3.9+](https://maven.apache.org/download.cgi)

## Seçenek A: azd + Bicep ile Sağlama (Önerilen)

`02-SetupDevEnvironment` klasöründen:

```bash
cd 02-SetupDevEnvironment

# Giriş yap (her iki araç)
azd auth login
az login

# Foundry hesabını ve model dağıtımlarını sağla
azd up
```
  
`azd` sizden bir **ortam adı** (örneğin `genai-java`) ve bir **bölge** ister. `gpt-4o-mini` ve `text-embedding-3-small` modelinin mevcut olduğu bir bölge seçin — örneğin `eastus2` veya `swedencentral`.

Sağlama tamamlandığında azd:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) içinde tanımlanan her şeyi dağıtır.
2. Sonrasında, [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) dosyasını uç nokta ve dağıtım adlarıyla (gizli bilgi yok) yazar.

> **İpucu:** Değişiklik uygulamak için istediğiniz zaman `azd up` komutunu tekrar çalıştırın. Her şeyi silmek ve maliyetten kaçınmak için `azd down` komutunu kullanın.

Oluşturulan ayarları görmek için:

```bash
azd env get-values
```
  
Şimdi [Kurulumunuzu Test Etme](#kurulumunuzu-test-etme) bölümüne geçin.

## Seçenek B: Kaynakları Elle Oluşturma

Portali tercih ediyorsanız, kaynakları el ile oluşturun:

1. [Azure AI Foundry portalına](https://ai.azure.com/) gidin ve oturum açın.
2. **Bir proje oluşturun** (bu ayrıca bir AI Foundry kaynağı da oluşturur). `GenAIJava` gibi bir isim verin.
3. Projenizde **Modeller + uç noktalar** → **Model dağıt** → **Temel modeli dağıt** bölümünü açın.
4. **gpt-4o-mini**’yi dağıtın (dağıtım adı `gpt-4o-mini`). Embedding örneklerini istiyorsanız **text-embedding-3-small** için de tekrarlayın.
5. **Genel Bakış** sayfasından **uç noktayı** kopyalayın (örneğin `https://<resource>.openai.azure.com/`).
6. Anahtarsız erişim izinleri verin: kaynakta **Erişim kontrolü (IAM)** → **Rol ataması ekle** → **Cognitive Services OpenAI User** rolünü hesabınıza atayın.

> **Hâlâ sorun mu yaşıyorsunuz?** [Azure AI Foundry dokümantasyonuna](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) bakın.

## Ortamınızı Yapılandırma

**Seçenek A’yı (`azd up`) kullandıysanız**, ayarlar dosyanız zaten yazılmıştır — yapılandıracak bir şey yoktur. [Kurulumunuzu Test Etme](#kurulumunuzu-test-etme) bölümüne geçin.

**Seçenek B’yi (elle) kullandıysanız**, örneğin `.env` dosyasını kendiniz oluşturun:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
`.env` dosyasını uç noktanızla düzenleyin (anahtar yok — kimlik doğrulama anahtarsızdır):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **Güvenlik notu:** Saklanacak bir API anahtarı yoktur. Yerelde `az login` ile Microsoft Entra ID üzerinden veya Azure’da yönetilen kimlikle kimlik doğrulaması yaparsınız. `.env` dosyası sadece gizli olmayan ayarları içerir ve zaten `.gitignore` tarafından korunmaktadır.

## Kurulumunuzu Test Etme

Anahtarsız kimlik doğrulamanın bir jeton alabilmesi için oturum açtığınızdan emin olun, sonra örneği çalıştırın:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # eğer zaten giriş yapmadıysanız
mvn clean spring-boot:run
```
  
`gpt-4o-mini` modelinden yanıt almalısınız!

> **VS Code kullanıcıları:** Çalıştırmak için `F5` tuşuna basın. Uygulama `.env` dosyanızı otomatik yükler.

> **Tam örnek:** Ayrıntılar ve sorun giderme için [Azure AI Foundry ile Temel Sohbet örneğine](./examples/basic-chat-azure/README.md) bakın.

## Sonraki Adımlar?

**Kurulum tamamlandı!** Artık şunlara sahipsiniz:  
- `gpt-4o-mini` ve `text-embedding-3-small` ile Azure AI Foundry dağıtıldı  
- Anahtarsız kimlik doğrulama (Microsoft Entra ID) — yönetilecek anahtar yok  
- Uç nokta ve dağıtım adlarını içeren yerel `.env` dosyası  
- Hazır bir Java geliştirme ortamı

**Devam edin:** [Bölüm 3: Temel Üretici AI Teknikleri](../03-CoreGenerativeAITechniques/README.md) ile AI uygulamaları geliştirmeye başlayın!

## Kaynaklar

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID ile anahtarsız kimlik doğrulama](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Dokümantasyonu](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Dokümantasyonu](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Ek Kaynaklar

- [VS Code indir](https://code.visualstudio.com/Download)
- [Docker Desktop edinin](https://www.docker.com/products/docker-desktop)
- [Dev Container Konfigürasyonu](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba sarf etsek de, otomatik çevirilerin hata veya yanlışlık içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek yanlış anlamalardan veya yanlış yorumlamalardan sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->