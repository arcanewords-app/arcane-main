# 🔧 Arcane Engine

Движок перевода новелл. Реализует 3-стадийный пайплайн и агент контекста.

---

## 📦 Экспорты

```typescript
// Types
export type { Language, Gender, Declensions, TextChunk, TranslationConfig } from './types/common.js';
export type { Character, Location, Term, Glossary, GlossaryUpdate } from './types/glossary.js';
export type { StyleProfile, ChapterSummary, CurrentContext, NovelAgentState, AnalysisResult, AgentContext } from './types/agent.js';
export type { StageType, StageResult, TranslationDraft, EditedTranslation, PipelineResult, PipelineOptions } from './types/pipeline.js';

// Interfaces
export type { ILLMProvider, LLMProviderConfig, Message, CompletionOptions, CompletionResult } from './interfaces/llm-provider.js';

// Classes
export { OpenAIProvider } from './providers/openai.js';
export { NovelAgent } from './agents/novel-agent.js';
export { GlossaryManager } from './glossary/glossary-manager.js';
export { TranslationPipeline } from './pipeline/translation-pipeline.js';
export { AnalyzeStage, TranslateStage, EditStage } from './stages/...';

// Utils
export { chunkText, mergeChunks, estimateTokens, splitIntoSections } from './utils/chunker.js';
export { translateAndDeclineName, declineNameRu, transliterateEnToRu, EN_RU_NAMES } from './glossary/declension-ru.js';

// Prompts
export { ANALYZER_SYSTEM_PROMPT, createAnalyzerPrompt } from './prompts/system/analyzer.js';
export { TRANSLATOR_SYSTEM_PROMPT, createTranslatorPrompt, createGlossaryPromptSection } from './prompts/system/translator.js';
export { EDITOR_SYSTEM_PROMPT, createEditorPrompt, QUALITY_CHECK_PROMPT } from './prompts/system/editor.js';
```

---

## 🔄 TranslationPipeline

Оркестрирует 3-стадийный процесс перевода.

### Использование

```typescript
import { TranslationPipeline, OpenAIProvider, NovelAgent } from 'arcane-engine';

// 1. Создать провайдер
const provider = new OpenAIProvider({
  apiKey: 'sk-...',
  model: 'gpt-4-turbo-preview',
});

// 2. Создать агента
const agent = NovelAgent.create({
  novelId: 'novel-1',
  title: 'My Novel',
  sourceLanguage: 'en',
  targetLanguage: 'ru',
});

// 3. Создать пайплайн
const pipeline = new TranslationPipeline({ provider, agent });

// 4. Перевести главу
const result = await pipeline.translateChapter(sourceText, chapterNumber, {
  skipAnalysis: false,  // Выполнять анализ (Stage 1)
  skipEditing: false,   // Выполнять редактуру (Stage 3)
  chunkSize: 2000,      // Токенов на чанк
});

console.log(result.finalTranslation);
console.log(`Токенов: ${result.totalTokensUsed}`);
console.log(`Время: ${result.totalDuration}ms`);
```

### Конфигурация

```typescript
interface PipelineOptions {
  skipAnalysis?: boolean;   // Пропустить Stage 1 (анализ)
  skipEditing?: boolean;    // Пропустить Stage 3 (редактура)
  chunkSize?: number;       // Размер чанка в токенах
  retryAttempts?: number;   // Попытки при ошибке
}
```

### Результат

```typescript
interface PipelineResult {
  chapterNumber: number;
  originalText: string;
  
  stage1: StageResult<AnalysisResult>;   // Результат анализа
  stage2: StageResult<TranslationDraft>; // Черновик перевода
  stage3: StageResult<EditedTranslation>; // Редактура
  
  finalTranslation: string;              // Финальный текст
  
  totalTokensUsed: number;
  totalDuration: number;
  
  updatedContext: AgentContext;          // Обновлённый контекст
}
```

---

## 🎭 NovelAgent

Поддерживает контекст произведения между главами.

### Создание

```typescript
const agent = NovelAgent.create({
  novelId: 'unique-id',
  title: 'Novel Title',
  sourceLanguage: 'en',
  targetLanguage: 'ru',
});
```

### Глоссарий

```typescript
// Добавить персонажа
agent.addCharacter({
  originalName: 'John',
  translatedName: 'Джон',
  gender: 'male',
  declensions: {
    nominative: 'Джон',
    genitive: 'Джона',
    dative: 'Джону',
    accusative: 'Джона',
    instrumental: 'Джоном',
    prepositional: 'Джоне',
  },
  description: 'Main protagonist',
  aliases: ['Johnny'],
  firstAppearance: 1,
  isMainCharacter: true,
});

// Получить контекст для промпта
const context = agent.getContext();
```

