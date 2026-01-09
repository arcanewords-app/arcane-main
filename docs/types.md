# 🔷 TypeScript Types Reference

Полная документация по типам, используемым в Arcane.

---

## arcane-engine

### Common Types

```typescript
// Поддерживаемые языки
type Language = 'ja' | 'zh' | 'ko' | 'en' | 'ru' | 'pl';

// Пол (для склонений)
type Gender = 'male' | 'female' | 'neutral' | 'unknown';

// Падежные формы
interface Declensions {
  nominative: string;    // кто? что?
  genitive: string;      // кого? чего?
  dative: string;        // кому? чему?
  accusative: string;    // кого? что?
  instrumental: string;  // кем? чем?
  prepositional: string; // о ком? о чём?
}

// Чанк текста для обработки
interface TextChunk {
  id: string;
  content: string;
  index: number;
  tokenCount?: number;
}

// Конфигурация перевода
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
// Персонаж
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

// Локация
interface Location {
  id: string;
  originalName: string;
  translatedName: string;
  description: string;
  type: 'city' | 'country' | 'building' | 'region' | 'world' | 'other';
}

// Термин
interface Term {
  id: string;
  originalTerm: string;
  translatedTerm: string;
  category: 'skill' | 'magic' | 'item' | 'title' | 'organization' | 'race' | 'other';
  description: string;
  context?: string;
}

// Глоссарий
interface Glossary {
  novelId: string;
  version: number;
  lastUpdated: Date;
  characters: Character[];
  locations: Location[];
  terms: Term[];
}

// Обновление глоссария
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
// Профиль стиля
interface StyleProfile {
  narrativeVoice: 'first-person' | 'third-person' | 'omniscient';
  formalityLevel: 'casual' | 'neutral' | 'formal';
  dialogueStyle: string;
  descriptionStyle: string;
  pacing: 'fast' | 'moderate' | 'slow';
}

// Краткое содержание главы
interface ChapterSummary {
  chapterNumber: number;
  summary: string;
  keyEvents: string[];
  activeCharacters: string[];
  location: string;
}

// Текущий контекст
interface CurrentContext {
  lastChapterNumber: number;
  recentEvents: string[];
  activeCharacters: string[];
  currentLocation: string;
  ongoingPlotThreads: string[];
}

// Полное состояние агента
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

// Результат анализа
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

// Контекст для передачи в стадии
interface AgentContext {
  glossary: Glossary;
  styleProfile: StyleProfile;
  recentChapters: ChapterSummary[];
  currentContext: CurrentContext;
}
```

### Pipeline Types

```typescript
// Тип стадии
type StageType = 'analyze' | 'translate' | 'edit';

// Результат стадии
interface StageResult<T> {
  stage: StageType;
  success: boolean;
  data?: T;
  error?: string;
  tokensUsed: number;
  duration: number; // ms
}

// Черновик перевода
interface TranslationDraft {
  originalText: string;
  translatedText: string;
  chunkResults: ChunkTranslation[];
}

// Перевод чанка
interface ChunkTranslation {
  chunkId: string;
  original: string;
  translated: string;
  notes?: string;
}

// Отредактированный перевод
interface EditedTranslation {
  finalText: string;
  changes: EditChange[];
  qualityScore?: number;
}

// Изменение при редактуре
interface EditChange {
  before: string;
  after: string;
  reason: string;
}

// Результат пайплайна
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

// Опции пайплайна
interface PipelineOptions {
  skipAnalysis?: boolean;
  skipEditing?: boolean;
  chunkSize?: number;
  retryAttempts?: number;
}
```

### Provider Types

```typescript
// Сообщение для LLM
interface Message {
  role: 'system' | 'user' | 'assistant';
  content: string;
}

// Опции запроса
interface CompletionOptions {
  temperature?: number;
  maxTokens?: number;
  stop?: string[];
  responseFormat?: { type: 'json_object' | 'text' };
}

// Результат запроса
interface CompletionResult {
  content: string;
  tokensUsed: {
    prompt: number;
    completion: number;
    total: number;
  };
  finishReason: 'stop' | 'length' | 'content_filter';
}

// Конфигурация провайдера
interface LLMProviderConfig {
  apiKey: string;
  model: string;
  baseURL?: string;
  timeout?: number;
}

// Интерфейс провайдера
interface ILLMProvider {
  complete(messages: Message[], options?: CompletionOptions): Promise<CompletionResult>;
  getModelInfo(): { name: string; maxTokens: number };
}
```

---

## arcane-reader

### Database Types

```typescript
// Проект
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

// Глава
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

// Запись глоссария (упрощённая для UI)
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
  notes?: string;
  autoDetected?: boolean;
}

// Настройки проекта
interface ProjectSettings {
  model: string;
  temperature: number;
  skipEditing: boolean;
}

// Схема базы данных
interface DatabaseSchema {
  projects: Project[];
  settings: {
    lastOpenedProject?: string;
  };
}
```

### Config Types

```typescript
// Конфигурация приложения
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
// Статус системы
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

// Проект (для списка)
interface ProjectListItem {
  id: string;
  name: string;
  chapterCount: number;
  translatedCount: number;
}

// Параграф главы
interface Paragraph {
  id: string;
  original: string;
  translated?: string;
}

// Тип записи глоссария
type GlossaryEntryType = 'character' | 'location' | 'term';

// Запись глоссария
interface GlossaryEntry {
  id: string;
  type: GlossaryEntryType;
  original: string;
  translated: string;
  gender?: 'male' | 'female' | 'neutral' | 'unknown';
  notes?: string;
  imageUrl?: string;
  declensions?: Declensions;
}

// Настройки проекта
interface ProjectSettings {
  model: string;
  temperature: number;
  stages: {
    analyze: boolean;
    translate: boolean;
    edit: boolean;
  };
}

// Статус главы
type ChapterStatus = 'pending' | 'translating' | 'completed' | 'error';

// Глава
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

// Полный проект
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

## Использование типов

### Импорт из arcane-engine

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

### Импорт из arcane-reader (сервер)

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

### Импорт из arcane-reader (клиент)

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

