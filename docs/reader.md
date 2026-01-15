# 📖 Arcane Reader

Web interface and REST API server for the novel translator.

**Client stack:** Preact + Vite + TypeScript

---

## 🚀 Running

### Development (separate servers)

```bash
cd arcane-reader

# Run both servers in parallel
npm run dev

# Or separately:
npm run dev:server  # Express API on :3000
npm run dev:client  # Vite HMR on :5173
```

### Production

```bash
cd arcane-reader

# Build client
npm run build:client

# Start server (serves static files from dist/client)
npm run start
```

Application will be available at `http://localhost:3000`

---

## 📜 NPM Scripts

| Script | Description |
|--------|-------------|
| `npm run dev` | Start dev server (API + Vite) |
| `npm run dev:server` | Express API only |
| `npm run dev:client` | Vite HMR only |
| `npm run build` | Full build (client + server) |
| `npm run build:client` | Build client to `dist/client` |
| `npm run build:server` | Compile TypeScript server |
| `npm run start` | Start production server |
| `npm run kill-port` | Free port 3000 |

---

## ⚙️ Configuration

Create `.env` file in `arcane-reader/`:

```env
# Required
OPENAI_API_KEY=sk-your-key-here

# Optional
OPENAI_MODEL=gpt-4-turbo-preview
PORT=3000
MAX_TOKENS_PER_CHUNK=2000
TRANSLATION_TEMPERATURE=0.7
SKIP_EDITING=false
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENAI_API_KEY` | — | OpenAI API key (required for translation) |
| `OPENAI_MODEL` | `gpt-4-turbo-preview` | Model for translation |
| `PORT` | `3000` | Server port |
| `MAX_TOKENS_PER_CHUNK` | `2000` | Max tokens per chunk |
| `TRANSLATION_TEMPERATURE` | `0.7` | Generation temperature |
| `SKIP_EDITING` | `false` | Skip editing stage |

---

## 📂 Structure

```
arcane-reader/
├── public/
│   ├── index.html          # Entry point for Vite
│   └── arcane_icon.png     # Application icon
│
├── src/
│   ├── server.ts           # Express API server
│   ├── config.ts           # Configuration loading
│   │
│   ├── services/
│   │   ├── engine-integration.ts   # Integration with arcane-engine
│   │   └── translation-service.ts  # Translation service
│   │
│   ├── storage/
│   │   └── database.ts     # LowDB operations
│   │
│   └── client/             # ⚡ Preact SPA
│       ├── main.tsx        # Entry point
│       ├── App.tsx         # Main component
│       ├── vite-env.d.ts   # Vite types
│       │
│       ├── api/
│       │   └── client.ts   # Typed API client
│       │
│       ├── types/
│       │   └── index.ts    # TypeScript interfaces
│       │
│       ├── styles/
│       │   └── index.css   # All styles
│       │
│       └── components/
│           ├── ui/         # Base components
│           │   ├── Button.tsx
│           │   ├── Card.tsx
│           │   ├── Modal.tsx
│           │   ├── Input.tsx
│           │   ├── Badge.tsx
│           │   └── index.ts
│           │
│           ├── Header.tsx
│           ├── ProjectInfo.tsx
│           │
│           ├── Sidebar/
│           │   ├── index.tsx
│           │   ├── ProjectList.tsx
│           │   └── ChapterList.tsx
│           │
│           ├── ChapterView/
│           │   ├── index.tsx
│           │   ├── ChapterHeader.tsx
│           │   ├── ReaderSettings.tsx
│           │   └── ParagraphList.tsx
│           │
│           ├── Glossary/
│           │   ├── index.ts
│           │   └── GlossaryModal.tsx
│           │
│           └── ReadingMode/
│               └── index.tsx
│
├── dist/
│   └── client/             # Production build (generated)
│
├── data/
│   ├── arcane-db.json      # Database (created automatically)
│   ├── images/             # Glossary images
│   └── exports/             # Exported files (EPUB/FB2)
│
├── vite.config.ts          # Vite configuration
├── tsconfig.client.json    # TypeScript for client
├── .env                    # Configuration (not in git)
└── env.example.txt         # Configuration example
```

