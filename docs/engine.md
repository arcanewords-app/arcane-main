# 🔧 Arcane Engine

Translation engine for novels. Implements a 3-stage pipeline and context agent.

---

## 📦 Exports

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

Orchestrates the 3-stage translation process.

### Usage

```typescript
import { TranslationPipeline, OpenAIProvider, NovelAgent } from 'arcane-engine';

// 1. Create provider
const provider = new OpenAIProvider({
  apiKey: 'sk-...',
  model: 'gpt-4-turbo-preview',
});

// 2. Create agent
const agent = NovelAgent.create({
  novelId: 'novel-1',
  title: 'My Novel',
  sourceLanguage: 'en',
  targetLanguage: 'ru',
});

// 3. Create pipeline
const pipeline = new TranslationPipeline({ provider, agent });

// 4. Translate chapter
const result = await pipeline.translateChapter(sourceText, chapterNumber, {
  skipAnalysis: false,  // Execute analysis (Stage 1)
  skipEditing: false,   // Execute editing (Stage 3)
  chunkSize: 2000,      // Tokens per chunk
});

console.log(result.finalTranslation);
console.log(`Tokens: ${result.totalTokensUsed}`);
console.log(`Time: ${result.totalDuration}ms`);
```

### Configuration

```typescript
interface PipelineOptions {
  skipAnalysis?: boolean;   // Skip Stage 1 (analysis)
  skipEditing?: boolean;    // Skip Stage 3 (editing)
  chunkSize?: number;        // Chunk size in tokens
  retryAttempts?: number;   // Retry attempts on error
}
```

### Result

```typescript
interface PipelineResult {
  chapterNumber: number;
  originalText: string;
  
  stage1: StageResult<AnalysisResult>;   // Analysis result
  stage2: StageResult<TranslationDraft>; // Translation draft
  stage3: StageResult<EditedTranslation>; // Edited version
  
  finalTranslation: string;              // Final text
  
  totalTokensUsed: number;
  totalDuration: number;
  
  updatedContext: AgentContext;          // Updated context
}
```

---

## 🎭 NovelAgent

Maintains work context between chapters.

### Creation

```typescript
const agent = NovelAgent.create({
  novelId: 'unique-id',
  title: 'Novel Title',
  sourceLanguage: 'en',
  targetLanguage: 'ru',
});
```

### Glossary

```typescript
// Add character
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

// Get context for prompt
const context = agent.getContext();
```

### Serialization

```typescript
// Save state
const state = agent.toJSON();

// Restore
const agent = NovelAgent.fromJSON(state);
```

---

## 📚 Declensions

Automatic generation of case forms for Russian names.

### Quick Usage

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

### Transliteration

```typescript
import { transliterateEnToRu } from 'arcane-engine';

transliterateEnToRu('Michael'); // 'Майкл'
transliterateEnToRu('Catherine'); // 'Кэтрин'
```

### Known Names Dictionary

```typescript
import { EN_RU_NAMES } from 'arcane-engine';

EN_RU_NAMES['Alexander']; // { ru: 'Александр', gender: 'male' }
EN_RU_NAMES['Elizabeth']; // { ru: 'Елизавета', gender: 'female' }
```

---

## 🧩 Translation Stages

### Stage 1: Analyze

Extracts entities and analyzes text style.

```typescript
import { AnalyzeStage } from 'arcane-engine';

const stage = new AnalyzeStage(provider);
const result = await stage.execute(sourceText, {
  chapterNumber: 1,
  existingGlossary: glossary,
});

// result.data contains:
// - foundCharacters[]
// - foundLocations[]
// - foundTerms[]
// - chapterSummary
// - keyEvents[]
// - styleNotes
```

### Stage 2: Translate

Performs translation with glossary consideration.

```typescript
import { TranslateStage } from 'arcane-engine';

const stage = new TranslateStage(provider);
const result = await stage.execute(sourceText, {
  context: agentContext,
  chunkSize: 2000,
});

// result.data.translatedText - full translation
// result.data.chunkResults[] - results per chunk
```

### Stage 3: Edit

Polishes translation for better quality.

```typescript
import { EditStage } from 'arcane-engine';

const stage = new EditStage(provider);
const result = await stage.execute(translatedText, originalText, {
  context: agentContext,
  checkQuality: true,
});

// result.data.finalText - edited text
// result.data.qualityScore - quality score (1-10)
```

---

## 📝 System Prompts

### Analyzer Prompt

Task: extract characters, locations, terms, analyze style.

Output: JSON with `characters`, `locations`, `terms`, `chapterSummary`, `keyEvents`, `styleNotes`.

### Translator Prompt

Task: accurate translation with glossary compliance.

Rules:
- Use EXACT translations from glossary
- Apply correct grammatical forms
- Preserve author's style and voice
- Adapt cultural references

### Editor Prompt

Task: improve readability and literary quality.

Rules:
- Fix unnatural constructions
- Preserve meaning and author's style
- DO NOT change names and terms from glossary
- DO NOT add/remove content

---

## 🔧 Utilities

### Chunker

Splits text into chunks for API.

```typescript
import { chunkText, mergeChunks, estimateTokens } from 'arcane-engine';

// Split text
const chunks = chunkText(longText, {
  maxTokens: 2000,
  preserveParagraphs: true,
});

// Estimate tokens
const tokens = estimateTokens(text); // ~4 characters = 1 token

// Merge back
const merged = mergeChunks(translatedChunks);
```

---

## 🔌 LLM Providers

### OpenAI Provider

```typescript
import { OpenAIProvider } from 'arcane-engine';

const provider = new OpenAIProvider({
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview', // or gpt-4o, gpt-3.5-turbo
  temperature: 0.7,
  maxTokens: 4096,
});
```

### ILLMProvider Interface

To add other providers (Anthropic, local LLM):

```typescript
interface ILLMProvider {
  complete(messages: Message[], options?: CompletionOptions): Promise<CompletionResult>;
  getModelInfo(): { name: string; maxTokens: number };
}
```
