# Praktikal na Mga Aplikasyon at Proyekto

## Ano ang Matututuhan Mo
Sa bahaging ito ipapakita namin ang tatlong praktikal na aplikasyon na nagpapakita ng mga pattern ng pag-develop ng generative AI gamit ang Java:
- Gumawa ng multi-modal na Pet Story Generator na pinaghalo ang client-side at server-side AI
- Ipatupad ang lokal na integrasyon ng AI model gamit ang Foundry Local Spring Boot demo
- Bumuo ng Model Context Protocol (MCP) service gamit ang Calculator na halimbawa

## Talaan ng mga Nilalaman

- [Panimula](#panimula)
  - [Foundry Local Spring Boot Demo](#foundry-local-spring-boot-demo)
  - [Pet Story Generator](#pet-story-generator)
  - [MCP Calculator Service (Beginner-Friendly MCP Demo)](#mcp-calculator-service-beginner-friendly-mcp-demo)
- [Pag-unlad ng Pagkatuto](#pag-unlad-ng-pagkatuto)
- [Buod](#buod)
- [Mga Susunod na Hakbang](#mga-susunod-na-hakbang)

## Panimula

Ipinapakita ng kabanatang ito ang **mga halimbawa ng proyekto** na nagtatanghal ng mga pattern ng pag-develop ng generative AI gamit ang Java. Ang bawat proyekto ay ganap na gumagana at nagpapakita ng tiyak na mga teknolohiya ng AI, mga pattern ng arkitektura, at mga pinakamahusay na kasanayan na maaari mong iangkop para sa sarili mong mga aplikasyon.

### Foundry Local Spring Boot Demo

Ipinapakita ng **[Foundry Local Spring Boot Demo](foundrylocal/README.md)** kung paano mag-integrate sa lokal na mga AI model gamit ang **OpenAI Java SDK**. Ipinapakita nito ang pagkonekta sa mga modelong tumatakbo sa Foundry Local (hal., **Phi-4-mini**), na may awtomatikong pagtuklas ng modelo, na nagpapahintulot sa iyo na magpatakbo ng mga AI na aplikasyon nang hindi umaasa sa mga serbisyo sa cloud.

### Pet Story Generator

Ang **[Pet Story Generator](petstory/README.md)** ay isang nakakaengganyong Spring Boot web application na nagpapakita ng **multi-modal AI processing** upang makabuo ng malikhaing mga kuwento ng alagang hayop. Pinaghalo nito ang client-side at server-side AI na kakayahan gamit ang transformer.js para sa browser-based na AI interactions at OpenAI SDK para sa server-side na pagproseso.

### MCP Calculator Service (Beginner-Friendly MCP Demo)

Ang **[MCP Calculator Service](calculator/README.md)** ay isang simpleng demonstrasyon ng **Model Context Protocol (MCP)** na gumagamit ng Spring AI. Nagbibigay ito ng madaling maintindihan na pagpapakilala sa mga konsepto ng MCP, na nagpapakita kung paano gumawa ng isang pangunahing MCP Server na nakikipag-ugnayan sa mga MCP client.

## Pag-unlad ng Pagkatuto

Ang mga proyektong ito ay dinisenyo upang itayo sa mga konsepto mula sa mga naunang kabanata:

1. **Magsimula nang Simple**: Simulan sa Foundry Local Spring Boot Demo upang maunawaan ang pangunahing AI integration gamit ang mga lokal na modelo
2. **Magdagdag ng Interaktibidad**: Magpatuloy sa Pet Story Generator para sa multi-modal AI at web-based na mga interaction
3. **Matutunan ang Mga Pangunahing Kaalaman sa MCP**: Subukan ang MCP Calculator Service upang maunawaan ang mga pundasyon ng Model Context Protocol

## Buod

Magaling! Ngayon ay nasiyasat mo na ang ilang tunay na aplikasyon:

- Mga multi-modal na karanasan sa AI na gumagana pareho sa browser at sa server
- Lokal na integrasyon ng AI model gamit ang mga modernong Java framework at SDK
- Ang iyong unang Model Context Protocol service upang makita kung paano nag-iintegrate ang mga tool sa AI

## Mga Susunod na Hakbang

[Chapter 5: Responsible Generative AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->