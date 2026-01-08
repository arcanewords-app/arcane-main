# Arcane - AI Novel Translator

## 📋 Przegląd projektu

Arcane to inteligentny tłumacz powieści wykorzystujący AI z 3-etapowym potokiem przetwarzania i agentem kontekstowym dla zachowania spójności tłumaczenia.

---

## 🏗️ Architektura modułów

| Moduł | Przeznaczenie |
|-------|---------------|
| **arcane-core** | Jądro systemu — wspólne typy, narzędzia, konfiguracja, interfejsy |
| **arcane-engine** | Silnik tłumaczenia — prompty, praca z AI API, słowniki, agent |
| **arcane-reader** | Frontend/UI — czytanie, ładowanie plików, zarządzanie projektami |

---

## 🎯 3-etapowy potok tłumaczenia

```
┌─────────────────────────────────────────────────────────────────┐
│                        ARCANE PIPELINE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📖 Novel File (.txt)                                           │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  STAGE 1: AGENT (Analiza)                               │   │
│  │  • Ekstrakcja postaci, lokacji, terminów                │   │
│  │  • Określenie stylu autora                              │   │
│  │  • Tworzenie/aktualizacja słownika                      │   │
│  │  • Analiza kontekstu rozdziału                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│       │                                                         │
│       ▼  [Słownik + Kontekst]                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  STAGE 2: TRANSLATOR (Dokładne tłumaczenie)             │   │
│  │  • Tłumaczenie z uwzględnieniem słownika                │   │
│  │  • Prawidłowa odmiana imion                             │   │
│  │  • Zachowanie stylistyki                                │   │
│  │  • Kontekstowe tłumaczenie                              │   │
│  └─────────────────────────────────────────────────────────┘   │
│       │                                                         │
│       ▼  [Wersja robocza]                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  STAGE 3: EDITOR (Redakcja)                             │   │
│  │  • Obróbka literacka                                    │   │
│  │  • Sprawdzenie spójności                                │   │
│  │  • Poprawienie nienaturalnych fraz                      │   │
│  │  • Ostateczna korekta                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│       │                                                         │
│       ▼                                                         │
│  📚 Translated Chapter                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 Struktura projektu

### arcane-core

```
arcane-core/
├── src/
│   ├── types/
│   │   ├── novel.ts          # Novel, Chapter, Paragraph
│   │   ├── glossary.ts       # GlossaryEntry, Character, Term
│   │   ├── translation.ts    # TranslationResult, TranslationStage
│   │   └── agent.ts          # AgentContext, AnalysisResult
│   ├── interfaces/
│   │   ├── parser.ts         # IParser interface
│   │   ├── llm-provider.ts   # ILLMProvider interface
│   │   └── storage.ts        # IStorage interface
│   ├── utils/
│   │   ├── chunker.ts        # Podział tekstu na fragmenty
│   │   ├── text-utils.ts     # Narzędzia tekstowe
│   │   └── logger.ts         # Logowanie
│   └── index.ts
├── package.json
└── tsconfig.json
```

### arcane-engine

```
arcane-engine/
├── src/
│   ├── agents/
│   │   └── novel-agent.ts       # Agent powieści (pamięć, słownik)
│   ├── stages/
│   │   ├── stage-1-analyze.ts   # Etap analizy
│   │   ├── stage-2-translate.ts # Etap tłumaczenia
│   │   └── stage-3-edit.ts      # Etap redakcji
│   ├── prompts/
│   │   ├── system/
│   │   │   ├── analyzer.ts      # Systemowy prompt dla analizy
│   │   │   ├── translator.ts    # Systemowy prompt dla tłumaczenia
│   │   │   └── editor.ts        # Systemowy prompt dla redakcji
│   │   └── templates/
│   │       ├── analyze.ts       # Szablon zapytania analizy
│   │       ├── translate.ts     # Szablon zapytania tłumaczenia
│   │       └── edit.ts          # Szablon zapytania redakcji
│   ├── providers/
│   │   ├── openai.ts            # OpenAI provider
│   │   ├── anthropic.ts         # Claude provider
│   │   └── ollama.ts            # Lokalne LLM
│   ├── glossary/
│   │   ├── glossary-manager.ts  # Zarządzanie słownikiem
│   │   └── declension.ts        # Odmiana (dla rosyjskiego/polskiego)
│   ├── pipeline/
│   │   └── translation-pipeline.ts  # Orkiestracja 3 etapów
│   ├── storage/
│   │   ├── project-storage.ts   # Przechowywanie projektu
│   │   └── cache.ts             # Cache tłumaczeń
│   └── index.ts
├── package.json
└── tsconfig.json
```

### arcane-reader

```
arcane-reader/
├── src/
│   ├── parsers/
│   │   ├── txt-parser.ts        # Parser .txt
│   │   └── epub-parser.ts       # (przyszłość) Parser .epub
│   ├── cli/
│   │   ├── commands/
│   │   │   ├── translate.ts     # Komenda tłumaczenia
│   │   │   ├── glossary.ts      # Zarządzanie słownikiem
│   │   │   └── project.ts       # Zarządzanie projektem
│   │   └── index.ts
│   ├── api/                     # (przyszłość) REST API
│   └── web/                     # (przyszłość) Web UI
├── package.json
└── tsconfig.json
```

---

## 🧠 Agent powieści

```typescript
interface NovelAgent {
  novelId: string;
  title: string;
  sourceLanguage: Language;
  targetLanguage: Language;
  
  glossary: {
    characters: Character[];
    locations: Location[];
    terms: Term[];
    items: Item[];
  };
  
  styleProfile: {
    tone: string;
    narrativeVoice: string;
    dialogueStyle: string;
  };
  
  translatedChapters: ChapterSummary[];
  
  currentContext: {
    lastEvents: string[];
    activeCharacters: string[];
    currentLocation: string;
  };
}

interface Character {
  originalName: string;
  translatedName: string;
  declensions: {
    nominative: string;
    genitive: string;
    dative: string;
    accusative: string;
    instrumental: string;
    prepositional: string;
  };
  gender: 'male' | 'female' | 'unknown';
  description: string;
  aliases: string[];
}
```

---

## 📋 Plan realizacji

| Etap | Moduł | Zadania | Priorytet |
|------|-------|---------|-----------|
| 1 | core | Podstawowe typy, interfejsy | 🔴 Wysoki |
| 2 | engine | LLM provider (OpenAI) | 🔴 Wysoki |
| 3 | engine | Stage 1 - Analizator | 🔴 Wysoki |
| 4 | engine | Słownik + odmiana | 🔴 Wysoki |
| 5 | engine | Stage 2 - Tłumacz | 🔴 Wysoki |
| 6 | engine | Stage 3 - Redaktor | 🟡 Średni |
| 7 | reader | TXT parser | 🟡 Średni |
| 8 | reader | CLI interfejs | 🟡 Średni |
| 9 | engine | Cache | 🟢 Niski |
| 10 | reader | Web UI | 🟢 Niski |

---

## 🔧 Stack technologiczny

- **Runtime**: Node.js
- **Język**: TypeScript
- **AI Providers**: OpenAI, Anthropic Claude, Ollama
- **Storage**: JSON files (MVP), SQLite/PostgreSQL (przyszłość)
- **CLI**: Commander.js
- **Web**: React + Vite (przyszłość)

