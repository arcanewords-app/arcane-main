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

### Core Translation
- **3-Stage AI Pipeline** — Analyze → Translate → Edit with context awareness
- **Smart Glossary** — Consistent character/location names across chapters
- **Russian Morphology** — Automatic noun declensions (6 grammatical cases)
- **Context Agent** — Maintains story context between chapters
- **Stage-Specific Models** — Different AI models for analysis, translation, and editing stages
- **Configurable Creativity** — Temperature control for translation quality

### User Interface
- **Modern Dashboard** — Kindle-like project grid with cover images
- **Project Types** — Support for books (EPUB/FB2) and plain text
- **Cover Images** — Upload and manage project covers
- **Responsive Design** — Mobile, tablet, and desktop layouts
- **Sidebar Navigation** — Context-aware sidebar for chapter navigation
- **Reading Mode** — Full-screen reading interface for translated chapters
- **Original Reading Mode** — Read-only mode for projects without translation

### Data & Export
- **Supabase Integration** — PostgreSQL database with Row Level Security
- **User Authentication** — Email/password with email confirmation
- **Export to EPUB/FB2** — Generate e-book files for reading apps
- **State Management** — @preact/signals for efficient caching
- **Client-Side Routing** — preact-router for SPA navigation

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
        ├── services/       # Supabase integration
        ├── middleware/     # Authentication middleware
        └── client/         # Preact SPA
            ├── pages/      # Route pages (Dashboard, ProjectPage, ChapterPage)
            ├── components/ # UI components
            ├── store/      # State management (@preact/signals)
            ├── api/        # Typed API client
            └── styles/     # CSS with variables
```

---

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- OpenAI API key
- Supabase account and project

### Installation

```bash
# Clone repository
git clone https://github.com/your-username/arcane.git
cd arcane

# Install dependencies
npm install

# Configure environment
cp arcane-reader/env.example.txt arcane-reader/.env
# Edit .env and add:
#   OPENAI_API_KEY=sk-...
#   SUPABASE_URL=https://your-project.supabase.co
#   SUPABASE_ANON_KEY=your-anon-key
#   SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Run database migrations
# See arcane-reader/migrations/README.md

# Build engine
npm run build

# Start development server
npm run dev

# Open http://localhost:3000
```

### Database Setup

1. Create a Supabase project at https://supabase.com
2. Run migration scripts from `arcane-reader/migrations/`
3. Configure Row Level Security (RLS) policies
4. Set environment variables in `.env`

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

- **Dashboard** — Main page with project grid (Kindle-like layout)
- **Project Management** — Create, edit, delete projects (books/text)
- **Chapter Management** — Upload, view, and edit chapters
- **Smart Glossary** — Auto-declensions with image support
- **Translation Settings** — Configure AI models, creativity, pipeline stages
- **Reading Modes** — Translation mode and original-only reading mode
- **Cover Images** — Upload and manage project covers
- **Export** — Generate EPUB/FB2 files
- **Authentication** — User accounts with email confirmation

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
| **Frontend** | Preact, Vite, TypeScript, CSS Variables, preact-router |
| **State Management** | @preact/signals |
| **Backend** | Node.js, Express, TypeScript |
| **Database** | Supabase (PostgreSQL with RLS) |
| **Authentication** | Supabase Auth |
| **AI** | OpenAI GPT-4/5, Custom Prompts |
| **Build** | npm workspaces, Vite |
| **File Upload** | Multer, FormData |

---

## 📖 Documentation

Detailed documentation in [`/docs`](./docs/):

- [Engine Documentation](./docs/engine.md) — Translation pipeline & agents
- [Reader Documentation](./docs/reader.md) — Web UI & API server
- [API Reference](./docs/api.md) — REST endpoints
- [Prompts Guide](./docs/prompts.md) — System prompts
- [Glossary Guide](./docs/glossary.md) — Declensions & terminology
- [Types Reference](./docs/types.md) — TypeScript types

## 🔐 Authentication & Security

- **User Accounts** — Email/password registration and login
- **Email Confirmation** — Required for account activation
- **Row Level Security** — Supabase RLS ensures data isolation
- **JWT Tokens** — Secure authentication tokens
- **Protected Routes** — All API endpoints require authentication

## 📱 User Interface

### Pages

- **Dashboard (`/`)** — Main page with project grid, search, and filters
- **Project Page (`/projects/:id`)** — Project information, settings, chapters list
- **Chapter Page (`/projects/:id/chapters/:chapterId`)** — Chapter view and editing
- **Reading Mode (`/projects/:id/chapters/:chapterId/reading`)** — Full-screen reading

### Features

- **Responsive Design** — Mobile-first approach with breakpoints for tablet/desktop
- **Project Cards** — Cover images, progress bars, metadata
- **Sidebar Navigation** — Context-aware sidebar for chapters and project navigation
- **Settings Modal** — Centralized project settings (AI models, creativity, stages)
- **Glossary Modal** — Manage glossary entries with images
- **Loading States** — Proper loading indicators and error handling

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

