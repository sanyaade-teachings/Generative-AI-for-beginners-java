# Praktické aplikace a projekty

## Co se naučíte
V této sekci předvedeme tři praktické aplikace, které ukazují vývojové vzory generativní AI v Javě:
- Vytvoření multimodálního generátoru příběhů o mazlíčcích kombinujícího klientskou a serverovou AI
- Implementace integrace lokálního AI modelu s ukázkou Foundry Local Spring Boot
- Vývoj služby Model Context Protocol (MCP) pomocí příkladu kalkulačky

## Obsah

- [Úvod](#úvod)
  - [Foundry Local Spring Boot Demo](#foundry-local-spring-boot-demo)
  - [Generátor příběhů o mazlíčcích](#generátor-příběhů-o-mazlíčcích)
  - [Služba MCP kalkulačky (MCP demo vhodné pro začátečníky)](#služba-mcp-kalkulačky-mcp-demo-vhodné-pro-začátečníky)
- [Postup učení](#postup-učení)
- [Shrnutí](#shrnutí)
- [Další kroky](#další-kroky)

## Úvod

Tato kapitola představuje **ukázkové projekty**, které demonstrují vývojové vzory generativní AI v Javě. Každý projekt je plně funkční a předvádí konkrétní AI technologie, architektonické vzory a osvědčené postupy, které můžete přizpůsobit pro své vlastní aplikace.

### Foundry Local Spring Boot Demo

**[Foundry Local Spring Boot Demo](foundrylocal/README.md)** ukazuje, jak integrovat lokální AI modely pomocí **OpenAI Java SDK**. Demonstruje připojení k modelům běžícím na Foundry Local (např. **Phi-4-mini**) s automatickou detekcí modelu, což umožňuje spouštět AI aplikace bez spoléhání se na cloudové služby.

### Generátor příběhů o mazlíčcích

**[Generátor příběhů o mazlíčcích](petstory/README.md)** je poutavá webová aplikace Spring Boot, která demonstruje **multimodální zpracování AI** pro generování kreativních příběhů o mazlíčcích. Kombinuje klientské a serverové AI schopnosti za pomoci transformer.js pro AI interakce v prohlížeči a OpenAI SDK pro serverové zpracování.

### Služba MCP kalkulačky (MCP demo vhodné pro začátečníky)

**[Služba MCP kalkulačky](calculator/README.md)** je jednoduchá ukázka **Model Context Protocol (MCP)** využívající Spring AI. Poskytuje přátelský úvod do konceptů MCP a ukazuje, jak vytvořit základní MCP server, který komunikuje s MCP klienty.

## Postup učení

Tyto projekty jsou navrženy tak, aby navazovaly na koncepty z předchozích kapitol:

1. **Začněte jednoduše**: Začněte demo Foundry Local Spring Boot, abyste pochopili základní integraci AI s lokálními modely
2. **Přidejte interaktivitu**: Pokračujte k generátoru příběhů o mazlíčcích pro multimodální AI a webové interakce
3. **Naučte se základy MCP**: Vyzkoušejte službu MCP kalkulačky, abyste pochopili základy Model Context Protocol

## Shrnutí

Dobrá práce! Nyní jste prozkoumali několik reálných aplikací:

- Multimodální AI zkušenosti fungující jak v prohlížeči, tak na serveru
- Integrace lokálních AI modelů pomocí moderních Java frameworků a SDK
- Vaše první služba Model Context Protocol pro ukázku, jak nástroje integrují s AI

## Další kroky

[Kap. 5: Zodpovědná generativní AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->