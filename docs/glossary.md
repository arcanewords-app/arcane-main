# 📚 Glossary and Declensions

Documentation on working with glossary and declension system for Russian language.

---

## Overview

Glossary ensures:
- **Consistency** — same translation of names and terms
- **Declensions** — correct case forms for Russian language
- **Context** — additional information for translator

---

## Entry Types

### Character

```typescript
interface Character {
  id: string;
  originalName: string;      // "Alexander"
  translatedName: string;    // "Александр"
  declensions: Declensions;  // Case forms
  gender: 'male' | 'female' | 'neutral' | 'unknown';
  description: string;       // "Main protagonist, young mage-researcher"
  aliases: string[];         // ["Alex", "Sasha"]
  firstAppearance: number;   // Chapter number of first mention
  isMainCharacter: boolean;
}
```

**Description**: Automatically extracted during chapter analysis or added manually. Helps translator better understand character context for more accurate translation of dialogues and descriptions.

### Location

```typescript
interface Location {
  id: string;
  originalName: string;      // "Crystal Palace"
  translatedName: string;    // "Хрустальный дворец"
  description: string;       // "Capital city, major trading hub"
  type: 'city' | 'country' | 'building' | 'region' | 'world' | 'other';
}
```

**Description**: Automatically extracted during analysis or added manually. Contains brief location characteristics for better translation context.

### Term

```typescript
interface Term {
  id: string;
  originalTerm: string;      // "mana"
  translatedTerm: string;    // "мана"
  category: 'skill' | 'magic' | 'item' | 'title' | 'organization' | 'race' | 'other';
  description: string;        // "Magical energy used for spells"
  context?: string;
}
```

**Description**: Automatically extracted during analysis or added manually. Explains term meaning and usage for consistent translation.

---

## Declensions

Russian language has 6 cases. All forms are needed for correct name usage in sentences.

### Cases

```typescript
interface Declensions {
  nominative: string;    // Nominative (who? what?) — Иван
  genitive: string;      // Genitive (whom? what?) — Ивана
  dative: string;        // Dative (to whom? to what?) — Ивану
  accusative: string;    // Accusative (whom? what?) — Ивана
  instrumental: string;  // Instrumental (with whom? with what?) — Иваном
  prepositional: string; // Prepositional (about whom? about what?) — Иване
}
```

### Usage Examples in Text

| Case | Question | Example |
|------|----------|---------|
| Nominative | Who? | **Иван** came home |
| Genitive | Whom is missing? | There is no **Ивана** here |
| Dative | To whom? | Give this to **Ивану** |
| Accusative | Whom do I see? | I see **Ивана** |
| Instrumental | With whom? | I'm going with **Иваном** |
| Prepositional | About whom? | Tell me about **Иване** |

---

## Automatic Declensions

Arcane uses the **Petrovich** library for automatic declension generation.

### Usage

```typescript
import { translateAndDeclineName } from 'arcane-engine';

const result = translateAndDeclineName('Alexander', 'male');

console.log(result);
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

For names without known translation, transliteration is used:

```typescript
import { transliterateEnToRu } from 'arcane-engine';

transliterateEnToRu('Michael');   // 'Майкл'
transliterateEnToRu('Catherine'); // 'Кэтрин'
transliterateEnToRu('George');    // 'Джордж'
```

### Known Names Dictionary

```typescript
import { EN_RU_NAMES } from 'arcane-engine';

// Predefined translations of popular names
EN_RU_NAMES['Alexander'] = { ru: 'Александр', gender: 'male' };
EN_RU_NAMES['Elizabeth'] = { ru: 'Елизавета', gender: 'female' };
EN_RU_NAMES['John']      = { ru: 'Джон', gender: 'male' };
EN_RU_NAMES['Mary']      = { ru: 'Мария', gender: 'female' };
// ... and others
```

---

## Glossary API

### Adding via REST API

```bash
# Add character (declensions generated automatically)
curl -X POST http://localhost:3000/api/projects/{id}/glossary \
  -H "Content-Type: application/json" \
  -d '{
    "type": "character",
    "original": "John",
    "gender": "male"
  }'

# Response:
# {
#   "id": "gl_001",
#   "type": "character",
#   "original": "John",
#   "translated": "Джон",
#   "gender": "male",
#   "declensions": {
#     "nominative": "Джон",
#     "genitive": "Джона",
#     ...
#   }
# }
```

### Adding via Code

```typescript
import { NovelAgent, translateAndDeclineName } from 'arcane-engine';

const agent = NovelAgent.create({ ... });

// Auto-declensions
const { translatedName, declensions, gender } = translateAndDeclineName('William', 'male');

