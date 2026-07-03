# Praktyczne Zastosowania i Projekty

## Czego się nauczysz
W tej sekcji zaprezentujemy trzy praktyczne zastosowania, które pokazują wzorce rozwoju generatywnej AI w Javie:
- Utwórz wielomodalny Generator Opowieści o Zwierzętach łączący AI po stronie klienta i serwera
- Wdroż lokalną integrację modelu AI za pomocą demo Foundry Local Spring Boot
- Opracuj usługę Model Context Protocol (MCP) na przykładzie Kalkulatora

## Spis treści

- [Wprowadzenie](#wprowadzenie)
  - [Foundry Local Spring Boot Demo](#foundry-local-spring-boot-demo)
  - [Pet Story Generator](#pet-story-generator)
  - [MCP Calculator Service (Beginner-Friendly MCP Demo)](#mcp-calculator-service-beginner-friendly-mcp-demo)
- [Postęp nauki](#postęp-nauki)
- [Podsumowanie](#podsumowanie)
- [Kolejne kroki](#kolejne-kroki)

## Wprowadzenie

Ten rozdział prezentuje **przykładowe projekty**, które demonstrują wzorce rozwoju generatywnej AI w Javie. Każdy projekt jest w pełni funkcjonalny i pokazuje konkretne technologie AI, wzorce architektoniczne oraz najlepsze praktyki, które możesz zaadaptować do własnych aplikacji.

### Foundry Local Spring Boot Demo

**[Foundry Local Spring Boot Demo](foundrylocal/README.md)** pokazuje, jak integrować się z lokalnymi modelami AI przy użyciu **OpenAI Java SDK**. Demonstruje połączenie z modelami uruchamianymi na Foundry Local (np. **Phi-4-mini**), z automatycznym wykrywaniem modeli, co pozwala uruchamiać aplikacje AI bez polegania na usługach chmurowych.

### Pet Story Generator

**[Pet Story Generator](petstory/README.md)** to angażująca aplikacja webowa oparta na Spring Boot, która demonstruje **wielomodalne przetwarzanie AI** do generowania kreatywnych opowieści o zwierzętach. Łączy możliwości AI po stronie klienta i serwera, używając transformer.js do interakcji AI w przeglądarce oraz OpenAI SDK do przetwarzania po stronie serwera.

### MCP Calculator Service (Beginner-Friendly MCP Demo)

**[MCP Calculator Service](calculator/README.md)** to prosta demonstracja **Model Context Protocol (MCP)** wykorzystująca Spring AI. Stanowi przyjazne dla początkujących wprowadzenie do koncepcji MCP, pokazując jak stworzyć podstawowy serwer MCP współdziałający z klientami MCP.

## Postęp nauki

Projekty te zostały zaprojektowane tak, aby opierać się na koncepcjach z poprzednich rozdziałów:

1. **Zacznij od podstaw**: Rozpocznij od demo Foundry Local Spring Boot, aby zrozumieć podstawową integrację AI z lokalnymi modelami
2. **Dodaj interaktywność**: Przejdź do Pet Story Generator, by poznać wielomodalne AI i interakcje webowe
3. **Naucz się podstaw MCP**: Wypróbuj MCP Calculator Service, aby zrozumieć podstawy Model Context Protocol

## Podsumowanie

Świetna robota! Poznałeś teraz kilka realnych zastosowań:

- Wielomodalne doświadczenia AI działające zarówno w przeglądarce, jak i na serwerze
- Lokalną integrację modelu AI przy użyciu nowoczesnych frameworków Java i SDK
- Twoją pierwszą usługę Model Context Protocol, by zobaczyć jak narzędzia integrują się z AI

## Kolejne kroki

[Chapter 5: Responsible Generative AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:
Niniejszy dokument został przetłumaczony za pomocą usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dążymy do dokładności, prosimy pamiętać, że automatyczne tłumaczenia mogą zawierać błędy lub niedokładności. Oryginalny dokument w jego języku źródłowym należy uznawać za autorytatywne źródło. W przypadku informacji krytycznych zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z użycia tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->