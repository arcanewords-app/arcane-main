# 🔮 Arcane — AI Novel Translator

<p align="center">
  <strong>AI-powered translation system for fiction (EN → RU)</strong><br>
  Delivering publication-quality translations through a 3-stage pipeline
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TypeScript-5.0-blue?logo=typescript" alt="TypeScript">
  <img src="https://img.shields.io/badge/Node.js-20+-green?logo=node.js" alt="Node.js">
  <img src="https://img.shields.io/badge/Preact-10-673AB8?logo=preact" alt="Preact">
  <img src="https://img.shields.io/badge/Vite-5-646CFF?logo=vite" alt="Vite">
  <img src="https://img.shields.io/badge/OpenAI-GPT--4-412991?logo=openai" alt="OpenAI">
  <img src="https://img.shields.io/license/MIT-yellow" alt="License">
</p>

---

## ✨ Features

- **3-Stage AI Pipeline** — Analyze → Translate → Edit with context awareness
- **Smart Glossary** — Consistent character/location names across chapters
- **Russian Morphology** — Automatic noun declensions (6 grammatical cases)
- **Context Agent** — Maintains story context between chapters
- **Modern Web UI** — Preact SPA with hot reload, ~15KB gzip bundle
- **Monorepo** — Reusable engine package + web application

---

## 🏗️ Architecture

```
arcane/
├── arcane-engine/          # 🔧 Translation engine (standalone library)
│   └── src/
│       ├── pipeline/       # 3-stage translation orchestration
│       ├── stages/         # Analyze, Translate, Edit stages
│       ├── agents/         # NovelAgent — story context manager
│       ├── glossary/       # Declensions (Petrovich library)
│       ├── prompts/        # System prompts for GPT
│       └── providers/      # LLM providers (OpenAI)
│
└── arcane-reader/          # 📖 Web UI + API server
    └── src/
        ├── server.ts       # Express REST API
        ├── storage/        # LowDB persistence
        └── client/         # Preact SPA
            ├── components/ # UI components
            ├── api/        # Typed API client
            └── styles/     # CSS with variables
```

---

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- OpenAI API key

### Installation

```bash
# Clone repository
git clone https://github.com/your-username/arcane.git
cd arcane

# Install dependencies
npm install

# Configure API key
cp arcane-reader/env.example.txt arcane-reader/.env
# Edit .env and add: OPENAI_API_KEY=sk-...

# Build engine
npm run build

# Start development server
npm run dev

# Open http://localhost:3000
```

---

## 📦 Packages

### arcane-engine

Standalone translation engine. Can be used as a library:

```typescript
import { 
  TranslationPipeline, 
  OpenAIProvider, 
  NovelAgent,
  translateAndDeclineName 
} from 'arcane-engine';

// Create provider
const provider = new OpenAIProvider({ apiKey: 'sk-...', model: 'gpt-4' });

// Create pipeline
const pipeline = new TranslationPipeline(provider, agent, glossary);

// Translate chapter
const result = await pipeline.translate(chapterText, {
  stages: { analyze: true, translate: true, edit: true }
});
```

### arcane-reader

Web application with:

- Project & chapter management
- Drag-n-drop file upload
- Smart glossary with auto-declensions
- Side-by-side original/translation view
- Translation progress tracking

---

## 🔧 Translation Pipeline

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Stage 1    │     │  Stage 2    │     │  Stage 3    │
│  ANALYZE    │ ──▶ │  TRANSLATE  │ ──▶ │    EDIT     │
│             │     │             │     │             │
│ • Extract   │     │ • Translate │     │ • Polish    │
│   names     │     │   with      │     │   style     │
│ • Detect    │     │   context   │     │ • Fix       │
│   context   │     │ • Apply     │     │   errors    │
│ • Update    │     │   glossary  │     │ • Ensure    │
│   glossary  │     │             │     │   quality   │
└─────────────┘     └─────────────┘     └─────────────┘
```

---

## 🌐 Tech Stack

| Layer | Technology |
|-------|------------|
| **Frontend** | Preact, Vite, TypeScript, CSS Variables |
| **Backend** | Node.js, Express, TypeScript |
| **Database** | LowDB (JSON file) |
| **AI** | OpenAI GPT-4, Custom Prompts |
| **Build** | npm workspaces, Vite |

---

## 📖 Documentation

Detailed documentation in [`/docs`](./docs/):

- [Engine Documentation](./docs/engine.md) — Translation pipeline & agents
- [Reader Documentation](./docs/reader.md) — Web UI & API server
- [API Reference](./docs/api.md) — REST endpoints
- [Prompts Guide](./docs/prompts.md) — System prompts
- [Glossary Guide](./docs/glossary.md) — Declensions & terminology

---

## 📜 Scripts

```bash
npm run dev          # Start development (API + Vite HMR)
npm run build        # Build engine
npm run build:all    # Build everything
npm run start        # Production server
npm run clean        # Remove dist folders
```

---

## 📄 License

[MIT](./LICENSE)

---

<p align="center">
  <sub>Built with ❤️ for translators who care about quality</sub>
</p>