agent.addCharacter({
  originalName: 'William',
  translatedName,
  declensions,
  gender,
  description: 'The prince',
  aliases: ['Will', 'Billy'],
  firstAppearance: 1,
  isMainCharacter: true,
});
```

---

## GlossaryManager

Class for managing glossary in the engine.

```typescript
import { GlossaryManager } from 'arcane-engine';

const manager = new GlossaryManager();

// Add character
manager.addCharacter({
  originalName: 'Sarah',
  translatedName: 'Сара',
  gender: 'female',
  // ...
});

// Find character
const character = manager.findCharacter('Sarah');

// Get all entries for prompt
const glossaryText = manager.toPromptFormat();
```

---

## Prompt Integration

Glossary is automatically included in translator prompt with descriptions for better context:

```typescript
import { createGlossaryPromptSection } from 'arcane-engine';

const section = createGlossaryPromptSection(
  [
    { 
      original: 'John', 
      translated: 'Джон', 
      declensions: { genitive: 'Джона', dative: 'Джону', ... },
      description: 'Main protagonist, young mage-researcher',
    },
  ],
  [
    { 
      original: 'Dark Forest', 
      translated: 'Тёмный лес',
      description: 'Mysterious forest where magic is strongest',
    },
  ],
  [
    { 
      original: 'mana', 
      translated: 'мана',
      description: 'Magical energy used for casting spells',
    },
  ]
);

// Result:
// ### Characters
// - John → Джон (gen: Джона, dat: Джону) - Main protagonist, young mage-researcher
//
// ### Locations
// - Dark Forest → Тёмный лес - Mysterious forest where magic is strongest
//
// ### Terms
// - mana → мана - Magical energy used for casting spells
```

Descriptions help translator better understand context and character traits, improving dialogue and description translation quality.

---

## Declension Features

### Male Names

| Original | Translation | Gen. | Dat. | Acc. | Inst. | Prep. |
|----------|-------------|------|------|------|-------|-------|
| Alexander | Александр | Александра | Александру | Александра | Александром | Александре |
| John | Джон | Джона | Джону | Джона | Джоном | Джоне |
| Michael | Майкл | Майкла | Майклу | Майкла | Майклом | Майкле |

### Female Names

| Original | Translation | Gen. | Dat. | Acc. | Inst. | Prep. |
|----------|-------------|------|------|------|-------|-------|
| Mary | Мария | Марии | Марии | Марию | Марией | Марии |
| Elizabeth | Елизавета | Елизаветы | Елизавете | Елизавету | Елизаветой | Елизавете |
| Sarah | Сара | Сары | Саре | Сару | Сарой | Саре |

### Invariable Names

Some names don't decline in Russian:

```typescript
// Names ending in vowel (foreign female)
'Sophie' → 'Софи' (all cases the same)

// Monosyllabic foreign names
'Lee' → 'Ли' (doesn't decline)
```

---

## Glossary Usage Tips

### 1. Add Characters in Advance

Before translation, add all known characters to the glossary. This ensures consistency from the first chapter.

### 2. Specify Gender

Gender affects declension correctness. If gender is unknown, use `'unknown'`.

### 3. Use Aliases

Add pseudonyms and nicknames:
```typescript
{
  originalName: 'William',
  aliases: ['Will', 'Billy', 'the Prince'],
}
```

### 4. Check Auto-declensions

Automatic declensions work well for most names, but complex cases should be checked.

### 5. Use Descriptions

Descriptions are automatically extracted during chapter analysis, but can be edited manually:

```typescript
{
  original: 'Mark',
  translated: 'Марк',
  description: 'Main protagonist, young mage-researcher, analytical',
  firstAppearance: 1,  // Chapter number of first mention
}
```

Descriptions are used in translation prompts, improving translation quality and context.

### 6. First Appearance (firstAppearance)

When automatically detecting new glossary entries, the system saves the chapter number of first mention:

```typescript
{
  original: 'Dark Forest',
  translated: 'Тёмный лес',
  firstAppearance: 3,  // First mentioned in chapter 3
  autoDetected: true,
}
```

This helps track when and where a character, location, or term first appeared.

### 7. Image Gallery

Each glossary entry can have multiple images:

```typescript
{
  id: 'gl_001',
  original: 'Mark',
  translated: 'Марк',
  imageUrls: [
    '/uploads/project1/gl_001/image1.jpg',
    '/uploads/project1/gl_001/image2.jpg',
  ],
}
```

Images can be added, deleted, and viewed in fullscreen mode via UI.

### 8. Add Notes

The `notes` field (separate from `description`) helps during manual verification:
```typescript
{
  original: 'The Order',
  translated: 'Орден',
  description: 'Secret organization of mages',  // Entity description
  notes: 'Always capitalized',                 // User notes
}
```
