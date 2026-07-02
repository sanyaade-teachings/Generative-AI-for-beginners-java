# Üretken Yapay Zekaya Giriş - Java Sürümü

## Neler Öğreneceksiniz

- LLM'ler, prompt mühendisliği, tokenlar, embeddingler ve vektör veritabanları dahil **Üretken Yapay Zeka temelleri**
- Azure OpenAI SDK, Spring AI ve OpenAI Java SDK dahil **Java Yapay Zeka geliştirme araçlarını karşılaştırma**
- **Model Context Protocol** ve AI ajan iletişimindeki rolünü keşfetme

## İçindekiler

- [Giriş](#giriş)
- [Üretken Yapay Zeka kavramlarına hızlı bir refresher](#üretken-yapay-zeka-kavramlarına-hızlı-bir-refresher)
- [Prompt mühendisliği incelemesi](#prompt-mühendisliği-incelemesi)
- [Tokenlar, embeddingler ve ajanlar](#tokenlar-embeddingler-ve-ajanlar)
- [Java için AI Geliştirme Araçları ve Kütüphaneleri](#java-için-ai-geliştirme-araçları-ve-kütüphaneleri)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Özet](#özet)
- [Sonraki Adımlar](#sonraki-adımlar)

## Giriş

Üretken Yapay Zeka Başlangıç Rehberi - Java Sürümü'nün ilk bölümüne hoş geldiniz! Bu temel ders, sizi üretken yapay zekanın temel kavramlarıyla tanıştırır ve bunlarla Java kullanarak nasıl çalışacağınızı gösterir. Büyük Dil Modelleri (LLM'ler), tokenlar, embeddingler ve AI ajanları gibi AI uygulamalarının temel yapı taşlarını öğreneceksiniz. Ayrıca bu kurs boyunca kullanacağınız birincil Java araçlarını keşfedeceğiz.

### Üretken Yapay Zeka kavramlarına hızlı bir refresher

Üretken Yapay Zeka, verilerden öğrenilen örüntüler ve ilişkiler temelinde metin, resim veya kod gibi yeni içerik oluşturan bir yapay zeka türüdür. Üretken AI modelleri insan benzeri yanıtlar üretebilir, bağlamı anlayabilir ve bazen insan gibi görünen içerikler yaratabilir.

Java AI uygulamalarınızı geliştirirken, **üretken AI modelleriyle** içerik oluşturacaksınız. Üretken AI modellerinin bazı yetenekleri şunlardır:

- **Metin üretimi**: Chatbotlar, içerik ve metin tamamlama için insan benzeri metinler oluşturma.
- **Resim üretimi ve analizi**: Gerçekçi resimler yaratma, fotoğrafları iyileştirme ve nesne algılama.
- **Kod üretimi**: Kod parçacıkları veya betikler yazma.

Farklı görevler için optimize edilmiş özel model tipleri vardır. Örneğin, hem **Küçük Dil Modelleri (SLM)** hem de **Büyük Dil Modelleri (LLM)** metin üretimi yapabilir; LLM'ler genellikle karmaşık görevlerde daha iyi performans sunar. Görüntü ile ilgili görevlerde ise özel görsel modeller veya çok modlu modeller kullanılır.

![Figure: Generative AI model types and use cases.](../../../translated_images/tr/llms.225ca2b8a0d34473.webp)

Elbette, bu modellerin yanıtları her zaman mükemmel değildir. Modellerin bazen "halüsinasyon" yapması veya yanlış bilgileri kesin ifadelerle üretmesi hakkında duymuşsunuzdur. Ancak modelleri daha iyi yanıtlar üretmeye yönlendirmek için onlara net talimatlar ve bağlam sağlayabilirsiniz. İşte burada **prompt mühendisliği** devreye girer.

#### Prompt mühendisliği incelemesi

Prompt mühendisliği, yapay zeka modellerini istenen çıktılara yönlendirmek için etkili girdiler tasarlama pratiğidir. İçerir:

- **Açıklık**: Talimatların net ve kesin olması.
- **Bağlam**: Gerekli temel bilgiler sağlama.
- **Kısıtlamalar**: Herhangi bir sınırlama veya format belirleme.

Prompt mühendisliği için bazı en iyi uygulamalar; prompt tasarımı, net talimatlar, görev bölümü, tek-örnek ve biraz-örnek öğrenme ve prompt ayarıdır. Farklı promptları test etmek, belirli kullanım durumunuz için en iyi sonucu bulmak için çok önemlidir.

Uygulama geliştirirken farklı prompt türleriyle çalışacaksınız:
- **Sistem promptları**: Modelin davranışı için temel kuralları ve bağlamı belirler
- **Kullanıcı promptları**: Uygulama kullanıcılarınızdan gelen giriş verisi
- **Asistan promptları**: Sistem ve kullanıcı promptlarına dayalı model yanıtları

> **Daha fazla bilgi**: [Üretken AI Başlangıç Kılavuzu'nun Prompt Mühendisliği bölümü](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals) hakkında detaylı bilgi alabilirsiniz.

#### Tokenlar, embeddingler ve ajanlar

Üretken AI modelleriyle çalışırken, **tokenlar**, **embeddingler**, **ajanlar** ve **Model Context Protocol (MCP)** terimleriyle karşılaşacaksınız. İşte bu kavramların detaylı bir özeti:

- **Tokenlar**: Tokenlar modeldeki en küçük metin birimidir. Kelimeler, karakterler veya alt kelimeler olabilirler. Tokenlar, modeli anlayabileceği biçimde metin verisini temsil etmek için kullanılır. Örneğin, "The quick brown fox jumped over the lazy dog" cümlesi tokenizasyon stratejisine bağlı olarak ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] veya ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] şeklinde tokenlara ayrılabilir.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/tr/tokens.6283ed277a2ffff4.webp)

Tokenizasyon, metni bu küçük birimlere bölme işlemidir. Bu çok önemlidir çünkü modeller ham metin yerine tokenlar üzerinde çalışır. Bir prompttaki token sayısı modelin yanıt uzunluğunu ve kalitesini etkiler; çünkü modeller bağlam penceresi için token sınırlarına sahiptir (örneğin, GPT-4o için toplam bağlamda 128K token, hem giriş hem çıkış dahil).

  Java'da, OpenAI SDK gibi kütüphaneler AI modellere istek gönderirken tokenizasyon işlemini otomatik olarak yapar.

- **Embeddingler**: Embeddingler, kelimelerin anlamsal anlamını yakalayan vektörel temsillerdir. Bunlar sayı dizileri (genellikle kayan nokta sayı dizileri) şeklindedir ve modellerin kelimeler arasındaki ilişkileri anlamasına ve bağlamsal olarak ilgili yanıtlar üretmesine olanak tanır. Benzer kelimelerin embeddingleri birbirine benzer, bu da modelin eşanlamlılar ve anlamsal ilişkiler gibi kavramları anlamasını sağlar.

![Figure: Embeddings](../../../translated_images/tr/embedding.398e50802c0037f9.webp)

  Java'da embedding oluşturmak için OpenAI SDK veya embedding oluşturmayı destekleyen diğer kütüphaneler kullanılabilir. Embeddingler, anlam temelli arama gibi, tam metin eşleşmeleri yerine anlam bazlı benzer içerik bulma gibi görevler için çok önemlidir.

- **Vektör veritabanları**: Vektör veritabanları embeddingler için optimize edilmiş özel depolama sistemleridir. Etkili benzerlik araması yapılabilmesini sağlarlar ve büyük veri kümeleri arasında semantik benzerliğe dayalı bilgi bulmanın önemli olduğu Retrieval-Augmented Generation (RAG) desenlerinde kritik rol oynarlar.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/tr/vector.f12f114934e223df.webp)

> **Not**: Bu derste Vektör veritabanlarını kapsamıyoruz ancak gerçek dünya uygulamalarında yaygın kullanıldıkları için bahsetmeye değer olduğunu düşünüyoruz.

- **Ajanlar ve MCP**: Modeller, araçlar ve dış sistemlerle otonom etkileşime giren AI bileşenleridir. Model Context Protocol (MCP), ajanların dış veri kaynaklarına ve araçlara güvenli şekilde erişimi için standart bir yol sağlar. Daha fazlası için [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners) kursumuzu inceleyebilirsiniz.

Java AI uygulamalarında, metin işleme için tokenları, anlamsal arama ve RAG için embeddingleri, veri erişimi için vektör veritabanlarını ve akıllı, araç kullanan sistemler inşa etmek için MCP ile ajanları kullanırsınız.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/tr/flow.f4ef62c3052d12a8.webp)

### Java için AI Geliştirme Araçları ve Kütüphaneleri

Java, AI geliştirme için mükemmel araç desteği sunar. Bu kursta keşfedeceğimiz üç ana kütüphane vardır - OpenAI Java SDK, Azure OpenAI SDK ve Spring AI.

Aşağıda her bölümün örneklerinde hangi SDK'nın kullanıldığına dair hızlı bir referans tablosu bulunmaktadır:

| Bölüm | Örnek | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK Dokümantasyon Bağlantıları:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK, OpenAI API için resmi Java kütüphanesidir. OpenAI modelleriyle etkileşim için basit ve tutarlı bir arayüz sağlar ve AI yeteneklerini Java uygulamalarına entegre etmeyi kolaylaştırır. Bölüm 4'teki Pet Story uygulaması ve Foundry Local örneği OpenAI SDK yaklaşımını Azure AI Foundry ile gösterir.

#### Spring AI

Spring AI, AI yeteneklerini Spring uygulamalarına getiren kapsamlı bir çerçevedir ve farklı AI sağlayıcıları için tutarlı bir soyutlama katmanı sunar. Spring ekosistemiyle kusursuz entegrasyon sağlar ve AI yeteneklerine ihtiyaç duyan kurumsal Java uygulamaları için ideal seçimdir.

Spring AI'nin gücü, bağımlılık enjeksiyonu, konfigürasyon yönetimi ve test çerçeveleri gibi tanıdık Spring kalıplarıyla üretime hazır AI uygulamaları oluşturmayı kolaylaştıran Spring ekosistemiyle sorunsuz entegrasyonunda yatar. Bu kursta, Spring AI'yi Bölüm 2 ve 4'te, OpenAI ve Model Context Protocol (MCP) Spring AI kütüphanelerini beraber kullanarak uygulamalar geliştirmek için kullanacaksınız.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) yapay zekanın dış veri kaynakları ve araçlarla güvenli şekilde etkileşim kurmasını sağlayan gelişmekte olan bir standarttır. MCP, AI modellerinin bağlamsal bilgiyi erişmesi ve uygulamanızda işlemler gerçekleştirmesi için standart bir yöntem sunar.

Bölüm 4'te, Model Context Protocol'ün temellerini Spring AI ile gösteren temel araç entegrasyonları ve hizmet mimarisi oluşturan basit bir MCP hesap makinesi servisi geliştireceksiniz.

#### Azure OpenAI Java SDK

Azure OpenAI Java istemci kitaplığı, OpenAI REST API'lerinin uyarlanmış versiyonudur ve Azure SDK ekosistemi ile doğal entegrasyon sağlar. Bölüm 3'te, sohbet uygulamaları, fonksiyon çağrıları ve RAG (Retrieval-Augmented Generation) desenlerini içeren Azure OpenAI SDK kullanarak uygulamalar geliştireceksiniz.

> Not: Özellikler açısından Azure OpenAI SDK, OpenAI Java SDK'nın gerisinde kalmaktadır; bu nedenle gelecekteki projelerde OpenAI Java SDK'yı kullanmayı düşünün.

## Özet

Temelleri öğrendiniz! Artık:

- Üretken yapay zekanın temel kavramlarını - LLM'ler, prompt mühendisliği, tokenlar, embeddingler ve vektör veritabanlarından
- Java AI geliştirme için kullanabileceğiniz araç setlerini: Azure OpenAI SDK, Spring AI ve OpenAI Java SDK'yı
- Model Context Protocol'ün ne olduğunu ve AI ajanlarının dış araçlarla nasıl çalışmasına olanak sağladığını

anlamaktasınız.

## Sonraki Adımlar

[2. Bölüm: Geliştirme Ortamını Kurma](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba sarf etsek de, otomatik çevirilerin hata veya yanlışlık içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek yanlış anlamalardan veya yanlış yorumlamalardan sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->