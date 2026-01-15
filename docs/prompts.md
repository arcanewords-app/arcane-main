# 📝 System Prompts

Documentation on system prompts used in the 3-stage translation pipeline.

---

## Stage Overview

| Stage | File | Purpose |
|-------|------|---------|
| Stage 1 | `analyzer.ts` | Text analysis, entity extraction |
| Stage 2 | `translator.ts` | Accurate translation with glossary |
| Stage 3 | `editor.ts` | Polishing and quality improvement |

---

## Stage 1: Analyzer

### System Prompt

```
You are an expert literary analyst specializing in novel analysis for translation preparation.

Your task is to analyze the provided chapter/text and extract:
1. Characters: Names, roles, relationships, gender (for proper declension)
2. Locations: Places, settings, world-building elements
3. Special Terms: Skills, magic systems, titles, organizations, items
4. Style Analysis: Narrative voice, tone, dialogue characteristics

You must output a structured JSON analysis that will be used by the translator.
```

### Output Format

```json
{
  "characters": [
    {
      "name": "original name",
      "suggestedTranslation": "suggested translation",
      "gender": "male|female|neutral|unknown",
      "role": "protagonist|antagonist|supporting|minor",
      "description": "brief description",
      "context": "first appearance context"
    }
  ],
  "locations": [
    {
      "name": "original name",
      "suggestedTranslation": "suggested translation",
      "type": "city|country|building|region|world|other",
      "description": "brief description"
    }
  ],
  "terms": [
    {
      "term": "original term",
      "suggestedTranslation": "suggested translation",
      "category": "skill|magic|item|title|organization|race|other",
      "description": "meaning and usage"
    }
  ],
  "chapterSummary": "2-3 sentence summary",
  "keyEvents": ["event 1", "event 2"],
  "mood": "chapter mood/atmosphere",
  "styleNotes": "notable stylistic elements"
}
```

### Prompt Creation Function

```typescript
createAnalyzerPrompt(
  sourceText: string,
  sourceLanguage: string,
  targetLanguage: string,
  existingGlossary?: string
): string
```

---

## Stage 2: Translator

### System Prompt

```
You are an expert literary translator specializing in novel translation.

Your task is to produce an accurate, natural-sounding translation that:
1. Preserves meaning: Capture the original intent and nuance
2. Maintains consistency: Use the provided glossary for all names and terms
3. Respects style: Match the author's voice and tone
4. Sounds natural: The translation should read like native literature
```

### Translation Rules

#### Names and Terms
- Use EXACT translations from glossary
- Apply correct grammatical forms (declensions)
- For Russian: use correct case endings

#### Style Preservation
- Preserve sentence structure when possible
- Preserve paragraphs and formatting
- Maintain consistent narrative voice
- Preserve dialogue style

#### Cultural Adaptation
- Adapt cultural references when necessary
- Preserve original setting atmosphere
- Correctly handle forms of address and titles

### Prompt Creation Function

```typescript
createTranslatorPrompt(
  sourceText: string,
  glossary: string,
  context: string,
  styleGuide: string
): string
```

### Glossary Section

```typescript
createGlossaryPromptSection(
  characters: { original: string; translated: string; declensions?: Record<string, string> }[],
  locations: { original: string; translated: string }[],
  terms: { original: string; translated: string }[]
): string
```

Example output:
```
### Characters
- John → Джон (gen: Джона, dat: Джону)
- Mary → Мария (gen: Марии, dat: Марии)

### Locations
- Crystal Palace → Хрустальный дворец
- Dark Forest → Тёмный лес

### Terms
- mana → мана
- skill → навык
```

---

## Stage 3: Editor

### System Prompt

```
You are an expert literary editor specializing in translated fiction.

Your task is to polish the provided translation to achieve:
1. Natural flow: Sentences should read smoothly in the target language
2. Literary quality: Elevate the prose while preserving the original voice
3. Consistency: Ensure terminology and style remain consistent
4. Readability: Fix any awkward or unnatural phrasings
```

### What to Fix

- Unnatural "calque" constructions from original
- Inappropriate word or phrase choices
- Inconsistent tone or style
- Repetitive vocabulary
- Grammatical and punctuation errors

### What to Preserve

- Original meaning and intent
- Unique author's voice and style
- Character speech patterns
- Emotional impact of scenes
- All proper names and established translations

### What NOT to Do

- Add new content
- Remove important details
- Change character names
- Alter plot
- Over-localize

### Prompt Creation Function

```typescript
createEditorPrompt(
  translatedText: string,
  originalText: string,
  glossary: string,
  styleNotes?: string
): string
```

### Quality Check

```typescript
QUALITY_CHECK_PROMPT = `
Review the translation for quality issues and provide a score from 1-10.

Check for:
- Accuracy (does it convey the original meaning?)
- Fluency (does it read naturally?)
- Consistency (are terms used consistently?)
- Style (does it match the original tone?)

Output JSON:
{
  "score": 8,
  "issues": ["issue 1", "issue 2"],
  "suggestions": ["suggestion 1"]
}
`
```

---

## Prompt Customization

Prompts can be modified for specific needs:

### 1. Changing Translation Style

```typescript
const customTranslator = TRANSLATOR_SYSTEM_PROMPT + `

## Additional Rules for This Novel

- Use formal "Вы" instead of informal "ты"
- Preserve Japanese honorifics (-san, -sama, -kun)
- Transliterate magic incantations, don't translate
`;
```

### 2. Adding Genre-Specific Features

```typescript
const fantasyPrompt = ANALYZER_SYSTEM_PROMPT + `

## Fantasy-specific Analysis

Pay special attention to:
- Magic system terminology
- Racial distinctions (elves, dwarves, etc.)
- Fictional creatures and monsters
- Power levels and rankings
`;
```

### 3. Novel-Specific Configuration

```typescript
const novelSpecificPrompt = `
## Novel-specific Context

This is a cultivation novel where:
- "Qi" should remain as "Ци"
- Realm names are translated but pinyin kept in parentheses
- Character cultivation levels are important plot elements
`;
```

---

## Usage Examples

### Full Pipeline

```typescript
import {
  ANALYZER_SYSTEM_PROMPT,
  TRANSLATOR_SYSTEM_PROMPT,
  EDITOR_SYSTEM_PROMPT,
  createTranslatorPrompt,
  createGlossaryPromptSection,
} from 'arcane-engine';

// Stage 1
const analysis = await llm.complete([
  { role: 'system', content: ANALYZER_SYSTEM_PROMPT },
  { role: 'user', content: sourceText },
]);

// Stage 2
const glossarySection = createGlossaryPromptSection(chars, locs, terms);
const translatorPrompt = createTranslatorPrompt(sourceText, glossarySection, context, styleGuide);

const translation = await llm.complete([
  { role: 'system', content: TRANSLATOR_SYSTEM_PROMPT },
  { role: 'user', content: translatorPrompt },
]);

// Stage 3
const edited = await llm.complete([
  { role: 'system', content: EDITOR_SYSTEM_PROMPT },
  { role: 'user', content: translation },
]);
```