---

## 💾 Database

Uses **LowDB** — file-based JSON database.

### Location

```
arcane-reader/data/arcane-db.json
```

### Schema

```typescript
interface DatabaseSchema {
  projects: Project[];
  settings: {
    lastOpenedProject?: string;
  };
}

interface Project {
  id: string;
  name: string;
  sourceLanguage: string;  // 'en'
  targetLanguage: string;  // 'ru'
  chapters: Chapter[];
  glossary: GlossaryEntry[];
  settings: ProjectSettings;
  createdAt: string;
  updatedAt: string;
}

interface Chapter {
  id: string;
  number: number;
  title: string;
  originalText: string;
  translatedText?: string;
  status: 'pending' | 'translating' | 'completed' | 'error';
  translationMeta?: {
    tokensUsed: number;
    duration: number;
    model: string;
    translatedAt: string;
  };
}

interface GlossaryEntry {
  id: string;
  type: 'character' | 'location' | 'term';
  original: string;
  translated: string;
  gender?: 'male' | 'female' | 'neutral';
  declensions?: Declensions;
  notes?: string;
  autoDetected?: boolean;
}
```

---

## 🔗 Engine Integration

File `src/services/engine-integration.ts` connects Reader with Engine:

### Functions

```typescript
// Get agent for project
getAgentForProject(project: Project): NovelAgent

// Create translation pipeline
createPipeline(config: AppConfig, project: Project): TranslationPipeline

// Translate chapter through full pipeline
translateChapterWithPipeline(
  config: AppConfig,
  project: Project,
  chapter: Chapter,
  options?: PipelineOptions
): Promise<{
  translatedText: string;
  tokensUsed: number;
  duration: number;
  glossaryUpdates?: GlossaryEntry[];
}>

// Simple translation (without pipeline, for short texts)
translateSimple(
  config: AppConfig,
  text: string,
  glossary: GlossaryEntry[]
): Promise<{ text: string; tokensUsed: number }>

// Auto-declensions for names
getNameDeclensions(
  englishName: string,
  gender?: 'male' | 'female' | 'neutral' | 'unknown'
): {
  translatedName: string;
  declensions: Declensions;
  gender: string;
}

// Clear agent cache
clearAgentCache(projectId: string): void
```

---

## 🖥️ UI Interface

### Architecture

Client is built on **Preact** — lightweight React alternative (3KB gzip):

- **Preact + Hooks** — component approach with hooks
- **Vite** — fast build and HMR
- **TypeScript** — full typing
- **CSS Variables** — theming via variables

### Components

| Component | Description |
|-----------|-------------|
| `Header` | Logo, API status |
| `Sidebar` | Project and chapter list |
| `ProjectList` | Project selection/creation |
| `ChapterList` | Chapters with filters, drag-n-drop upload |
| `ProjectInfo` | Project settings, EPUB/FB2 export, bulk translation |
| `ChapterView` | Chapter viewing/editing |
| `ParagraphList` | Parallel text (original/translation) |
| `GlossaryModal` | Glossary management |
| `ReadingMode` | Full-screen mode for reading translated chapters |

### UI Kit

Base components in `components/ui/`:

```tsx
import { Button, Card, Modal, Input, Badge, Select } from './components/ui';

<Button variant="primary" loading={isLoading}>Translate</Button>
<Card title="📁 Projects">...</Card>
<Modal isOpen={show} onClose={close} title="Title">...</Modal>
```

### Features

- 🇷🇺 Russian language interface
- 🎨 Dark theme with gradients
- ⚡ Instant updates (HMR)
- 📱 Responsive design
- ✨ Animations and transitions

### API Client

Typed client in `api/client.ts`:

```typescript
import { api } from './api/client';

// Projects
const projects = await api.getProjects();
const project = await api.getProject(id);
await api.createProject(name);
await api.deleteProject(id);

// Chapters
await api.uploadChapter(projectId, file, title);
await api.translateChapter(projectId, chapterId);
await api.deleteChapter(projectId, chapterId);

// Glossary
await api.addGlossary(projectId, entry);
await api.updateGlossaryEntry(projectId, entryId, data);
await api.deleteGlossaryEntry(projectId, entryId);

// Export
await api.exportProject(projectId, 'epub');
await api.exportProject(projectId, 'fb2');
```

---

## 📊 Logging

During translation, detailed logs are output:

```
════════════════════════════════════════════════════════════
🔮 TRANSLATION REQUEST
────────────────────────────────────────────────────────────
📖 Chapter: Chapter 1
📊 Size: 5420 characters, ~980 words
🔑 API key: ✅ Configured
🤖 Model: gpt-4-turbo-preview
────────────────────────────────────────────────────────────
🚀 Starting arcane-engine TranslationPipeline...
   Stages: ✅ Analysis | ✅ Translation | ✅ Editing
────────────────────────────────────────────────────────────
✅ TRANSLATION COMPLETED (arcane-engine)
⏱️  Time: 12.3s
📝 Tokens: 2840
════════════════════════════════════════════════════════════
```

---

## 🔒 Security

### .gitignore

```gitignore
.env
node_modules/
dist/
data/arcane-db.json
```

### Important

- **Never** commit `.env` with API keys
- Database `.json` is also in gitignore
- Use `env.example.txt` as template

---

## ⚡ Vite Configuration

File `vite.config.ts`:

```typescript
import { defineConfig } from 'vite';
import preact from '@preact/preset-vite';

export default defineConfig({
  plugins: [preact()],
  server: {
    proxy: {
      '/api': 'http://localhost:3000',
      '/images': 'http://localhost:3000',
    },
  },
  build: {
    outDir: 'dist/client',
    emptyOutDir: true,
  },
});
```

### Proxy

In dev mode, Vite proxies API requests to Express server:
- `/api/*` → `http://localhost:3000`
- `/images/*` → `http://localhost:3000`

### Production Build

```bash
npm run build:client
# Result: dist/client/
#   ├── index.html
#   └── assets/
#       ├── index-*.css  (~5 KB gzip)
#       └── index-*.js   (~15 KB gzip)
```

Express server automatically serves static files from `dist/client/` in production mode.

---

## 📤 EPUB/FB2 Export

The system supports exporting translated projects to e-book formats.

### Supported Formats

- **EPUB** — standard format for most e-readers
- **FB2** — popular format in Russia (FictionBook 2.0)

### Usage

1. Open a project with translated chapters
2. On the project page, click "📚 Export EPUB" or "📖 Export FB2"
3. File will automatically download in browser

### Features

- Only chapters with `completed` status are exported
- Chapters are sorted by number (`number`)
- Text is automatically formatted (paragraphs, headings)
- Metadata includes project name, author, translation date

### API

```typescript
// Export to EPUB
const result = await api.exportProject(projectId, 'epub');
// result.url - URL to download file

// Export to FB2
const result = await api.exportProject(projectId, 'fb2');
```

### File Location

Exported files are saved in:
```
arcane-reader/data/exports/
```

And accessible via URL:
```
http://localhost:3000/exports/{filename}
```

---

## 📖 Reading Mode

Full-screen interface for comfortable reading of translated chapters.

### Features

- **Navigation** — move between chapters (← → keys)
- **Table of Contents** — quick jump to any chapter
- **Settings** — font, size, line spacing, theme
- **Share** — generate link to current chapter
- **URL parameters** — support for direct links to chapters

### Usage

1. Open a project with translated chapters
2. Click "📖 Reading Mode" on the project page
3. Or click "📖 Read" in chapter view

### URL Parameters

```
# Open project
?project={projectId}

# Open specific chapter
?project={projectId}&chapter={chapterId}

# Open in reading mode
?project={projectId}&reading=true

# Open specific chapter in reading mode
?project={projectId}&chapter={chapterId}&reading=true
```

### Reading Settings

- **Font** — font family selection
- **Size** — text size (12-24px)
- **Line spacing** — distance between lines
- **Theme** — dark, light, sepia, high contrast
