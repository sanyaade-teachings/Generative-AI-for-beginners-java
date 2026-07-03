# Java için Üretken Yapay Zeka Geliştirme Ortamını Kurma

> **Hızlı Başlangıç:** Yapay zeka modellerinizi **Azure AI Foundry** üzerinde Bicep + `azd` koduyla birkaç dakika içinde hazırlayın — bkz. [Azure AI Foundry Kurulum Kılavuzu](getting-started-azure-openai.md). Kimlik doğrulama **anahtarsızdır** (Microsoft Entra ID), böylece yönetilecek API anahtarı yoktur.

## Neler Öğreneceksiniz

- AI uygulamaları için bir Java geliştirme ortamı kurma  
- Tercih ettiğiniz geliştirme ortamını seçme ve yapılandırma (öncelikli bulut ile Codespaces, yerel geliştirme konteyneri veya tam yerel kurulum)  
- Kurulumunuzu Azure AI Foundry modeline bağlanarak test etme  

## İçindekiler

- [Neler Öğreneceksiniz](#neler-öğreneceksiniz)  
- [Giriş](#giriş)  
- [Adım 1: Geliştirme Ortamınızı Kurun](#adım-1-geliştirme-ortamınızı-kurun)  
  - [Seçenek A: GitHub Codespaces (Önerilen)](#seçenek-a-github-codespaces-önerilen)  
  - [Seçenek B: Yerel Geliştirme Konteyneri](#seçenek-b-yerel-geliştirme-konteyneri)  
  - [Seçenek C: Varolan Yerel Kurulumunuzu Kullanın](#seçenek-c-varolan-yerel-kurulumunuzu-kullanın)  
- [Adım 2: Azure AI Foundry Sağlama](#adım-2-azure-ai-foundry-sağlama)  
- [Adım 3: Kurulumunuzu Test Edin](#adım-3-kurulumunuzu-test-edin)  
- [Sorun Giderme](#sorun-giderme)  
- [Özet](#özet)  
- [Sonraki Adımlar](#sonraki-adımlar)  

## Giriş

Bu bölüm, geliştirme ortamınızın kurulmasında size rehberlik edecektir. Bu kurs boyunca modeller için **Azure AI Foundry** kullanacağız. Modelleri Bicep ve Azure Developer CLI (`azd`) ile kod olarak sağlarsınız, ardından **anahtarsız kimlik doğrulama** (Microsoft Entra ID) ile bağlanırsınız — kopyalanacak veya sızdırılacak API anahtarı yoktur.

**Yerel kurulum gerekmez!** Tam gelişmiş bir ortam sunan GitHub Codespaces'i tarayıcınızda kullanabilir ve Foundry'i oradan sağlayabilirsiniz.

Bu kurs için **Azure AI Foundry** kullanıyoruz çünkü:  
- **Kod olarak sağlanır** — tek `azd up` komutu ile hesap ve model dağıtımları yapılır  
- **Anahtarsızdır** — Azure oturum açmanız veya yönetilen kimlik ile kimlik doğrulaması  
- **Üretim hazırdır** — aynı kod yerelde ve Azure'da çalışır  
- **Esnektir** — model değiştirirken sadece dağıtım adını değiştirirsiniz, kodu değil  

> **Not**: Azure AI Foundry dağıtımları token başına ücretlendirilir (kullandıkça öde). Sağlama, bölge ve maliyet detayları için [Azure AI Foundry kurulum kılavuzuna](getting-started-azure-openai.md) bakınız.

## Adım 1: Geliştirme Ortamınızı Kurun

<a name="quick-start-cloud"></a>

Bu Generative AI for Java kursu için gereken tüm araçların olduğu önceden yapılandırılmış bir geliştirme konteyneri oluşturduk. Tercih ettiğiniz geliştirme yöntemini seçin:

### Ortam Kurulum Seçenekleri:

#### Seçenek A: GitHub Codespaces (Önerilen)

**2 dakika içinde kodlamaya başlayın - yerel kurulum gerekmez!**

1. Bu depoyu GitHub hesabınıza fork edin  
   > **Not**: Temel yapılandırmayı düzenlemek isterseniz [Dev Container Configuration](../../../.devcontainer/devcontainer.json) dosyasına göz atın  
2. **Code** → **Codespaces** sekmesine tıklayın → **...** → **New with options...** seçeneğine tıklayın  
3. Varsayılanları kullanın – bu, bu kurs için oluşturulmuş **Generative AI Java Development Environment** özel devcontainer yapılandırmasını seçecektir  
4. **Create codespace** tıklayın  
5. Ortam hazır olana kadar yaklaşık 2 dakika bekleyin  
6. [Adım 2: Azure AI Foundry Sağlama](#adım-2-azure-ai-foundry-sağlama) adımına geçin  

<img src="../../../translated_images/tr/codespaces.9945ded8ceb431a5.webp" alt="Ekran görüntüsü: Codespaces alt menüsü" width="50%">

<img src="../../../translated_images/tr/image.833552b62eee7766.webp" alt="Ekran görüntüsü: Seçeneklerle yeni oluştur" width="50%">

<img src="../../../translated_images/tr/codespaces-create.b44a36f728660ab7.webp" alt="Ekran görüntüsü: Codespace oluşturma seçenekleri" width="50%">


> **Codespaces'in Faydaları**:  
> - Yerel kurulum gerektirmez  
> - Tarayıcı olan her cihazda çalışır  
> - Tüm araçlar ve bağımlılıklarla önceden yapılandırılmıştır  
> - Kişisel hesaplar için ayda 60 saat ücretsiz  
> - Tüm öğrenenler için tutarlı ortam sağlar  

#### Seçenek B: Yerel Geliştirme Konteyneri

**Docker ile yerel geliştirmeyi tercih eden geliştiriciler için**

1. Depoyu kendi bilgisayarınıza fork edip klonlayın  
   > **Not**: Temel yapılandırmayı düzenlemek isterseniz [Dev Container Configuration](../../../.devcontainer/devcontainer.json) dosyasına göz atın  
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) ve [VS Code](https://code.visualstudio.com/) kurun  
3. VS Code'da [Dev Containers uzantısını](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) yükleyin  
4. Depo klasörünü VS Code’da açın  
5. İstendiğinde **Reopen in Container** tıklayın (veya `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" komutunu kullanın)  
6. Konteynerin inşası ve başlatılması tamamlanana kadar bekleyin  
7. [Adım 2: Azure AI Foundry Sağlama](#adım-2-azure-ai-foundry-sağlama) adımına geçin  

<img src="../../../translated_images/tr/devcontainer.21126c9d6de64494.webp" alt="Ekran görüntüsü: Geliştirme konteyneri kurulumu" width="50%">

<img src="../../../translated_images/tr/image-3.bf93d533bbc84268.webp" alt="Ekran görüntüsü: Geliştirme konteyneri inşası tamamlandı" width="50%">

#### Seçenek C: Varolan Yerel Kurulumunuzu Kullanın

**Zaten yerelde Java ortamı olan geliştiriciler için**

Önkoşullar:  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) veya tercih ettiğiniz IDE  

Adımlar:  
1. Depoyu yerel bilgisayarınıza klonlayın  
2. Projeyi IDE'nizde açın  
3. [Adım 2: Azure AI Foundry Sağlama](#adım-2-azure-ai-foundry-sağlama) adımına geçin  

> **Pratik İpucu**: Düşük özellikli bir bilgisayarınız varsa ama yerel VS Code kullanmak istiyorsanız, GitHub Codespaces kullanın! Yerel VS Code'unuzu bulutta barındırılan bir Codespace'e bağlayarak en iyisini yapabilirsiniz.  

<img src="../../../translated_images/tr/image-2.fc0da29a6e4d2aff.webp" alt="Ekran görüntüsü: oluşturulmuş yerel geliştirme konteyneri örneği" width="50%">

## Adım 2: Azure AI Foundry Sağlama

Kursun AI modellerini Azure AI Foundry üzerinde kod olarak dağıtın. Depo kökünden:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` sizden ortam adı ve bölge ister, `gpt-4o-mini` ve `text-embedding-3-small` dağıtımlarıyla bir Azure AI Foundry hesabı sağlar ve endpoint bilgisini örneğin `.env` dosyasına yazar — tümü **anahtarsız** kimlik doğrulama ile (API anahtarı yok).

> **Tam kılavuz:** Önkoşullar, manuel (portal) alternatif, bölge rehberi ve maliyet/temizlik notları için [Azure AI Foundry Kurulum Kılavuzuna](getting-started-azure-openai.md) bakın.

## Adım 3: Kurulumunuzu Test Edin

Foundry modellerinizi sağladıktan sonra, örnek uygulamayla bağlantıyı test edin: [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) içinde.

1. Geliştirme ortamınızda terminali açın.  
2. Örneğe gidin:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Oturum açtığınızdan emin olun (anahtarsız kimlik doğrulama için token gerekir):  
   ```bash
   az login
   ```
  
   > Eğer `azd up` komutunu çalıştırdıysanız endpoint içeren `.env` dosyası size otomatik olarak yazılmıştır.  
4. Uygulamayı çalıştırın:  
   ```bash
   mvn clean spring-boot:run
   ```
  
`gpt-4o-mini` modelinden bir yanıt görmelisiniz.

### Örnek Kodu Anlamak

`examples/basic-chat-azure` altındaki örnek, **Spring AI** kullanarak Azure AI Foundry’ye anahtarsız kimlik doğrulama ile bağlanan bir Spring Boot uygulamasıdır.

**Bu kod ne yapar:**  
- Azure AI Foundry’ye Azure oturum açmanız (Microsoft Entra ID) ile bağlanır — API anahtarı yok  
- `gpt-4o-mini` modeline bir soru gönderir  
- Yapay zekanın yanıtını alır ve gösterir  
- Kurulumunuzun doğru çalıştığını doğrular  

**Ana Bağımlılık** (`pom.xml` içinde):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Yapılandırma** (`application.yml`):  
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
  
## Özet  

Harika! Artık her şey hazır:  

- Bicep + `azd` ile kod olarak Azure AI Foundry modelleri sağlandı  
- Java geliştirme ortamınız çalışıyor (Codespaces, geliştirme konteynerleri veya yerel fark etmez)  
- Azure AI Foundry’e anahtarsız kimlik doğrulama (Microsoft Entra ID) ile bağlandınız — API anahtarı yok  
- Modelinizle konuşan basit bir örnekle her şeyin çalıştığını test ettiniz  

## Sonraki Adımlar

[3. Bölüm: Temel Üretken Yapay Zeka Teknikleri](../03-CoreGenerativeAITechniques/README.md)

## Sorun Giderme

Sorun mu yaşıyorsunuz? İşte yaygın problemler ve çözümleri:

- **Kimlik doğrulama başarısız (401/403)?**  
  - `az login` komutunu çalıştırın — kimlik doğrulama anahtarsızdır ve oturum açmak zorundasınız  
  - Hesabınızda kaynağa **Cognitive Services OpenAI User** rolünün atandığını doğrulayın  
  - Yeni sağladıysanız, rol atamasının yayılması için birkaç dakika bekleyin  

- **Maven bulunamıyor?**  
  - Geliştirme konteynerleri/Codespaces kullanıyorsanız Maven önceden kuruludur  
  - Yerel kurulumda Java 21+ ve Maven 3.9+ olduğundan emin olun  
  - Kurulumu doğrulamak için `mvn --version` komutunu deneyin  

- **`azd` bulunamıyor veya sağlama başarısız oluyor?**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) yükleyin ve `azd auth login` ile giriş yapın  
  - `gpt-4o-mini` mevcut bir bölge seçin (örneğin `eastus2`)  
  - Detaylar için [Azure AI Foundry kurulum kılavuzuna](getting-started-azure-openai.md) bakın  

- **Geliştirme konteyneri başlamıyor?**  
  - Docker Desktop’un çalıştığından emin olun (yerel geliştirme için)  
  - Konteyneri yeniden inşa etmeyi deneyin: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"  

- **Uygulama derleme hataları?**  
  - Doğru dizinde olduğunuzdan emin olun: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Temizleyip yeniden derlemeyi deneyin: `mvn clean compile`  

> **Yardıma mı ihtiyacınız var?**: Hâlâ sorun mu yaşıyorsunuz? Repoda bir issue açın, size yardımcı olalım.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba sarf etsek de, otomatik çevirilerin hata veya yanlışlık içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek yanlış anlamalardan veya yanlış yorumlamalardan sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->