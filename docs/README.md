# 🔮 Arcane - AI Novel Translator

**Version:** 0.1.0  
**Stack:** TypeScript, Node.js, Express, LowDB, OpenAI API, **Preact + Vite**  
**Focus:** EN → RU translation of fiction literature

---

## 📋 Overview

Arcane is a system for translating novels using AI. Key features:

- **3-stage pipeline** — analysis → translation → editing
- **Context agent** — maintains state between chapters
- **Glossary** — consistent translation of names and terms
  - **Automatic extraction** — characters, locations, and terms are identified during analysis
  - **Descriptions** — automatically extracted and used in prompts for better context
  - **First appearance** — tracking the chapter number of first mention
  - **Image gallery** — multiple images for each entry
- **Declensions** — automatic case forms for Russian language
- **Persistent storage** — LowDB for data persistence
- **EPUB/FB2 export** — generate files for e-readers
- **Reading mode** — full-screen interface for reading translated chapters

---

## 🏗️ Architecture

```
arcane/
├── package.json              # Monorepo root (npm workspaces)
├── docs/                     # Documentation
│
├── arcane-engine/            # 🔧 Translation engine
│   └── src/
│       ├── agents/           # NovelAgent - work context
│       ├── pipeline/         # TranslationPipeline - orchestration
│       ├── stages/           # 3 translation stages
│       ├── prompts/          # System prompts
│       ├── glossary/         # Glossary and declensions
│       ├── providers/        # LLM providers (OpenAI)
│       └── types/            # TypeScript types
│
└── arcane-reader/            # 📖 Web UI + server
    ├── public/               # Static files
    ├── vite.config.ts        # Vite configuration
    └── src/
        ├── server.ts         # Express API server
        ├── services/         # Engine integration
        ├── storage/          # LowDB database
        ├── config.ts         # Configuration
        └── client/           # ⚡ Preact SPA (NEW)
            ├── main.tsx      # Entry point
            ├── App.tsx       # Main component
            ├── api/          # API client
            ├── types/        # TypeScript client types
            ├── styles/       # CSS styles
            └── components/   # UI components
```

---

## 🚀 Quick Start

```bash
# 1. Clone repository
cd arcane

# 2. Install dependencies
npm install

# 3. Configure API key
# Copy arcane-reader/env.example.txt → arcane-reader/.env
# Add OPENAI_API_KEY=sk-...

# 4. Build engine
npm run build

# 5. Start server
npm run dev

# Open http://localhost:3000
```

---

## 📦 Modules

### arcane-engine

Translation engine core. Can be used as a standalone library.

```typescript
import { 
  TranslationPipeline, 
  OpenAIProvider, 
  NovelAgent,
  translateAndDeclineName 
} from 'arcane-engine';
```

[→ More details: engine.md](./engine.md)

### arcane-reader

Web interface and API server.

- Project and chapter management
- Glossary with auto-declensions
- Drag-n-drop file upload
- Translation display

[→ More details: reader.md](./reader.md)

---

## 🔗 API Reference

[→ More details: api.md](./api.md)

---

## 📚 See Also

- [engine.md](./engine.md) — engine documentation
- [reader.md](./reader.md) — UI/server documentation  
- [api.md](./api.md) — REST API
- [prompts.md](./prompts.md) — system prompts
- [glossary.md](./glossary.md) — glossary usage
