# Arcane - AI Novel Translator

## 📋 Project Overview

Arcane is an intelligent AI-powered novel translator with a 3-stage processing pipeline and a context agent for maintaining translation consistency.

---

## 🏗️ Module Architecture

| Module | Purpose |
|--------|---------|
| **arcane-core** | System core — shared types, utilities, configuration, interfaces |
| **arcane-engine** | Translation engine — prompts, AI API integration, glossaries, agent |
| **arcane-reader** | Frontend/UI — reading, file loading, project management |

---

## 🎯 3-Stage Translation Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                        ARCANE PIPELINE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📖 Novel File (.txt)                                           │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  STAGE 1: AGENT (Analysis)                               │   │
│  │  • Extract characters, locations, terms                  │   │
│  │  • Determine author's style                             │   │
│  │  • Create/update glossary                               │   │
│  │  • Analyze chapter context                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│       │                                                         │
│       ▼  [Glossary + Context]                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  STAGE 2: TRANSLATOR (Accurate translation)             │   │
│  │  • Translate with glossary consideration                │   │
│  │  • Proper name declensions                               │   │
│  │  • Preserve stylistics                                  │   │
│  │  • Contextual translation                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│       │                                                         │
│       ▼  [Draft version]                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  STAGE 3: EDITOR (Editing)                              │   │
│  │  • Literary refinement                                   │   │
│  │  • Consistency check                                     │   │
│  │  • Fix unnatural phrases                                 │   │
│  │  • Final proofreading                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│       │                                                         │
│       ▼                                                         │
│  📚 Translated Chapter                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

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
│   │   ├── chunker.ts        # Text chunking
│   │   ├── text-utils.ts     # Text utilities
│   │   └── logger.ts         # Logging
│   └── index.ts
├── package.json
└── tsconfig.json
```

### arcane-engine

```
arcane-engine/
├── src/
│   ├── agents/
│   │   └── novel-agent.ts       # Novel agent (memory, glossary)
│   ├── stages/
│   │   ├── stage-1-analyze.ts   # Analysis stage
│   │   ├── stage-2-translate.ts # Translation stage
│   │   └── stage-3-edit.ts      # Editing stage
│   ├── prompts/
│   │   ├── system/
│   │   │   ├── analyzer.ts      # System prompt for analysis
│   │   │   ├── translator.ts    # System prompt for translation
│   │   │   └── editor.ts        # System prompt for editing
│   │   └── templates/
│   │       ├── analyze.ts       # Analysis query template
│   │       ├── translate.ts    # Translation query template
│   │       └── edit.ts          # Editing query template
│   ├── providers/
│   │   ├── openai.ts            # OpenAI provider
│   │   ├── anthropic.ts         # Claude provider
│   │   └── ollama.ts            # Local LLM
│   ├── glossary/
│   │   ├── glossary-manager.ts  # Glossary management
│   │   └── declension.ts        # Declension (for Russian/Polish)
│   ├── pipeline/
│   │   └── translation-pipeline.ts  # 3-stage orchestration
│   ├── storage/
│   │   ├── project-storage.ts   # Project storage
│   │   └── cache.ts             # Translation cache
│   └── index.ts
├── package.json
└── tsconfig.json
```

### arcane-reader

```
arcane-reader/
├── src/
│   ├── parsers/
│   │   ├── txt-parser.ts        # .txt parser
│   │   └── epub-parser.ts       # (future) .epub parser
│   ├── cli/
│   │   ├── commands/
│   │   │   ├── translate.ts     # Translation command
│   │   │   ├── glossary.ts      # Glossary management
│   │   │   └── project.ts       # Project management
│   │   └── index.ts
│   ├── api/                     # (future) REST API
│   └── web/                     # (future) Web UI
├── package.json
└── tsconfig.json
```

---

## 🧠 Novel Agent

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

## 📋 Implementation Plan

| Stage | Module | Tasks | Priority |
|------|--------|-------|----------|
| 1 | core | Basic types, interfaces | 🔴 High |
| 2 | engine | LLM provider (OpenAI) | 🔴 High |
| 3 | engine | Stage 1 - Analyzer | 🔴 High |
| 4 | engine | Glossary + declension | 🔴 High |
| 5 | engine | Stage 2 - Translator | 🔴 High |
| 6 | engine | Stage 3 - Editor | 🟡 Medium |
| 7 | reader | TXT parser | 🟡 Medium |
| 8 | reader | CLI interface | 🟡 Medium |
| 9 | engine | Cache | 🟢 Low |
| 10 | reader | Web UI | 🟢 Low |

---

## 🔧 Technology Stack

- **Runtime**: Node.js
- **Language**: TypeScript
- **AI Providers**: OpenAI, Anthropic Claude, Ollama
- **Storage**: JSON files (MVP), SQLite/PostgreSQL (future)
- **CLI**: Commander.js
- **Web**: React + Vite (future)
