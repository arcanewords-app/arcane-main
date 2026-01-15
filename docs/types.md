# 🔷 TypeScript Types Reference

Complete documentation on types used in Arcane.

---

## arcane-engine

### Common Types

```typescript
// Supported languages
type Language = 'ja' | 'zh' | 'ko' | 'en' | 'ru' | 'pl';

// Gender (for declensions)
type Gender = 'male' | 'female' | 'neutral' | 'unknown';

// Case forms
interface Declensions {
  nominative: string;    // who? what?
  genitive: string;      // whom? what?
  dative: string;        // to whom? to what?
  accusative: string;    // whom? what?
  instrumental: string;  // with whom? with what?
  prepositional: string; // about whom? about what?
}

// Text chunk for processing
interface TextChunk {
  id: string;
  content: string;
  index: number;
  tokenCount?: number;
}

// Translation configuration
interface TranslationConfig {
  sourceLanguage: Language;
  targetLanguage: Language;
  preserveFormatting: boolean;
  maxTokensPerChunk: number;
  temperature: number;
}
```

### Glossary Types

```typescript
// Character
interface Character {
  id: string;
  originalName: string;
  translatedName: string;
  declensions: Declensions;
  gender: Gender;
  description: string;
  aliases: string[];
  firstAppearance: number;
  isMainCharacter: boolean;
}

// Location
interface Location {
  id: string;
  originalName: string;
  translatedName: string;
  description: string;
  type: 'city' | 'country' | 'building' | 'region' | 'world' | 'other';
}

// Term
interface Term {
  id: string;
  originalTerm: string;
  translatedTerm: string;
  category: 'skill' | 'magic' | 'item' | 'title' | 'organization' | 'race' | 'other';
  description: string;
  context?: string;
}

// Glossary
interface Glossary {
  novelId: string;
  version: number;
  lastUpdated: Date;
  characters: Character[];
  locations: Location[];
  terms: Term[];
}

// Glossary update
interface GlossaryUpdate {
  newCharacters: Omit<Character, 'id'>[];
  newLocations: Omit<Location, 'id'>[];
  newTerms: Omit<Term, 'id'>[];
  updatedCharacters: Partial<Character>[];
  updatedLocations: Partial<Location>[];
  updatedTerms: Partial<Term>[];
}
```

### Agent Types

```typescript
// Style profile
interface StyleProfile {
  narrativeVoice: 'first-person' | 'third-person' | 'omniscient';
  formalityLevel: 'casual' | 'neutral' | 'formal';
  dialogueStyle: string;
  descriptionStyle: string;
  pacing: 'fast' | 'moderate' | 'slow';
}

// Chapter summary
interface ChapterSummary {
  chapterNumber: number;
  summary: string;
  keyEvents: string[];
  activeCharacters: string[];
  location: string;
}

// Current context
interface CurrentContext {
  lastChapterNumber: number;
  recentEvents: string[];
  activeCharacters: string[];
  currentLocation: string;
  ongoingPlotThreads: string[];
}

// Full agent state
interface NovelAgentState {
  novelId: string;
  title: string;
  sourceLanguage: Language;
  targetLanguage: Language;
  glossary: Glossary;
  styleProfile: StyleProfile;
  chapterSummaries: ChapterSummary[];
  currentContext: CurrentContext;
  translationHistory: {
    chapterNumber: number;
    translatedAt: Date;
    tokensUsed: number;
  }[];
}

// Analysis result
interface AnalysisResult {
  foundCharacters: {
    name: string;
    suggestedTranslation?: string;
    gender: Gender;
    role: string;
    description: string;
    context: string;
    isNew: boolean;
  }[];
  foundLocations: {
    name: string;
    suggestedTranslation?: string;
    type: string;
    description: string;
    isNew: boolean;
  }[];
  foundTerms: {
    term: string;
    suggestedTranslation?: string;
    category: string;
    description: string;
    isNew: boolean;
  }[];
  chapterSummary: string;
  keyEvents: string[];
  mood: string;
  styleNotes: string;
}

// Context for passing to stages
interface AgentContext {
  glossary: Glossary;
  styleProfile: StyleProfile;
  recentChapters: ChapterSummary[];
  currentContext: CurrentContext;
}
```

### Pipeline Types

```typescript
// Stage type
type StageType = 'analyze' | 'translate' | 'edit';

// Stage result
interface StageResult<T> {
  stage: StageType;
  success: boolean;
  data?: T;
  error?: string;
  tokensUsed: number;
  duration: number; // ms
}

// Translation draft
interface TranslationDraft {
  originalText: string;
  translatedText: string;
  chunkResults: ChunkTranslation[];
}

// Chunk translation
interface ChunkTranslation {
  chunkId: string;
  original: string;
  translated: string;
  notes?: string;
}

// Edited translation
interface EditedTranslation {
  finalText: string;
  changes: EditChange[];
  qualityScore?: number;
}

// Edit change
interface EditChange {
  before: string;
  after: string;
  reason: string;
}

// Pipeline result
interface PipelineResult {
  chapterNumber: number;
  originalText: string;
  
  stage1: StageResult<AnalysisResult>;
  stage2: StageResult<TranslationDraft>;
  stage3: StageResult<EditedTranslation>;
  
  finalTranslation: string;
  
  totalTokensUsed: number;
  totalDuration: number;
  
  updatedContext: AgentContext;
}

// Pipeline options
interface PipelineOptions {
  skipAnalysis?: boolean;
  skipEditing?: boolean;
  chunkSize?: number;
  retryAttempts?: number;
}
```

### Provider Types