### Сериализация

```typescript
// Сохранить состояние
const state = agent.toJSON();

// Восстановить
const agent = NovelAgent.fromJSON(state);
```

---

## 📚 Склонения (Declensions)

Автоматическая генерация падежных форм для русских имён.

### Быстрое использование

```typescript
import { translateAndDeclineName } from 'arcane-engine';

const result = translateAndDeclineName('Alexander', 'male');
// {
//   translatedName: 'Александр',
//   gender: 'male',
//   declensions: {
//     nominative: 'Александр',
//     genitive: 'Александра',
//     dative: 'Александру',
//     accusative: 'Александра',
//     instrumental: 'Александром',
//     prepositional: 'Александре',
//   }
// }
```

### Транслитерация

```typescript
import { transliterateEnToRu } from 'arcane-engine';

transliterateEnToRu('Michael'); // 'Майкл'
transliterateEnToRu('Catherine'); // 'Кэтрин'
```

### Словарь известных имён

```typescript
import { EN_RU_NAMES } from 'arcane-engine';

EN_RU_NAMES['Alexander']; // { ru: 'Александр', gender: 'male' }
EN_RU_NAMES['Elizabeth']; // { ru: 'Елизавета', gender: 'female' }
```

---

## 🧩 Стадии перевода

### Stage 1: Analyze (Анализ)

Извлекает сущности и анализирует стиль текста.

```typescript
import { AnalyzeStage } from 'arcane-engine';

const stage = new AnalyzeStage(provider);
const result = await stage.execute(sourceText, {
  chapterNumber: 1,
  existingGlossary: glossary,
});

// result.data содержит:
// - foundCharacters[]
// - foundLocations[]
// - foundTerms[]
// - chapterSummary
// - keyEvents[]
// - styleNotes
```

### Stage 2: Translate (Перевод)

Выполняет перевод с учётом глоссария.

```typescript
import { TranslateStage } from 'arcane-engine';

const stage = new TranslateStage(provider);
const result = await stage.execute(sourceText, {
  context: agentContext,
  chunkSize: 2000,
});

// result.data.translatedText - полный перевод
// result.data.chunkResults[] - результаты по чанкам
```

### Stage 3: Edit (Редактура)

Полирует перевод для лучшего качества.

```typescript
import { EditStage } from 'arcane-engine';

const stage = new EditStage(provider);
const result = await stage.execute(translatedText, originalText, {
  context: agentContext,
  checkQuality: true,
});

// result.data.finalText - отредактированный текст
// result.data.qualityScore - оценка качества (1-10)
```

---

## 📝 Системные промпты

### Analyzer Prompt

Задача: извлечь персонажей, локации, термины, проанализировать стиль.

Выход: JSON с `characters`, `locations`, `terms`, `chapterSummary`, `keyEvents`, `styleNotes`.

### Translator Prompt

Задача: точный перевод с соблюдением глоссария.

Правила:
- Использовать ТОЧНЫЕ переводы из глоссария
- Применять правильные грамматические формы
- Сохранять стиль и голос автора
- Адаптировать культурные отсылки

### Editor Prompt

Задача: улучшить читаемость и литературное качество.

Правила:
- Исправлять неестественные конструкции
- Сохранять смысл и авторский стиль
- НЕ менять имена и термины из глоссария
- НЕ добавлять/удалять контент

---

## 🔧 Утилиты

### Chunker

Разбивает текст на чанки для API.

```typescript
import { chunkText, mergeChunks, estimateTokens } from 'arcane-engine';

// Разбить текст
const chunks = chunkText(longText, {
  maxTokens: 2000,
  preserveParagraphs: true,
});

// Оценить токены
const tokens = estimateTokens(text); // ~4 символа = 1 токен

// Объединить обратно
const merged = mergeChunks(translatedChunks);
```

---

## 🔌 LLM Providers

### OpenAI Provider

```typescript
import { OpenAIProvider } from 'arcane-engine';

const provider = new OpenAIProvider({
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview', // или gpt-4o, gpt-3.5-turbo
  temperature: 0.7,
  maxTokens: 4096,
});
```

### Интерфейс ILLMProvider

Для добавления других провайдеров (Anthropic, local LLM):

```typescript
interface ILLMProvider {
  complete(messages: Message[], options?: CompletionOptions): Promise<CompletionResult>;
  getModelInfo(): { name: string; maxTokens: number };
}
```