```typescript
// Message for LLM
interface Message {
  role: 'system' | 'user' | 'assistant';
  content: string;
}

// Request options
interface CompletionOptions {
  temperature?: number;
  maxTokens?: number;
  stop?: string[];
  responseFormat?: { type: 'json_object' | 'text' };
}

// Request result
interface CompletionResult {
  content: string;
  tokensUsed: {
    prompt: number;
    completion: number;
    total: number;
  };
  finishReason: 'stop' | 'length' | 'content_filter';
}

// Provider configuration
interface LLMProviderConfig {
  apiKey: string;
  model: string;
  baseURL?: string;
  timeout?: number;
}

// Provider interface
interface ILLMProvider {
  complete(messages: Message[], options?: CompletionOptions): Promise<CompletionResult>;
  getModelInfo(): { name: string; maxTokens: number };
}
```

---

## arcane-reader

### Database Types

```typescript
// Project
interface Project {
  id: string;
  name: string;
  sourceLanguage: string;
  targetLanguage: string;
  chapters: Chapter[];
  glossary: GlossaryEntry[];
  settings: ProjectSettings;
  createdAt: string;
  updatedAt: string;
}

// Chapter
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

// Glossary entry (simplified for UI)
interface GlossaryEntry {
  id: string;
  type: 'character' | 'location' | 'term';
  original: string;
  translated: string;
  gender?: 'male' | 'female' | 'neutral';
  declensions?: {
    nominative: string;
    genitive: string;
    dative: string;
    accusative: string;
    instrumental: string;
    prepositional: string;
  };
  description?: string;        // Entity description (from analysis or manual)
  notes?: string;              // User notes (separate from description)
  firstAppearance?: number;     // Chapter number of first mention
  imageUrls?: string[];         // Image gallery
  autoDetected?: boolean;       // Automatically detected during analysis
  // Legacy support
  imageUrl?: string;           // Old field (migrates to imageUrls)
}

// Project settings
interface ProjectSettings {
  model: string;
  temperature: number;
  skipEditing: boolean;
}

// Database schema
interface DatabaseSchema {
  projects: Project[];
  settings: {
    lastOpenedProject?: string;
  };
}
```

### Config Types

```typescript
// Application configuration
interface AppConfig {
  openai: {
    apiKey: string;
    model: string;
  };
  server: {
    port: number;
  };
  translation: {
    maxTokensPerChunk: number;
    temperature: number;
    skipEditing: boolean;
  };
}
```

---

## arcane-reader/client

### Client Types (src/client/types/index.ts)

```typescript
// System status
interface SystemStatus {
  status: 'ok' | 'error';
  ai: {
    configured: boolean;
    model: string;
  };
  database: {
    connected: boolean;
    projectCount: number;
  };
}

// Project (for list)
interface ProjectListItem {
  id: string;
  name: string;
  chapterCount: number;
  translatedCount: number;
}

// Chapter paragraph
interface Paragraph {
  id: string;
  original: string;
  translated?: string;
}

// Glossary entry type
type GlossaryEntryType = 'character' | 'location' | 'term';

// Glossary entry
interface GlossaryEntry {
  id: string;
  type: GlossaryEntryType;
  original: string;
  translated: string;
  gender?: 'male' | 'female' | 'neutral' | 'unknown';
  description?: string;        // Entity description (from analysis or manual)
  notes?: string;              // User notes (separate from description)
  firstAppearance?: number;     // Chapter number of first mention
  imageUrls?: string[];         // Image gallery
  autoDetected?: boolean;       // Automatically detected during analysis
  declensions?: Declensions;
  // Legacy support
  imageUrl?: string;           // Old field (migrates to imageUrls)
}

// Project settings
interface ProjectSettings {
  model: string;
  temperature: number;
  stages: {
    analyze: boolean;
    translate: boolean;
    edit: boolean;
  };
}

// Chapter status
type ChapterStatus = 'pending' | 'translating' | 'completed' | 'error';

// Chapter
interface Chapter {
  id: string;
  number: number;
  title: string;
  originalText: string;
  translatedText?: string;
  status: ChapterStatus;
  paragraphs: Paragraph[];
  translationMeta?: {
    tokensUsed: number;
    duration: number;
    model: string;
    translatedAt: string;
  };
}

// Full project
interface Project {
  id: string;
  name: string;
  sourceLanguage: string;
  targetLanguage: string;
  chapters: Chapter[];
  glossary: GlossaryEntry[];
  settings: ProjectSettings;
  createdAt: string;
  updatedAt: string;
}
```

---

## Type Usage

### Import from arcane-engine

```typescript
import type {
  // Common
  Language,
  Gender,
  Declensions,
  TextChunk,
  TranslationConfig,
  
  // Glossary
  Character,
  Location,
  Term,
  Glossary,
  GlossaryUpdate,
  
  // Agent
  StyleProfile,
  ChapterSummary,
  CurrentContext,
  NovelAgentState,
  AnalysisResult,
  AgentContext,
  
  // Pipeline
  StageType,
  StageResult,
  TranslationDraft,
  EditedTranslation,
  PipelineResult,
  PipelineOptions,
  
  // Provider
  ILLMProvider,
  LLMProviderConfig,
  Message,
  CompletionOptions,
  CompletionResult,
} from 'arcane-engine';
```

### Import from arcane-reader (server)

```typescript
import type {
  Project,
  Chapter,
  GlossaryEntry,
  ProjectSettings,
  DatabaseSchema,
} from './storage/database.js';

import type { AppConfig } from './config.js';
```

### Import from arcane-reader (client)

```typescript
import type {
  SystemStatus,
  ProjectListItem,
  Project,
  Chapter,
  Paragraph,
  GlossaryEntry,
  GlossaryEntryType,
  ProjectSettings,
  ChapterStatus,
  Declensions,
} from './types';
```
